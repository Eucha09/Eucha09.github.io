---
title: "[웹 서버] Hello Blazor Client + SPA"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-03-09 13:00:00 +0900
categories: [웹 서버(Web Server), ASP.NET Core 둘러보기]
tags: [Blazor, Blazor Client, SPA]     # TAG names should always be lowercase
---

## **Blazor**

**Blazor**는 Microsoft에서 개발한 **오픈소스 웹 개발 프레임워크**이다. 가장 큰 특징으로는 **.NET Core 기반의 C#으로 웹 앱을 개발할 수 있다.**

Blazor는 **SPA(Single Page Application)** 방식을 사용하고 서버측에서 동작하는 **Blazor Server**와 클라이언트측에서 동작하는 **Blazor Client(WebAssembly)** 형식이 있다.

## **SPA(Single Page Application)**

**SPA(Single Page Application)**는 **서버로부터 완전한 새로운 페이지를 불러오지 않고 현재의 페이지를 동적으로 다시 작성**함으로써 사용자와 소통하는 웹 애플리케이션을 말한다.

![SPA](/assets/img/posts/webserver/SPA.png){: w="500"}
_SPA_

이전의 전통적인 방식에서는 처음 웹홈페이지에 접속하면 서버에서는 그에 맞는 HTML 정보를 전달해준다. 그러고 다른 페이지에 들어간다거나 하면 서버에 요청해 또 그에 맞는 HTML 정보를 받는다. 그러다보니 매번 Page Reload가 발생하게 된다.

예를 들어 네이버 웹툰에 들어가 아무 메뉴나 눌러보면 매번 페이지 자체가 Reload되는 것을 볼 수 있다.   
![Page Reload](/assets/img/posts/webserver/PageReload.png){: w="400"}
_[네이버 웹툰](https://comic.naver.com/index) Page Reload_

그러나 SPA 방식에서는 처음 접속할때만 서버로부터 필요한 HTML 정보들을 받고 이후에는 AJAX를 통해 데이터만 요청하고 받아 클라이언트에서 렌더링하여 사용자에게 보여준다.

* AJAX(Asynchronous Javascript And XML) - Javascript 비동기 통신, XML 데이터를 주고 받는 기술

Blazor Server 프로젝트를 생성해 실행시켜보면 차이를 볼 수 있다.   
![Blazor SPA](/assets/img/posts/webserver/Blazor_FetchData.png){: w="800"}
_[Blazor Server 프로젝트 생성](https://eucha09.github.io/posts/HelloBlazorServer/)_
이런저런 메뉴들을 아무리 눌러봐도 Page Reload는 발생하지 않는다.

## **Blazor Server(Server Side Blazor)**

Blazor Server는 ASP.NET Core 서버와 Razor 엔진을 이용한 에디션으로, 서버에서 대부분의 렌더링과 프로세싱이 이루어지는 것이 특징이다. 따라서 클라이언트의 부담이 적으며, 서버 쪽에서 이루어지는 업데이트들을 클라이언트 쪽에서 SignalR 기반의 웹소켓 방식으로 수신받는다.

![Blazor Server](/assets/img/posts/webserver/blazor-server.png){: w="500"}
_출처:[Blazor Server](https://docs.microsoft.com/ko-kr/aspnet/core/blazor/?view=aspnetcore-6.0)_
* DOM(Document Object Model) - 문서 객체 모델(HTML, XML 문서의 프로그래밍 interface), 우리가 작성한 HTML도 브라우저에 의해 파싱되어 DOM으로 변환된다.

## **Blazor Client(Client side Blazor, WebAssembly)**

Blazor WebAssembly라고 불리기도 하는데, Blazor Server 에디션과 반대로 클라이언트에서 대부분의 렌더링이 이루어진다. 따라서 서버의 부담은 감소하지만 클라이언트의 부담이 높아지는 동시에 클라이언트가 처음 내려받는 파일의 용량도 증가한다. 하지만 서버를 거치지 않고 클라이언트에서 모든 로직이 실행되기 때문에 일반적인 윈도우 어플리케이션에 준하는 반응속도와 풍부한 사용자 경험을 제공할 수 있다는 장점이 있다.

쉽게 말해 처음 접속할 때 필요한 모든 것들을 로드받고 이후에는 서버와 어떠한 통신도 하지 않으면서 알아서 혼자 잘 동작한다.

![Blazor WebAssembly](/assets/img/posts/webserver/blazor-webassembly.png){: w="500"}
_출처:[Blazor Client](https://docs.microsoft.com/ko-kr/aspnet/core/blazor/?view=aspnetcore-6.0)_

## **Blazer Client 프로젝트 생성 및 실행**

Visual Studio에서 'Blazor 앱 > Blazor WebAssembly App'을 선택해 생성해 볼 수 있다.

이대로 컴파일을 하여 실행시켜보면 아래와 같이 뜰 것이다.

![HelloBlazorClient](/assets/img/posts/webserver/HelloBlazorClient.png){: w="800"}
_Blazor Client 실행 화면_

Blazor Server 프로젝트와 동일하게 뜨는 것을 볼 수 있으며 프로젝트 구조 또한 비슷한 형태를 띄고 있는 것을 볼 수 있다. 다만 Startup.cs파일이 없는 등 약간의 차이는 있다.

Blazor Client는 위에서도 설명했듯이 처음 접속할 때말곤 서버와 통신하지 않고 클라이언트상에서만 동작하기 때문에 브라우저로 접속해 있는 상태에서 서버를 종료시켜도 잘 동작하는 것을 볼 수 있다.

Blazor Server의 경우 중간에 서버를 종료시키면 브라우저상에서 바로 연결이 끊기는 현상을 관찰해볼 수 있다.

![Blazor Server 연결끊김](/assets/img/posts/webserver/BlazorServer연결끊김.png){: w="800"}
_Blazor Server 연결 끊김_
