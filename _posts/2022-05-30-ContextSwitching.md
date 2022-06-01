---
title: "[멀티쓰레드] Context Switching"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-05-30 15:51:00 +0900
categories: [서버, 멀티쓰레드 프로그래밍]
tags: [Lock, Context Switching]     # TAG names should always be lowercase
---

## **Context Switching**

**Context Switching**이란 하나의 프로세스(또는 쓰레드)가 CPU를 사용 중인 상태에서 다른 프로세스(또는 쓰레드)가 CPU를 사용하도록 교체하는 작업을 말한다. 이 과정속에서 이전의 프로세스의 상태를 보관하고 새로운 프로세스의 상태를 적재하는 작업이 이루어진다.

보통 일반적으로 운영체제가 각 프로세스한테 실행 시간(timeslice)을 배정해서, 그 시간만큼 프로세스가 실행되고, 시간이 끝나면 CPU 소유권을 반납하여 다른 프로세스가 소유권을 받아 실행되도록 한다(Context Switching).

근데 만약 CPU 소유권을 받았는데 딱히 할일이 없다면 미연에 소유권을 받납할 수도 있다.

앞 포스트에서의 SpinLock을 예로 들어보자면 SpinLock의 경우 Lock을 얻지못했다면 얻을 때까지 계속 루프를 돌며 재시도를 하게 된다. 이때 만약 Lock을 획득한 쓰레드가 꽤나 오래걸리는 작업을 하고 있다고 한다면 Lock을 기다리고 있는 쓰레드는 계속 무한루프를 돌면서 기다리게 된다.

[SpinLock에 대한 내용](https://eucha09.github.io/posts/SpinLock/)

이때 무작정 기다리고 있는것 보단 CPU 소유권을 다른 쓰레드에게 넘겨주어 기다리는 동안 다른 작업을 하고 있도록 한다면 나름 효율적이지 않을까라는 생각을 할 수 있을 것이다.

언어마다 다르겠지만 C#의 경우 Thread.Sleep()와 Thread.Yield()를 사용해볼 수 있다.

## **Thread.Sleep(), Thread.Yield()**

크게 CPU 소유권을 양보하는 방법으로 Thread.Sleep()와 Thread.Yield()로 나누어 볼 수 있지만 Thread.Sleep()또한 인자로 양수를 전달하는 경우와 그냥 0을 전달하는 경우로 나누어볼 수 있기에 크게 3가지 방법이 있다.   
일단 3가지 방법을 비교해보자면 아래와 같다.

```cs
Thread.Sleep(1) // 무조건 1ms 정도 쉰다. 쉬는 동안 다른 쓰레드한테 양보
Thread.Sleep(0) // 나보다 우선순위가 같거나 높은 쓰레드가 있다면 양보
Thread.Yield(); // 지금 실행가능한 쓰레드가 있으면 양보
```

약간의 차이는 있지만 기본적으로 자기가 할당받은 실행 시간(timeslice)을 반납하면서 다른 프로세스(또는 쓰레드)한테 양보한다는 것은 똑같다.   
사실 어느것을 선택해도 큰 차이는 없어 아무거나 사용해도 무방하다. 다만 이러한 차이가 있다는 것은 알아두면 좋겠다.

이전 포스트의 SpinLock 코드에서 이를 사용해본다면 아래와 같이 사용해 볼 수 있다.

```cs
class SpinLock
{
    int _locked = 0;

    public void Enter()
    {
        while (true)
        {
            // Lock 획득 시도
            if (Interlocked.CompareExchange(ref _locked, 1, 0) == 0)
                break;

            // Lock 획득을 실패했다면 나중에 다시 시도
            Thread.Yield(); // CPU 소유권 양보
        }
    }

    public void Exit()
    {
        _locked = 0;
    }
}
```

사실 위 코드는 SpinLock 개념을 벗어났기 때문에 SpinLock이라고 부르진 못하겠지만 이또한 일종의 Lock 구현하는 방법 중 하나라고 할 수 있겠다.

다만 여기서 꼭 유의할 점은 CPU 소유권을 양보한다는 것이 결코 가볍게 이루어지는 작업이 아니라는 점이다. Context Switching이 이루어지는 과정속에서 현재 프로세스(또는 쓰레드)에 대한 정보를 메모리에 저장하고, 다른 프로세스(또는 쓰레드)에 대한 정보를 메모리에서 불어오는 등 이런저런 복잡한 작업들을 하기 때문에 Context Switching이 많이 발생하면 CPU입장에서는 꽤나 부담이 될 수 있다. 그렇기에 경우에 따라 기존 SpinLock처럼 계속 루프를 돌면서 시도하는것이 오히려 효율적일 수도 있다.
