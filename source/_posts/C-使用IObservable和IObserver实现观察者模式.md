---
title: C#使用IObservable和IObserver实现观察者模式
date: 2021-08-20 13:14:33
tags:
- DotNet
- CSharp
- 发布订阅
categories: 
- 发布订阅
---

## 被观察者 `IObservable<string>`

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Utils
{
    /// <summary>
    /// 消息- 被观察者，提供订阅接口
    /// </summary>
    public class MsgTracker : IObservable<string>
    {
        public MsgTracker()
        {
            observers = new List<IObserver<string>>();
        }

        private List<IObserver<string>> observers;

        public IDisposable Subscribe(IObserver<string> observer)
        {
            if (!observers.Contains(observer))
                observers.Add(observer);
            return new Unsubscriber(observers, observer);
        }

        // 用于取消订阅通知的IDisposable对象的实现
        private class Unsubscriber : IDisposable
        {
            private List<IObserver<string>> _observers;
            private IObserver<string> _observer;

            public Unsubscriber(List<IObserver<string>> observers, IObserver<string> observer)
            {
                this._observers = observers;
                this._observer = observer;
            }

            public void Dispose()
            {
                if (_observer != null && _observers.Contains(_observer))
                    _observers.Remove(_observer);
            }
        }

        public void TrackMsg(string msg)
        {
            foreach (var observer in observers)
            {
                if (string.IsNullOrWhiteSpace(msg))
                    observer.OnError(new MsgUnknownException());
                else
                    observer.OnNext(msg);
            }
        }
        public void EndMsg()
        {
            foreach (var observer in observers.ToArray())
                if (observers.Contains(observer))
                    observer.OnCompleted();

            observers.Clear();
        }

    }
    public class MsgUnknownException : Exception
    {
        internal MsgUnknownException()
        { }
    }
}
```
<!--more-->

## 观察者 `IObserver<string>`

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Utils
{
    /// <summary>
    /// 消息发布者
    /// </summary>
    public class MsgReporter : IObserver<string>
    {
        private IDisposable unsubscriber;
        private Action<string> _next;

        public MsgReporter(Action<string> next)
        {
            this._next = next;
        }
         
        public virtual void Subscribe(IObservable<string> provider)
        {
            if (provider != null)
                unsubscriber = provider.Subscribe(this);
        }
        // 取消订阅
        public virtual void Unsubscribe()
        {
            unsubscriber.Dispose();
        }

        public virtual void OnCompleted()
        {
            this.Unsubscribe();
        }

        public virtual void OnError(Exception e)
        {
        }

        public virtual void OnNext(string value)
        {
            _next(value);
        }
    }
}
```

## 使用

```cs
MsgTracker msgTracker = new MsgTracker();

MsgReporter reporter1 = new MsgReporter(msg =>
{
    this.richTextBox1.AppendText(msg);
});

msgTracker.Subscribe(reporter1);
msgTracker.TrackMsg($"时间：{DateTime.Now:HH:mm:ss}-->{str}");
```

参考：

[IObservable<T> 接口](https://docs.microsoft.com/zh-cn/dotnet/api/system.iobservable-1?view=net-5.0)