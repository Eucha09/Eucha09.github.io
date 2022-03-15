---
title: "[ASP.NET Core] Hello WebAPI"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-03-07 14:33:00 +0900
categories: [웹 서버, ASP.NET Core 둘러보기]
tags: [WebAPI]     # TAG names should always be lowercase
---

## **WebAPI**

**WebAPI**는 **웹 서버 또는 웹 브라우를 위한 인터페이스**를 말한다.

WebAPI 프로젝트를 MVC 구조와 비교하자면 Model-View-Controller에서 View가 빠진 느낌이라고 생각하면 이해하기 편하다.

## **WebAPI 프로젝트 생성**

Visual Studio에서 'ASP.NET Core 웹 애플리케이션 > API'를 선택해 WebAPI 프로젝트를 생성해 볼 수 있다.

이대로 컴파일하여 실행시켜보면 아래와 같이 뜰 것이다.

![HelloWebAPI](/assets/img/posts/webserver/HelloWebAPI.png){: w="800"}
_ASP.NET Core WebAPI 실행 화면_

웹페이지 화면이라기엔 이상하게 보일 수 있는데 정상적으로 실행된 화면이다.

이것이 MVC, Razor Pages와 webAPI의 차이점으로 View가 따로 없다는 것을 알 수 있다.

## **빈 프로젝트에서 시작**

WebAPI 템플릿을 이용하여 프로젝트를 만들어도 되지만 WebAPI 프로젝트 구조 및 동작원리를 이해하기 위해 빈 프로젝트에서부터 만들어보겠다.

### **빈 프로젝트 생성**

'새 프로젝트 생성 > ASP.NET Core 웹 애플리케이션 > 비어 있음'을 선택해 빈 프로젝트를 생성한다.

빈 프로젝트가 생성 되었다면 Models와 Controllers 폴더를 생성한다.   
Models에서는 데이터들을 정의 및 관리하고 Controllers에서는 데이터 요청 및 응답에 대한 처리를 할 것이다.
* Models
* Controllers
* Program.cs
* Startup.cs

### **Models**

먼저 간단한 메세지를 표현할 데이터를 정의해주자.   
Models 폴더에서 '새 항목 추가 > 클래스 > 파일이름 HelloMessage.cs'를 생성하고 아래와 같이 정의한다.
```cs
public class HelloMessage
{
    public string Message { get; set; }
}
```

### **Controllers**

그 다음 간단한 메세지를 전달할 Controller를 생성해보자.
Controllers 폴더에 '새 항목 추가 > API 컨트롤러 클래스 - 비어있 > 파일이를 ValuesController.cs'를 생성해준다.

ValuesController.cs
```cs
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{

}
```

코드를 작성하기전 먼저 첫 번째 줄을 살펴보면 라우팅 설정을 해주고 있는 것을 볼 수 있다.

WebAPI 프로젝트에서는 이렇게 각 Controller에서 라우팅 설정을 해주고 있다.

MVC, Razor Pages 프로젝트와 비교해보자면   
MVC 프로젝트에서는 Startup.cs에서 pattern을 설정해 매핑시켜주고 있었고,   
Razor Pages 프로젝트에서는 pattern을 따로 설정해주진 않았지만 알아서 Pages 폴더 안에 있는 페이지와 매핑시켜주었다.

마저 ValuesController.cs를 작성해보자.
```cs
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet]
    public List<HelloMessage> Get()
    {
        List<HelloMessage> messages = new List<HelloMessage>();
        messages.Add(new HelloMessage() { Message = "Hello Web API 1 ! " });
        messages.Add(new HelloMessage() { Message = "Hello Web API 2 ! " });
        messages.Add(new HelloMessage() { Message = "Hello Web API 3 ! " });

        return messages;
    }
}
```

HttpGet 방식으로 요청이 들어올 경우 Get()이 호출돼 메세지들을 반환해주게 된다.

### **Startup**

마지막으로 Startup.cs에서 Controllers들을 연결시켜주고 실행시켜보자.

```cs
public class Startup
{
    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers(); // Controllers들을 사용하겠다는 의미
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers(); // Controllers들을 연결시켜주겠다는 의미
        });
    }
}
```

실행시켜보면 아래와 같이 뜰 것이다.

![HelloWebAPI1](/assets/img/posts/webserver/HelloWebAPI1.png){: w="800"}
_첫 실행 화면_

잘못된것이 아니고 WebAPI에서는 기본적인 Index라는 View가 없기 때문이다.

아까 작성한 Controller가 잘 동작하는지 확인해보기 위해서는 Controller에서 라우팅 설정을 했던 부분을 생각해보면 된다.

ValuesController.cs
```cs
[Route("api/[controller]")]
```

"api/[controller]" 라고 설정되어 있으니 다시 웹페이지화면에서 주소 옆에 'api/values'라고 입력해보자.

![HelloWebAPI2](/assets/img/posts/webserver/HelloWebAPI2.png){: w="800"}
_api/values 화면_

일단 아까 작성한 메세지로 보이는 데이터가 뜰 것이다.
좀 더 정확하게 확인해보려면 F12를 눌러 네트워크에 들어가 방금 페이지를 새로고침해보면 어떠한 데이터들이 오고 있는지 확인해볼 수 있다.   
또한 지금은 데이터들이 json 형식으로 오고 있다는 것을 알 수 있다.

![HelloWebAPI3](/assets/img/posts/webserver/HelloWebAPI3.png){: w="800"}
_F12로 데이터 전송 확인_

WebAPI는 이런식으로 데이터만 전송해주는 역할을 하기 때문에 따로 View가 필요없고 클라이언트에서는 이러한 데이터를 받아 UI와 연동해 표현해주면 된다.   

웹서버에서 View 정보까지 왜 굳이 전송할 필요가 없는지에 대해서는 예를 들어 유니티와 연동한다고 하였을 때 유니티에서는 어짜피 프리팹화 되어 있는 UI들로 표현해야하기 때문에 html 태그로 이루어진 정보가 필요 없고, 웹 서비스를 개발한다 해도 어떻게 표현해야한다는 강압적인 규칙이 따로 없다보니 클라이언트에서는 데이터들만 받아 자유롭게 표현할 수 있게 된다. 그렇기에 WebAPI는 좀 더 범용적으로 데이터들을 활용할 수 있다는 장점이 있다.
