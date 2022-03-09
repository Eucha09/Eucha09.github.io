---
title: "[웹 서버] Hello Blazor Server"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-03-07 17:56:00 +0900
categories: [웹 서버(Web Server), ASP.NET Core 둘러보기]
tags: [Blazor, Blazor Server, Razor Component, Bootstrap]     # TAG names should always be lowercase
---

## **Blazor**

**Blazor**는 Microsoft에서 개발한 **오픈소스 웹 개발 프레임워크**이다. 가장 큰 특징으로는 **.NET Core 기반의 C#으로 웹 앱을 개발할 수 있다.**

Blazor에서는 Blazor Server와 Blazor Client(WebAssembly) 형식이 있다.
 
### **Blazor Server**

Blazor Server는 ASP.NET Core 서버와 Razor 엔진을 이용한 에디션으로, 서버에서 대부분의 렌더링과 프로세싱이 이루어지는 것이 특징이다.

### **Blazor Client(WebAssembly)**

Blazor WebAssembly라고 불리기도 하는데, Blazor Server 에디션과 반대로 클라이언트에서 대부분의 렌더링이 이루어진다.

## **Blazor Server 프로젝트 생성**

Visual Studio에서 'Blazor 앱 > Blazor 서버 앱'을 선택해 Blazor Server 프로젝트를 생성해 볼 수 있다.

이대로 컴파일을 하여 실행시켜보면 아래와 같이 뜰 것이다.

![HelloBlazorServer](/assets/img/posts/webserver/HelloBlazorServer.png){: w="800"}
_Blazor Server 실행 화면_

## **Blazor Server 살펴보기**

위에서 생성한 Blazor Server 프로젝트의 코드를 살펴보면서 전체적인 동작 흐름을 알아보자.

### **Program.cs**

```cs
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

먼저 Program.cs를 살펴보자면 Main()함수가 있는 것으로 보아 Blazor Server 또한 콘솔 프로그램인 것을 알 수 있고 Blazor Server를 실행시키면 Main()함수가 먼저 호출이 될 것이다.

Main()함수가 호출되면 그 밑에 함수를 실행시키면서 Startup을 호출하는 것을 볼 수 있다.

### **Startup.cs**

```cs
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddRazorPages();
        services.AddServerSideBlazor();
        services.AddSingleton<WeatherForecastService>();
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseExceptionHandler("/Error");
            // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
            app.UseHsts();
        }

        app.UseHttpsRedirection();
        app.UseStaticFiles();

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapBlazorHub();
            endpoints.MapFallbackToPage("/_Host");
        });
    }
}
```

Main함수에서 Startup을 호출하면 ConfigureServices와 Configure이 실행되게 되는데 서비스들을 연결하고 라우팅 설정 등을 하게된다.

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages();
    services.AddServerSideBlazor();
    services.AddSingleton<WeatherForecastService>();
}
```

ConfigureServices에서는 어떠한 서비스들을 사용할 것인지를 추가해주는 역할을 한다.   
Blazor Server 프로젝트에서는 기본적으로 AddRazorPages(), AddServerSideBlazor()를 추가해주고 있는 것을 알 수 있다.   
그 밑에 AddSingleton\<WeatherForecastService\>()의 경우 현재 자동으로 생성된 예제에서 추가한 서비스이다. 이름이 WeatherForecastService인 것으로 보아 날씨 관련된 서비스인 것으로 보인다.

```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapBlazorHub();
        endpoints.MapFallbackToPage("/_Host");
    });
}
```

Configure에서는 라우팅 설정 및 여러 설정들을 해주는데 여기서 가장 중요한 부분은 Endpoints 부분이다.   
Endpoints는 쉽게 말해 어떤 주소를 입력하였을 때 어떤 애로 연동시켜줄 것인지 매핑해주는 부분이라고 생각하면 된다.
코드를 대략적으로 봐보면 endpoints를 BlazorHub와 매핑시켜주고 있고, FallbackToPage라고 _Host 페이지와 연결시켜주고 있는 것으로 보아 현재 웹 서버 주소로 접속하면 _Host 페이지와 연결시켜준다라는 것을 알 수 있다.

그러면 _Host 페이지가 실제로 우리에게 보여지는 페이지라는 의미인데 _Host 페이지 파일은 Pages 폴더 안에 _Host.cshtml 이름으로 찾아볼 수 있다.

### **_Host.cshtml**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>HelloBlazorServer</title>
    <base href="~/" />
    <link rel="stylesheet" href="css/bootstrap/bootstrap.min.css" />
    <link href="css/site.css" rel="stylesheet" />
</head>
<body>
    <app>
        <component type="typeof(App)" render-mode="ServerPrerendered" />
    </app>

    <div id="blazor-error-ui">
        <environment include="Staging,Production">
            An error has occurred. This application may no longer respond until reloaded.
        </environment>
        <environment include="Development">
            An unhandled exception has occurred. See browser dev tools for details.
        </environment>
        <a href="" class="reload">Reload</a>
        <a class="dismiss">🗙</a>
    </div>

    <script src="_framework/blazor.server.js"></script>
</body>
</html>
```

_Host.cshtml를 확인해보면 html 태그들로 구성되어 있는 것을 볼 수 있으며, 본문 내용을 담고 있는 \<body\>태그 안을 살펴보면 \<app\>태그 안에 \<component\>태그로 "typeof(App)"을 불러오고 있는 것을 볼 수 있다.

App은 Pages 폴더 바깥쪽에 App.razor 이름으로 찾아볼 수 있다.

### **App.razor**

```html
<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
```

App.razor에서는 Found, NotFound로 나눠져있는 것을 볼 수 있으며, Found일 때를 보면 "@typeof(MainLayout)"을 불러오고 있는 것을 알 수 있다.

MainLayout은 Shared 폴더안에 MainLayout.razor 이름으로 찾아볼 수 있다.

Blazor는 이런식으로 Component 단위로 관리하고 있는 것을 볼 수 있고 Component 파일 확장자를 보면 .razor인 것을 알 수 있다.   
.razor 파일은 Razor Component라고 하기도 하며 '새 항목 추가 > Razor 구성 요소'를 선택해 생성할 수 있다.   
Razor Component 파일은 Blazor에서 사용하는 파일 형식이다.

### **MainLayout.razor**

```html
<div class="sidebar">
    <NavMenu />
</div>

<div class="main">
    <div class="top-row px-4">
        <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
    </div>

    <div class="content px-4">
        @Body
    </div>
</div>
```

MainLayout.razor를 보면 sidebar와 main으로 구역이 나눠져 있다.   
이름만 보고 예측해보자면 사용자에게 보여지는 화면에서 좌측 부분이 sidebar, 나머지 부분이 main 구역으로 예측해볼 수 있다. (실제로 그러함)

![sidebar-main](/assets/img/posts/webserver/sidebar-main.png){: w="800"}
_sidebar 구역 main 구역_

자세히 확인해보고 싶으면 또 들어가보면 된다.   
sidebar 부분을 보면 \<NavMenu /\>라고 되어 있다. NavMenu부분은 Shared 폴더안에 NavMenu.razor 이름으로 있다.

### **NavMenu.razor**

```html
<div class="top-row pl-4 navbar navbar-dark">
    <a class="navbar-brand" href="">HelloBlazorServer</a>
    <button class="navbar-toggler" @onclick="ToggleNavMenu">
        <span class="navbar-toggler-icon"></span>
    </button>
</div>

<div class="@NavMenuCssClass" @onclick="ToggleNavMenu">
    <ul class="nav flex-column">
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="" Match="NavLinkMatch.All">
                <span class="oi oi-home" aria-hidden="true"></span> Home
            </NavLink>
        </li>
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="counter">
                <span class="oi oi-plus" aria-hidden="true"></span> Counter
            </NavLink>
        </li>
        <li class="nav-item px-3">
            <NavLink class="nav-link" href="fetchdata">
                <span class="oi oi-list-rich" aria-hidden="true"></span> Fetch data
            </NavLink>
        </li>
    </ul>
</div>

@code {
    private bool collapseNavMenu = true;

    private string NavMenuCssClass => collapseNavMenu ? "collapse" : null;

    private void ToggleNavMenu()
    {
        collapseNavMenu = !collapseNavMenu;
    }
}
```

여기에서도 \<div\>로 구역이 나누어져 있는 것을 볼 수 있는데   
첫 번째 구역은 HelloBlazorServer라는 문구가 나오는 걸로 보아 sidebar에서 위쪽 구역을 나타내는 것으로 볼 수 있고, 그 다음 구역은 Home, Counter, Fetch data라는 문구가 등장하는 것으로 보아 바로 그 밑 메뉴들을 나타내는 것을 알 수 있다.

메뉴들도 관찰해보면 link가 걸려 있는데 href 속성으로 연결되어있는 것을 볼 수 있다. 각 Counter, Fetch data 문구는 "counter", "fetchdata" 페이지와 연결되어 있고 각 "counter", "fetchdata" 페이지는 Pages 폴더에 Counter.razor, FetchData.razor 이름으로 있다.

### **Counter.razor**

```html
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

여기서 중요하게 볼만한 부분은 첫 줄에 @page "/counter" 부분이다.   
Blazor에서는 이런식으로 페이지 경로를 정의해주고 있다.   
아까 NavMenu.razor에서 href="counter" 라고 설정해 놓았는데 이 부분과 위에 @page "/counter" 부분과 맞아야 연결이 된다.

또한 Blazor의 특징으로 MVC 모델에서는 View와 Controller를 아에 따로 관리를 하였고, Razor Pages에서는 View와 Controller를 상하위 파일로 묶어 관리하였는데 Blazor에서는 .razor 파일 안에 @code 를 넣어 View와 Controller를 한 파일로 관리하고 있다.

![Counter](/assets/img/posts/webserver/Blazor_Counter.png){: w="800"}
_Counter 페이지_

### **FetchData.razor**

```html
@page "/fetchdata"

@using HelloBlazorServer.Data
@inject WeatherForecastService ForecastService

<h1>Weather forecast</h1>

<p>This component demonstrates fetching data from a service.</p>

@if (forecasts == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Temp. (C)</th>
                <th>Temp. (F)</th>
                <th>Summary</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var forecast in forecasts)
            {
                <tr>
                    <td>@forecast.Date.ToShortDateString()</td>
                    <td>@forecast.TemperatureC</td>
                    <td>@forecast.TemperatureF</td>
                    <td>@forecast.Summary</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
    }
}
```

FetchData.razor 또한 Counter.razor와 비슷하며,   
여기서 볼만한 부분은 if else문과 foreach문을 html 태그들과 섞어서 사용할 수도 있다는 부분이다.

![Fetch data](/assets/img/posts/webserver/Blazor_FetchData.png){: w="800"}
_Counter 페이지_

여기까지 코드들을 무작정 따라가보면서 대략적이라도 Blazor Server 동작과정과 구조를 살펴보았는데 웹 서버는 이처럼 구조가 복잡한 면을 가지고 있다보니 이렇게 무작정 코드들을 따라가보는 것도 중요하다.

## **bootstrap**

추가적으로 하나 더 언급하자면 웹페이지를 만들 때 전문적으로 꾸밀 것이 아니라면 일일이 CSS코드를 짜는 것이 번거로울 것이다. 그렇기에 라이브러리를 활용하여 CSS를 적용해볼 수 있다.

방금 만들어본 Blazor Server 프로젝트 또한 살펴보면 bootstrap이라는 라이브러리를 사용하고 있는 것을 확인해 볼 수 있다.

![bootstrap](/assets/img/posts/webserver/bootstrap.png){: w="400"}
_Blazor Server 프로젝트안 bootstrap 파일_

**Bootstrap**은 각종 레이아웃, 버튼, 입력창 등의 디자인과 기능을 **CSS와 JavaScript로 만들어 놓은 프레임워크**이다.

Bootstrap또한 공부해 놓으면 좋다.
