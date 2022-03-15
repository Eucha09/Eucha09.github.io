---
title: "[Blazor] Cascading Parameter"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-03-15 14:28:00 +0900
categories: [웹 서버, Blazor 입문]
tags: [Blazor, Parameter, Cascading Parameter]     # TAG names should always be lowercase
---

## **Cascading Parameter**

Cascading Parameter는 상위 Component에서 원하는 수의 하위 Component들에게 데이터를 전달할 수 있는 기능이다.

쉽게 말해 일반적인 Parameter의 경우 데이터를 전달해줄때 Component마다 개별적으로 전달해주어야 하지만 Cascading Parameter의 경우 폭포(cascade)처럼 한번에 하위 Component들에게 데이터를 전달해 줄 수 있다.

예를 들어 웹 페이지에서 테마 색상을 바꿀 수 있는 옵션을 추가한다고 하였을 때 상위 Component에서 색상을 바꾸게 되면 하위 Component들도 똑같이 색상 정보를 받아 테마 색을 바꿔야 할 것이다.   
이때 일반적인 Parameter로 색상 정보를 전달할 경우 아래와 같이 하위 Component 태그마다 일일이 전달해 주어야 할 것이다.   
```html
<component1 Color="_selectedColor"></component1>
<component2 Color="_selectedColor"></component2>
<component3 Color="_selectedColor"></component3>
...
```
하지만 Cascading Parameter를 사용할 경우 아래와 같이 하위 Component들에게 한번에 전달해 줄 수 있다.
```html
<CascadingValue Name="ThemeColor" Value="_selectedColor">
  <component1></component1>
  <component2></component2>
  <component3></component3>
  ...
</CascadingValue>
```

## **Cascading Parameter 사용법**

### **\<CascadingValue\>**

```html
<CascadingValue Name="ThemeColor" Value="_selectedColor">
    <!-- Component -->
    <!-- Component -->
    <!-- Component -->
</CascadingValue>

@code {
    string _selectedColor = "Green";

}
```

값을 전달할 때는 \<CascadingValue\> 태그를 사용해 CascadingValue 이름과 전달할 Value를 지정한다.

### **[CascadingParameter]**

```html
@code {
    [CascadingParameter(Name = "ThemeColor")]
    string _color { get; set; }

}
```

값을 받을 때는 Parameter로 받을 변수 위에 [CascadingParameter]를 붙이고 받을 CascadingValue 이름을 지정한다.   
Parameter로 받을 변수는 프로퍼티로 선언해야 한다.

## **예제코드(User 관리 페이지)**

이전 포스트에서 구현해놓은 User 관리 페이지에다 테마 색상 옵션을 추가해 간단하게 User 이름 색상만 바꾸어 보자.

[이전 포스트](https://eucha09.github.io/posts/ParameterRefEventCallback/)

**User.razor**
```html
@page "/user"

@using BlazorApp.Data;

<h3>Online Users</h3>

<!-- 테마 색상 선택 UI-->
<label>Theme Color:</label>
<select class="form-control" @bind="_selectedColor">
    @foreach(var option in _options)
    {
        <option value="@option">
            @option
        </option>
    }
</select>

<!-- CascadingValue를 사용해 색상 정보 전달하기 -->
<CascadingValue Name="ThemeColor" Value="_selectedColor">
    <ShowUser Users="_users" CallbackTest="CallbackTestFunc" @ref="_showUser"></ShowUser>
</CascadingValue>

<br />

<div class="container">
    <div class="row">
        <div class="col">
            <input class="form-control" placeholder="Add User" @bind-value="_inputName" />
        </div>
        <div class="col">
            <button class="btn btn-primary" type="button" @onclick="AddUser">Add a User</button>
        </div>
    </div>
</div>

@code {
    // 테마 색상
    string _selectedColor = "Green";
    List<string> _options = new List<string>() { "Green", "Red", "Blue" };

    List<UserData> _users = new List<UserData>();
    ShowUser _showUser;

    string _inputName;

    void AddUser()
    {
        _showUser.AddUser(new UserData() { Name = _inputName });
        _inputName = "";
    }

    void CallbackTestFunc()
    {
        _inputName = "CallbackTest";

        //StateHasChanged();
    }
}
```
**ShowUser.razor**
```html
@using BlazorApp.Data;

<p>
    Users: <b>@Users.Count()</b>
</p>

<br />

<ul class="list-group">
    @foreach (UserData user in Users)
    {
        <li @key="user" class="list-group-item">
            <button type="button" class="btn btn-link" @onclick="(() => KickUser(user))">[Kick]</button>
            <!-- 라벨에다 색상 적용 -->
            <label style="color:@_color">@user.Name</label>
        </li>
    }
</ul>

@code {
    // Cascading Parameter로 색상 정보 받기
    [CascadingParameter(Name = "ThemeColor")]
    string _color { get; set; }

    [Parameter]
    public List<UserData> Users { get; set; }

    [Parameter]
    public EventCallback CallbackTest { get; set; }

    protected override void OnInitialized()
    {
        Users.Add(new UserData() { Name = "define_chan" });
        Users.Add(new UserData() { Name = "Rookiss" });
        Users.Add(new UserData() { Name = "Dongglee" });
    }

    public void AddUser(UserData user)
    {
        Users.Add(user);
    }

    public void KickUser(UserData user)
    {
        Users.Remove(user);

        CallbackTest.InvokeAsync(null);
    }
}
```

주석부분을 유심히 살펴보면 이해가 될것이다.

User.razor에서는 Cascading Parameter를 통해 테마 색상 정보를 ShowUser.razor에게 전달하고 있고 ShowUser.razor에서는 색상 정보를 받아 Label 색상에 적용시키고 있다.

실행시켜보면 색상을 선택할 때마다 User 이름들의 색이 바뀌는 것을 볼 수 있다.

![CascadingParameter](/assets/img/posts/webserver/CascadingParameter.png){: w="800"}
_Theme Color_
