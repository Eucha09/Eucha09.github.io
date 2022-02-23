---
title: "[서버] Interlocked"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-02-23 14:41:00 +0900
categories: [서버(Server), 멀티쓰레드 프로그래밍(Multi-thread programming)]
tags: [경쟁상태, Race Condition, Interlocked, 원자성, Atomic]     # TAG names should always be lowercase
---

## **경쟁 상태(Race Condition)**

경쟁 상태(Race Condition)란 두 개 이상의 프로세스 또는 쓰레드가 하나의 공유 자원에 동시에 접근하여 결과값에 영향을 줄 수 있는 상태를 말한다.

### **경쟁 상태(Race Condition) 예**

```c#
static int number = 0;

static void Thread_1()
{
    for (int i = 0; i < 100000; i++)
        number++;
}

static void Thread_2()
{
    for (int i = 0; i < 100000; i++)
        number--;
}

static void Main(String[] args)
{
    Task t1 = new Task(Thread_1);
    Task t2 = new Task(Thread_2);
    t1.Start();
    t2.Start();

    Task.WaitAll(t1, t2);

    Console.WriteLine(number);
}
```

위 코드를 보면 Thread_1에서는 number를 1씩 100000번 증가시켜주고 있고 Thread_2에서는 number를 1씩 100000번 감소해주고 있기 때문에 최종적으로 number는 0이 될것이라고 예상할 수 있다.

그러나 실행시켜보면 0이 아닌 다른 값이 나온다.
```console
24632
```

예상 결과와 다른 값이 나오는 이유는 두 쓰레드가 number라는 공유 자원에 동시에 접근하면서 Race Condition이 발생하였기 때문이다.

자세한 부분은 number++와 number--하는 과정에서 찾아볼 수 있다.   
우리가 사용한 number++이 실제 어셈블리어로 동작하는 과정을 보면 아래와 같다.
```console
00007FFDBB551513  mov         ecx,dword ptr [7FFDBB5EFC0Ch]  ; 데이터를 불러와서 ecx에 저장
00007FFDBB551519  inc         ecx  ; ecx에 있는 값 1증가
00007FFDBB55151B  mov         dword ptr [7FFDBB5EFC0Ch],ecx  ; 아까 불러온 위치에 ecx에 있는 값 다시 저장
```
```c#
//의사 코드
int temp = number;
temp = temp + 1;
number = temp;
```

우리가 볼 땐 한 줄짜리 코드지만 실제로는 위와 같이 처리된다.   
그렇기에 number++과 number--이 거의 동시에 실행하게 된다면
```console
// 초기 상태 number = 0
Thread_1 : int temp1 = number;  // temp1 = 0
Thread_1 : temp1 = temp1 + 1;   // temp1 = 1
Thread_2 : int temp2 = number;  // temp2 = 0
Thread_2 : temp2 = temp2 - 1;   // temp2 = -1
Thread_1 : number = temp1;      // number = 1
Thread_2 : number = temp2;      // number = -1
```
위와 같이 최종 number가 0이 아닌 값이 나올 수 있게 된다.

일단 위 문제는 Interlocked를 사용하여 해결해 볼 수 있다.

## Interlocked

Interlocked는 C#에서 지원해주고 있는 기능이며, 멀티쓰레드에서 공유하고 있는 변수에 대한 연산에 원자성(Atomic)을 보장해준다.   

여기서 원자성(Atomic)이란 더이상 쪼갤 수 없다라는 의미로 더 이상 과정이 쪼개지지 않고 한 번에 실행되는 것을 보장해주는 것을 말한다.

### Interlocked 사용법

```c#
static int number = 0;

static void Thread_1()
{
    for (int i = 0; i < 100000; i++)
        Interlocked.Increment(ref number); // 1 증가
}

static void Thread_2()
{
    for (int i = 0; i < 100000; i++)
        Interlocked.Decrement(ref number); // 1 감소
}

static void Main(String[] args)
{
    Task t1 = new Task(Thread_1);
    Task t2 = new Task(Thread_2);
    t1.Start();
    t2.Start();

    Task.WaitAll(t1, t2);

    Console.WriteLine(number);
}
```
```console
0
```

아까 코드에서 number++과 number-- 대신 Interlocked를 사용하여 1 증가, 감소를 해주고 있다.

앞에 문제에선 number++ 또는 number-- 하는 과정에서 다른 쓰레드가 끼어들어 Race Condition이라는 문제가 발생한 것인데, 여기에서는 Interlocked를 사용하여 원자성(Atomic)을 보장받고 있기 때문에 1 증가 또는 감소하는 과정에서 다른 쓰레드가 끼어들지 못해 결과에 문제없음을 볼 수 있다.
