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
        
        void EventLoop::loop()
        {
          assert(!looping_);
          assertInLoopThread();
          looping_ = true;
          quit_ = false;  // FIXME: what if someone calls quit() before loop() ?
          LOG_TRACE << "EventLoop " << this << " start looping";
        
          while (!quit_)
          {
            activeChannels_.clear();
            pollReturnTime_ = poller_->poll(kPollTimeMs, &activeChannels_);
            ++iteration_;
            if (Logger::logLevel() <= Logger::TRACE)
            {
              printActiveChannels();
            }
            // TODO sort channel by priority
            eventHandling_ = true;
            for (ChannelList::iterator it = activeChannels_.begin();
                it != activeChannels_.end(); ++it)
            {
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
> 会负责监控此fd的的 POLLIN / POLLOUT / POLLHUP 等event：PollPoller::updateChannel

        void PollPoller::updateChannel(Channel* channel)
        {
          Poller::assertInLoopThread();
          LOG_TRACE << "fd = " << channel->fd() << " events = " << channel->events();
          if (channel->index() < 0)
          {
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
          }
          else
          {
            // update existing one
            assert(channels_.find(channel->fd()) != channels_.end());
            assert(channels_[channel->fd()] == channel);
            int idx = channel->index();
            assert(0 <= idx && idx < static_cast<int>(pollfds_.size()));
            struct pollfd& pfd = pollfds_[idx];
            assert(pfd.fd == channel->fd() || pfd.fd == -channel->fd()-1);
            pfd.events = static_cast<short>(channel->events());
            pfd.revents = 0;
            if (channel->isNoneEvent())
            {
              // ignore this pollfd
              pfd.fd = -channel->fd()-1;
            }
          }
        }
