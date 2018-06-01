---
title: Muduo Source Read
date: 2018-05-31 23:36:13
tags:
---
# muduo 源码学习记录
## EventLoop 的核心代码：
1. EventLoop，顾名思义就是事件循环，主要工作是完成poll到的Event的分发和处理

        分发: 
        pollReturnTime_ = poller_->poll(kPollTimeMs, &activeChannels_);
        处理
        currentActiveChannel_->handleEvent(pollReturnTime_);
        
        void EventLoop::loop() {
          assert(!looping_);
          assertInLoopThread();
          looping_ = true;
          quit_ = false;  // FIXME: what if someone calls quit() before loop() ?
          LOG_TRACE << "EventLoop " << this << " start looping";
        
          while (!quit_) {
            activeChannels_.clear();
            pollReturnTime_ = poller_->poll(kPollTimeMs, &activeChannels_);
            ++iteration_;
            if (Logger::logLevel() <= Logger::TRACE) {
              printActiveChannels();
            }
            // TODO sort channel by priority
            eventHandling_ = true;
            for (ChannelList::iterator it = activeChannels_.begin();
                it != activeChannels_.end(); ++it) {
              currentActiveChannel_ = *it;
              currentActiveChannel_->handleEvent(pollReturnTime_);
            }
            currentActiveChannel_ = NULL;
            eventHandling_ = false;
            doPendingFunctors();
          }
          LOG_TRACE << "EventLoop " << this << " stop looping";
          looping_ = false;
        }
    
2. 消息的来源有三种：
- 来自sock 的 POLLIN / POLLOUT / POLLHUP 等event：用于socket的 nonblock read/write
- 来自timefd 的readable event: 用于timer functor的定时执行
- 来之 eventfd 的消息：用于唤醒loop，doPendingFunctor
    > 任何一个Channel对象在call update的时候，都会将自身的fd加到所属loop的poller的pollfds_队列里，以后poller就  
    会负责监控此fd的的 POLLIN / POLLOUT / POLLHUP 等event：PollPoller::updateChannel

        void PollPoller::updateChannel(Channel* channel) {
          Poller::assertInLoopThread();
          LOG_TRACE << "fd = " << channel->fd() << " events = " << channel->events();
          if (channel->index() < 0) {
            // a new one, add to pollfds_
            assert(channels_.find(channel->fd()) == channels_.end());
            struct pollfd pfd;
            pfd.fd = channel->fd();
            pfd.events = static_cast<short>(channel->events());
            pfd.revents = 0;
            pollfds_.push_back(pfd);
            int idx = static_cast<int>(pollfds_.size())-1;
            channel->set_index(idx);
            channels_[pfd.fd] = channel;
          } else {
            // update existing one
            assert(channels_.find(channel->fd()) != channels_.end());
            assert(channels_[channel->fd()] == channel);
            int idx = channel->index();
            assert(0 <= idx && idx < static_cast<int>(pollfds_.size()));
            struct pollfd& pfd = pollfds_[idx];
            assert(pfd.fd == channel->fd() || pfd.fd == -channel->fd()-1);
            pfd.events = static_cast<short>(channel->events());
            pfd.revents = 0;
            if (channel->isNoneEvent()) {
              // ignore this pollfd
              pfd.fd = -channel->fd()-1;
            }
          }
        }
                

3. poll event 处理流程：
    > poll event -> Channel -> TcpConnection -> TcpClient
    
    1. poll event -> Channel
    
            POLLHUP && !POLLIN -> Channel::closeCallback_()
            POLLERR || POLLNVAL -> Channel::errorCallback_()
            POLLIN || POLLPRI || POLLRDHUP -> Channel::readCallback_()
            POLLOUT -> Channel::writeCallback_()
            
            void setReadCallback(const ReadEventCallback& cb)
            { readCallback_ = cb; }
            void setWriteCallback(const EventCallback& cb)
            { writeCallback_ = cb; }
            void setCloseCallback(const EventCallback& cb)
            { closeCallback_ = cb; }
            void setErrorCallback(const EventCallback& cb)
            { errorCallback_ = cb; }
            
    2. Channel -> TcpConnection
            
            Channel::closeCallback_() -> TcpConnection::handleClose()
            Channel::errorCallback_() -> TcpConnection::handleError()
            Channel::readCallback_() -> TcpConnection::handleRead()
            Channel::writeCallback_() -> TcpConnection::handleWrite()
            
            TcpConnection::TcpConnection(EventLoop* loop,
                                         const string& nameArg,
                                         int sockfd,
                                         const InetAddress& localAddr,
                                         const InetAddress& peerAddr)
              : loop_(CHECK_NOTNULL(loop)),
                name_(nameArg),
                state_(kConnecting),
                socket_(new Socket(sockfd)),
                channel_(new Channel(loop, sockfd)),
                localAddr_(localAddr),
                peerAddr_(peerAddr),
                highWaterMark_(64*1024*1024),
                reading_(true)
            {
              channel_->setReadCallback(
                  std::bind(&TcpConnection::handleRead, this, std::placeholders::_1));
              channel_->setWriteCallback(
                  std::bind(&TcpConnection::handleWrite, this));
              channel_->setCloseCallback(
                  std::bind(&TcpConnection::handleClose, this));
              channel_->setErrorCallback(
                  std::bind(&TcpConnection::handleError, this));
              LOG_DEBUG << "TcpConnection::ctor[" <<  name_ << "] at " << this
                        << " fd=" << sockfd;
              socket_->setKeepAlive(true);
            }
            
    3. TcpConnection -> TcpClient()
        1. TcpConnection::handleClose() -> closeCallback_(guardThis); -> TcpClient::removeConnection
        - channel_->disableAll()  
        - connectionCallback_(guardThis);  
        - closeCallback_(guardThis);
        
                void TcpConnection::handleClose()
                {
                  loop_->assertInLoopThread();
                  LOG_TRACE << "fd = " << channel_->fd() << " state = " << stateToString();
                  assert(state_ == kConnected || state_ == kDisconnecting);
                  // we don't close fd, leave it to dtor, so we can find leaks easily.
                  setState(kDisconnected);
                  channel_->disableAll();
                
                  TcpConnectionPtr guardThis(shared_from_this());
                  connectionCallback_(guardThis);
                  // must be the last line
                  closeCallback_(guardThis);
                }

        2. TcpConnection::handleError()
        
                void TcpConnection::handleError()
                {
                  int err = sockets::getSocketError(channel_->fd());
                  LOG_ERROR << "TcpConnection::handleError [" << name_
                            << "] - SO_ERROR = " << err << " " << strerror_tl(err);
                }

        3. TcpConnection::handleWrite() -> writeCompleteCallback_()
        - sockets::write       
        - writeCompleteCallback_
        
                void TcpConnection::handleWrite() {
                  loop_->assertInLoopThread();
                  if (channel_->isWriting()) {
                    ssize_t n = sockets::write(channel_->fd(),
                                               outputBuffer_.peek(),
                                               outputBuffer_.readableBytes());
                    if (n > 0) {
                      outputBuffer_.retrieve(n);
                      if (outputBuffer_.readableBytes() == 0) {
                        channel_->disableWriting();
                        if (writeCompleteCallback_) {
                          loop_->queueInLoop(std::bind(writeCompleteCallback_, shared_from_this()));
                        }
                        if (state_ == kDisconnecting) {
                          shutdownInLoop();
                        }
                      }
                    }
                    else {
                      LOG_SYSERR << "TcpConnection::handleWrite";
                    }
                  } else {
                    LOG_TRACE << "Connection fd = " << channel_->fd()
                              << " is down, no more writing";
                  }
                }
            
        4. TcpConnection::handleRead() -> messageCallback()
        
        - inputBuffer_.readFd(channel_->fd(), &savedErrno);  
        - messageCallback_(shared_from_this(), &inputBuffer_, receiveTime);
        - handleClose();
        
                void TcpConnection::handleRead(Timestamp receiveTime) {
                  loop_->assertInLoopThread();
                  int savedErrno = 0;
                  ssize_t n = inputBuffer_.readFd(channel_->fd(), &savedErrno);
                  if (n > 0) {
                    messageCallback_(shared_from_this(), &inputBuffer_, receiveTime);
                  } else if (n == 0) {
                    handleClose();
                  } else {
                    errno = savedErrno;
                    LOG_SYSERR << "TcpConnection::handleRead";
                    handleError();
                  }
                }
                
                
 4. TcpClient connect的流程
    1. 创建 Connector 做实际的connect逻辑： connector_(new Connector(loop, serverAddr))
    2. 设置 Connector 的 newConnection 处理 Callback，Connector::newConnection 返回一个TcpConnection  
    对象。
    3. 在 newConnection 里面注册 TcpConnection的几个处理Callback：  
    - conn->setConnectionCallback(connectionCallback_);
    - conn->setMessageCallback(messageCallback_);
    - conn->setWriteCompleteCallback(writeCompleteCallback_);
    - conn->setCloseCallback(std::bind(&TcpClient::removeConnection, this, std::placeholders::_1));
    最后在newConnection的末尾通知 TcpConnection 对象，连接已经建立完成：
    - conn->connectEstablished();