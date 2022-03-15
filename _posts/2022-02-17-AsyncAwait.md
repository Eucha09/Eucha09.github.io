---
title: "[고급 C#] Async, Await"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-02-17 14:36:00 +0900
categories: [웹 서버, "고급 C# 문법"]
tags: [동기, 비동기, Async, Await]     # TAG names should always be lowercase
---

## **동기(Synchronous)와 비동기(Asynchronous)**

**동기(Synchronous)**란 동시에 일어난다는 의미로 한자리에서 요청과 동시에 결과가 일어난다. 그렇기에 **결과가 발생해야지만 다음 동작을 수행할 수 있다.**

**비동기(Asynchronous)**란 동시에 일어나지 않는다는 의미로 요청과 결과가 한자리에서 동시에 일어나지 않을 수 있다. 그렇기에 **결과가 아직 주어지지 않더라도 다음 동작을 수행할 수 있다.**

비동기적으로 처리할 경우 병렬적으로 처리할 수 있다는 장점이 있다.

비동기가 멀티쓰레드와 비슷하게 동작하다보니 같은 것이라고 생각할 수 있는데 비동기와 멀티쓰레드는 다른 개념이다. 비동기라고 꼭 멀티쓰레드로 동작하는 것은 아니다.   
예를 들자면 유니티에서의 코루틴(Coroutine) 또한 일종의 비동기라고 할 수 있지만 싱글쓰레드로 동작한다.

## **Async, Await**

C#에서는 async와 await를 사용하여 비동기 프로그래밍을 할 수 있다.

async와 await를 사용하기전 아래 코드를 실행시켜보자.

```c#
static void TestAsync()
{
    Console.WriteLine("TestAsync start");

    Task t = Task.Delay(3000); // 복잡한 작업 또는 오래 기다려야 하는 작업
    t.Wait();

    Console.WriteLine("TestAsync end");
}

static void Main(String[] args)
{
    TestAsync();

    Console.WriteLine("while start");

    while(true)
    {

    }
}
```
```console
TestAsync start
TestAsync end
while start
```

TestAsync start가 출력된 후 3초 후에 TestAsync end와 while start가 순서대로 출력된다.   
Main함수에서 TestAsync()를 호출하면 TestAsync 함수 내의 모든 작업을 수행한 후에 Main함수로 넘어온다라는 것을 알 수 있다.

만약 TestAsync 함수 내의 작업이 매우 오래 기다려야하는 작업이라면 그 작업이 완료될 때까지 Main함수에서는 기다리기만 해야 하고 다른 작업으로는 넘어가지 못한다.   
기다리는 동안 다른 작업을 하고 있을 수 있다면 효율적일 것이라고 생각이 들것이다. 이때 할 수 있는 것이 비동기 프로그래밍이다.

C#에서는 async와 await를 사용해 비동기 프로그래밍을 할 수 있으며 위 코드에 적용시켜보면 아래와 같다.

```c#
static async Task TestAsync()
{
    Console.WriteLine("TestAsync start");

    // await의 오른쪽 작업이 끝날 때까지 기다리지만 코드 흐름은 밖으로 넘긴다.
    await Task.Delay(3000); // 복잡한 작업 또는 오래 기다려야 하는 작업

    Console.WriteLine("TestAsync end");
}

static void Main(String[] args)
{
    Task t = TestAsync();

    Console.WriteLine("while start");

    while(true)
    {

    }
}
```
```console
TestAsync start
while start
TestAsync end
```

TestAsync start와 while start가 순서대로 출력되고 3초 후 TestAsync end가 출력된다.   
TestAsync함수 내의 작업이 다 끝나지 않았음에도 Main함수에서는 다음 작업으로 넘어간 것을 알 수 있다.

await의 특징은 await의 오른쪽 작업이 끝날 때까지 기다리긴 하지만 코드 흐름은 밖으로 보내 그 동안 다른 처리를 할 수 있도록 해준다.   

그러면 작업을 완료한 후에 다음 작업은 누가 실행시켜주는 것일까

await 밑에 중단점을 찍어 디버깅을 해보면 알 수 있다.

![async디버깅](/assets/img/posts/webserver/async디버깅.png){: w="700"}
_중단점 지정_
![async디버깅2](/assets/img/posts/webserver/async디버깅2.png){: w="500"}
_쓰레드 확인_

여기에서는 작업자 쓰레드가 생성되어 실행해주고 있는 것을 알 수 있다.   
위 경우에서는 멀티쓰레드로 동작하고 있긴하지만 꼭 모든 비동기가 멀티쓰레드로 동작하는 것은 아니다. 싱글 쓰레드가 다시 돌아와서 실행시켜 주는 경우도 있다.

아까는 TestAsync함수에서만 await를 해주었는데 Main에서도 await를 해주면 어떻게 될까

```c#
static async Task TestAsync()
{
    Console.WriteLine("TestAsync start");

    await Task.Delay(3000); // 복잡한 작업 또는 오래 기다려야 하는 작업

    Console.WriteLine("TestAsync end");
}

static async Task Main(String[] args)
{
    await TestAsync();

    Console.WriteLine("while start");

    while(true)
    {

    }
}
```
```console
TestAsync start
TestAsync end
while start
```

TestAsync start가 출력되고 3초후에 TestAsync end와 while start가 순서대로 출력된다.   
await 동작방식을 이해하였다면 충분히 해석할 수 있다.

추가적으로 async와 await를 사용할 때 경우에 따라 데이터를 반환에 줄 수 도 있다.

```c#
static async Task<int> TestAsync()
{
    Console.WriteLine("TestAsync start");

    await Task.Delay(3000);

    Console.WriteLine("TestAsync end");

    return 1; // 결과 값 반환
}

static async Task Main(String[] args)
{
    int ret = await TestAsync();

    Console.WriteLine(ret); // 결과 값 출력
    Console.WriteLine("while start");

    while(true)
    {

    }
}
```
```console
TestAsync start
TestAsync end
1
while start
```

## **Async, Await 참고**

Microsoft 문서를 보면 Async, Await에 대해 참고하기 좋다.   
[https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)

위 문서에서는 아침 식사를 만드는 과정을 비유하여 예제를 보여주고 있다.

결과부터 말하자면,

아침식사를 만드는 과정이 아래와 같다고 했을 때
1. Pour a cup of coffee.
2. Heat up a pan, then fry two eggs.
3. Fry three slices of bacon.
4. Toast two pieces of bread.
5. Add butter and jam to the toast.
6. Pour a glass of orange juice.

동기적으로 처리할 경우

![동기적처리](/assets/img/posts/webserver/synchronous-breakfast.png){: w="300"}
_출처:[동기적 처리](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)_

30분이 걸리는 데 생각해보면 비효율적이라는 것을 알 수 있다.   
pan 데워지기를 기다리는 동안 바로 eggs와 bacon을 올리면되고 eggs와 bacon이 익기를 기다리는 동안 식빵을 토스트기에 넣어두면 되기 때문이다.

이부분에 대해서 비동기적으로 처리할 경우

![비동기적처리](/assets/img/posts/webserver/whenany-async-breakfast.png){: w="800"}
_출처:[비동기적 처리](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/)_

15분만에 처리할 수 있다는 것을 보여주고 있다.
