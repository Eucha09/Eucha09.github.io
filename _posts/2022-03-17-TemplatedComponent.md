---
title: "[Blazor] Templated Component"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-03-17 14:47:00 +0900
categories: [웹 서버, Blazor 입문]
tags: [Blazor, Templated Component, RenderFragment]     # TAG names should always be lowercase
---

## **Templated Component**

Blazor Templated Component는 하나 이상의 UI 템플릿을 매개 변수로 허용한 다음, 이 매개 변수를 구성 요소 렌더링 논리의 일부로 사용할 수 있는 구성 요소이다.

쉽게 말해 어떤 UI 구조를 템플릿화하여 재사용이 용이하도록 만드는 것을 말한다.

예를 들어 아래와 같은 페이지에서 테이블 형식으로 표현된 UI 구조를 다른 곳에서도 사용하고 싶다면   
매번 코드 자체를 복사하여 사용해도 되지만   
저 테이블 구조를 템플릿화하여 어디에서든지 저 구조를 사용할 수 있게 한다면 좀 더 효율적일 것이다.

![FetchData](/assets/img/posts/webserver/Blazor_FetchData.png){: w="800"}
_테이블 구조_

## **Templated Component 실습 (FetchData.razor)**

Blazor 프로젝트를 생성하면 기본적으로 생성되는 FetchData.razor 예제를 가지고 실습을 해보자.

FetchData.razor에서 테이블 부분을 다른 곳에서도 사용할 수 있도록 Templated Component로 만들어 볼 것이다.

먼저 '새 항목 추가 > Razor 구성 요소'를 선택해 TableTemplate.razor를 생성해보자.

다음 코드를 작성하기 전에 잠깐 알아야 하는 것이 있어 설명하자면   
Templated Component에서는 RenderFragment 꼭 사용하게 된다.   
RenderFragment는 쉽게 말해 HTML 태그 등이 포함되어 있는 UI 컨텐츠를 인자로 받을 수 있게 하는 매개변수 형태라고 생각하면 된다.

코드를 통해 간단하게 사용법을 보여드리자면   
**TableTemplate.razor**
```html
<h3>TableTemplate</h3>

<table class="table">
    <thead>
        <tr>
            @Header
        </tr>
    </thead>

</table>

@code {
    // Parameter로 UI 컨텐츠 받기
    [Parameter]
    public RenderFragment Header { get; set; }
}
```
**FetchData.razor**
```html
...

<!-- Header 정보를 인자로 넘기기 -->
<TableTemplate>
    <Header>
        <th>Date</th>
        <th>Temp. (C)</th>
        <th>Temp. (F)</th>
        <th>Summary</th>
    </Header>
</TableTemplate>

...
```

데이터를 받아 사용하는 쪽에서는 Parameter로 RenderFragment형 매개변수를 선언하면 되고, 데이터를 넘기는 쪽에서는 태그 형식으로 넘겨주면 된다.

그럼 나머지 부분들도 작성해보자.

**TableTemplate.razor**
```html
@using BlazorApp.Data;

<!-- type도 Parameter로 받아 사용하고 있다. -->
@typeparam TItem

<h3>TableTemplate</h3>

<table class="table">
    <thead>
        <tr>
            @Header  <!-- Parameter로 받은 RenderFragment 사용 -->
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Items)
        {
            <tr>
                @Row(item)  <!-- Parameter로 받은 RenderFragment 사용 -->
            </tr>
        }
    </tbody>
</table>

@code {
    [Parameter]
    public RenderFragment Header { get; set; }

    // 인자가 있는 형태의 RenderFragment
    [Parameter]
    public RenderFragment<TItem> Row { get; set; }

    // UI에 표시할 데이터
    [Parameter]
    public IReadOnlyList<TItem> Items { get; set; }
}
```
**FetchData.razor**
```html
@page "/fetchdata"

@using BlazorApp.Data
@inject WeatherForecastService ForecastService

<h1>Weather forecast</h1>

<p>This component demonstrates fetching data from a service.</p>

@if (forecasts == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <!-- TableTemplate 사용 -->
    <!-- UI로 표시할 데이터와 type을 인자로 넘겨주고 있음 -->
    <TableTemplate Items="forecasts" TItem="WeatherForecast">
        <!-- RenderFragment -->
        <Header>
            <th>Date</th>
            <th>Temp. (C)</th>
            <th>Temp. (F)</th>
            <th>Summary</th>
        </Header>
        <!-- 인자가 있는 형태의 RenderFragment -->
        <!-- Context="인자로 받을 매개변수 이름"(아무렇게 지어도 됨) -->
        <Row Context="forecast">
            <td>@forecast.Date.ToShortDateString()</td>
            <td>@forecast.TemperatureC</td>
            <td>@forecast.TemperatureF</td>
            <td>@forecast.Summary</td>
        </Row>
    </TableTemplate>
}

@code {
    private WeatherForecast[] forecasts;

    protected override async Task OnInitializedAsync()
    {
        forecasts = await ForecastService.GetForecastAsync(DateTime.Now);
    }
}
```

주석으로 표시한 부분을 유심히 보면 된다.

실행시켜보면 정상적으로 테이블이 생성돼 표현된 것을 볼 수 있다.

![TableTemplate](/assets/img/posts/webserver/TableTemplate.png){: w="800"}
_TableTemplate 사용 모습_

이런식으로 자주 사용하게 될 것같은 UI구조를 Templated Component로 만들면 동일한 UI구조를 다른 곳에서도 쉽게 사용할 수 있으며 코드 재사용률도 높일 수 있다.
