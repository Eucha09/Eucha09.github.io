---
title: "[멀티쓰레드] AutoResetEvent"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-06-01 15:02:00 +0900
categories: [서버, 멀티쓰레드 프로그래밍]
tags: [Lock, AutoResetEvent, ManualResetEvent, Mutex]     # TAG names should always be lowercase
---

## **Lock 구현 이론**

[Lock 구현 이론](https://eucha09.github.io/posts/Lock%EA%B5%AC%ED%98%84%EC%9D%B4%EB%A1%A0/)

앞 포스트에서 Lock 구현 방법에 대해 얘기를 해보았다.   
대략 3가지 방법이 있었는데,

1. **그냥 무작정 기다린다.**
2. **일단 다른 작업으로 돌아가 있다가 나중에 다시 와본다.**
3. **Event call back 방식으로 누군가에게 다시 사용할 수 있는 상태가 되면 호출해 달라한다.**

1번 방법은 SpinLock이라고 부른다고 하였고 이전 포스트에서 직접 구현해보기도 하였다.   
[SpinLock](https://eucha09.github.io/posts/SpinLock/)

2번 방법은 그냥 무작정 기다리는것이 아닌 일단 CPU 소유권을 양보하였다가 나중에 다시 시도하는 방식이다. 이또한 Context Switching을 배워보면서 간단하게 구현해보았다.   
[Context Switching](https://eucha09.github.io/posts/ContextSwitching/)

3번 방법은 커널(운영체제)한테 Event call back 방식으로 나중에 다시 사용가능해지면 호출해달라고 부탁해놓고 나오는 방식인데 이번 포스트에서 AutoResetEvent를 사용해 구현해볼 것이다.

## **AutoResetEvent**

C#에서는 AutoResetEvent 기능이 있는데 쉽게 말해 Thread.Sleep()와 같이 쉬고 있다가 다른 쓰레드가 신호를 보내면 깨어나 다음 문장을 실행할 수 있도록 해주는 기능이다.

이를 이용해 누군가가 이미 Lock을 점유하고 있다면 WaitOne()을 호출해 잠들어있다가 누군가가 Set()을 호출해 신호를 준다면 그때 깨어나 자신이 Lock을 점유하면 된다.

AutoResetEvent를 이용해 Lock을 구현해보면 아래와 같다.

```cs
class Lock
{
    // bool 변수라고 생각하면 편하다. <- 커널
    // bool _available = true;
    AutoResetEvent _available = new AutoResetEvent(true);

    public void Enter()
    {
        _available.WaitOne(); // true일때까지 기다린다.
        //_available.Reset(); // _available = false, AutoResetEvent는 자동으로 해준다.
    }

    public void Exit()
    {
        _available.Set(); // _available = true;
    }
}
```

AutoResetEvent 사용법은 하나의 bool 변수라고 생각하면 편하다. 다만 bool값을 커널에서 관리한다.

코드를 살펴보자면 입장하려는데 false라면 true로 바뀔때까지 기다리고 true이면 입장하면서 false로 바꾼다(다른 쓰레드가 못들어오도록). 그리고 나갈 때는 true로 다시 바꾼다.

원래 Reset()이라고 false로 바꾸는 함수가 있는데 AutoResetEvent는 자동으로 해준다.

Lock을 구현했으니 간단하게 테스트를 해보자.

```cs
class Program
{
    static int _num = 0;
    static Lock _lock = new Lock();

    static void Thread_1()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.Enter(); // 자물쇠로 잠군다. (혹시나 잠겨 있으면 기다린다.)

            _num++;

            _lock.Exit(); // 잠금을 푼다.
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.Enter(); // 자물쇠로 잠군다. (혹시나 잠겨 있으면 기다린다.)

            _num--;

            _lock.Exit(); // 잠금을 푼다.
        }
    }

    static void Main(String[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1, t2);

        Console.WriteLine(_num);
    }
}
```
```console
0
```

실행해보면 정상적으로 0이 잘 출력되는 것을 볼 수 있다. 다만 좀 느리다는 것을 느낄 수 있을 것이다.   
AutoResetEvent가 커널단에서 실행되다보니 좀 느리다는 단점이 있다. 그렇기에 위처럼 Lock을 걸고 하는 작업이 오래걸리지 않는다면 SpinLock처럼 그냥 while문을 돌면서 기다리는게 나을 수 있고 만약 Lock을 걸고 하는 작업이 오래걸리는 작업이라면 위처럼 커널한테 신호보내달라는 식으로 맡기는 방법이 나을 수 있다.

AutoResetEvent는 꼭 Lock에서 말고도 사용될 수 있으니 알아두면 좋다.

추가적으로 AutoResetEvent와 비슷한 ManualResetEvent와   
이러한 Event말고도 비슷하게 사용할 수 있는 Mutex에 대해 간단히 알아보려고 한다.

## **ManualResetEvent**

ManualResetEvent또한 AutoResetEvent와 거의 동일한 기능을 하지만 한가지 다른 점은 WaitOne()을 하였을 때 Reset()을 자동으로 안해준다는 점이다.

그래서 AutoResetEvent 대신 ManualResetEvent를 사용한다면 아래와 같이 Reset()을 넣어주어야 한다.

```cs
class Lock
{
    // bool 변수라고 생각하면 편하다. <- 커널
    // bool _available = true;
    ManualResetEvent _available = new ManualResetEvent(true);

    public void Enter()
    {
        _available.WaitOne(); // true일때까지 기다린다.
        _available.Reset(); // _available = false, ManualResetEvent는 자동으로 안해준다.
    }

    public void Exit()
    {
        _available.Set(); // _available = true;
    }
}
```

다만 ManualResetEvent를 사용하여 Lock을 구현했을 때 테스트를 해보면 0이 안나오는 것을 볼 수 있다.

```cs
class Program
{
    static int _num = 0;
    static Lock _lock = new Lock();

    static void Thread_1()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.Enter(); // 자물쇠로 잠군다. (혹시나 잠겨 있으면 기다린다.)

            _num++;

            _lock.Exit(); // 잠금을 푼다.
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.Enter(); // 자물쇠로 잠군다. (혹시나 잠겨 있으면 기다린다.)

            _num--;

            _lock.Exit(); // 잠금을 푼다.
        }
    }

    static void Main(String[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1, t2);

        Console.WriteLine(_num);
    }
}
```
```console
9
```

이러한 결과가 나오는 이유는 Lock에서 기다리고 잠구는 작업을 2단계로 나누어서 처리를 하고 있기 때문이다. 그렇기에 ManualResetEvent로 Lock을 구현하기에는 적합하지 않다라고 볼 수 있다.

굳이 Lock이 아니여도 이러한 EventCallback 방식은 다른 곳에 쓰일 때도 있으니 차이만 알아두자.

마지막으로 이러한 Event말고도 비슷하게 사용할 수 있는 mutex를 알아보자.

## **Mutex**

Mutex또한 Lock의 기능을 가지고 있기에 Lock과 동일하게 사용할 수 있다.

간단하게 사용해보자면 아래와 같이 사용해 볼 수 있다.

```cs
class Program
{
    static int _num = 0;
    static Mutex _lock = new Mutex(); // Mutex 사용

    static void Thread_1()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.WaitOne(); // 자물쇠로 잠군다. (혹시나 잠겨 있으면 기다린다.)

            _num++;

            _lock.ReleaseMutex(); // 잠금을 푼다.
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.WaitOne(); // 자물쇠로 잠군다. (혹시나 잠겨 있으면 기다린다.)

            _num--;

            _lock.ReleaseMutex(); // 잠금을 푼다.
        }
    }

    static void Main(String[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);
        t1.Start();
        t2.Start();

        Task.WaitAll(t1, t2);

        Console.WriteLine(_num);
    }
}
```
```console
0
```

사용법은 우리가 구현한 Lock과 거의 동일하게 사용해볼 수 있고, 실행시켜보면 정상적으로 0이 나오는것을 볼 수 있다.

그러나 Mutex는 앞에서 Event로 구현한 Lock보다 더 느린것을 느낄 수 있다.   
AutoResetEvent와 동일하게 커널까지 내려가서 실행되며, AutoResetEvent의 경우 커널에서 사실상 bool 값 하나를 가지고 처리하지만 Mutex의 경우 누가 잠구었는지, 몇번 잠구었는지 등 좀 더 많은 정보를 가지고 처리하기 때문에 좀 더 느린것이다.
