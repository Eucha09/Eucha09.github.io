---
title: "[Blazor] Parameter, Ref, EventCallback"
author:
  name: define_chan
  link: https://eucha09.github.io/about/
date: 2022-03-14 15:01:00 +0900
categories: [웹 서버, Blazor 입문]
tags: [Blazor, Parameter, Ref, EventCallback]     # TAG names should always be lowercase
---

## **Component 분리**

Blazor에서는 Component 단위로 관리한다. Blazor에서의 Component를 Razor Component라고 부르며 확장자는 .razor이다.   
Component 단위로 분리하여 관리하면 코드 재사용률을 높일 수 있으며, 부품 조립하듯이 Component들을 조립하는 식으로 웹 페이지를 구성할 수 있다.

앞에서 만들어본 User 관리 페이지를 보면   
[User 관리 페이지](https://eucha09.github.io/posts/Binding%EC%8B%A4%EC%8A%B5/)   
User 목록을 보여주는 부분은 다른 곳에서도 사용할 수 있다.   
다른 곳에서도 사용하는 방법은 코드를 그대로 복사 붙여넣기 방법도 있겠지만 User 목록을 보여주는 부분을 부품화하여 사용하고 싶은 곳 어디든 가져와 붙일 수 있다면 편할 것이다.

그렇다면 User 목록을 보여주는 부분을 따로 꺼내 Component를 분리해보자.

Pages 폴더에 ‘새 항목 추가 > Razor 구성 요소’를 선택해 ShowUser.razor를 생성한다.   
그리고 User 목록을 보여주는 부분만 잘라내 ShowUser.razor에 붙여넣는다.

**ShowUser.razor**
```html
<p>
    Users: <b>@_users.Count()</b>
</p>

<br />

<ul class="list-group">
    @foreach (UserData user in _users)
    {
        <li @key="user" class="list-group-item">
            <button type="button" class="btn btn-link" @onclick="(() => KickUser(user))">[Kick]</button>
            <label>@user.Name</label>
        </li>
    }
</ul>

@code {

}
```

그 다음 User.razor에서 ShowUser.razor를 사용하는 방법은 간단하다.   
\<ShowUser\>\</ShowUser\> 이런식으로 태그를 넣으면 된다.

**User.razor**
```html
@page "/user"

@using BlazorApp.Data;

<h3>Online Users</h3>

<!-- ShowUser 붙여넣기 -->
<ShowUser></ShowUser>

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
    List<UserData> _users = new List<UserData>();

    string _inputName;

    protected override void OnInitialized()
    {
        _users.Add(new UserData() { Name = "define_chan" });
        _users.Add(new UserData() { Name = "Rookiss" });
        _users.Add(new UserData() { Name = "Dongglee" });
    }

    void AddUser()
    {
        _users.Add(new UserData() { Name = _inputName });
        _inputName = "";
    }

    void KickUser(UserData user)
    {
        _users.Remove(user);
    }
}
```

다만 이렇게 했을 때 문제가 발생한다.   
첫 번째로 _users 라는 데이터를 둘다 사용하려고 하고 있다는 점이다.   
_users 라는 데이터를 공유할 수 있다면 좋을텐데 이때 사용할 수 있는 방법이 Parameter이다.

## **Parameter**

Razor Component를 불러올때 함수를 호출하는 것처럼 Parameter를 통해 데이터를 전달해 줄 수 있다.

ShowUser.razor에서는 Parameter를 통해 전달받을 변수를 선언해주고   
User.razor에서는 ShowUser.razor를 부를 때 Parameter로 데이터를 넘겨준다.

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
            <!--<button type="button" class="btn btn-link" @onclick="(() => KickUser(user))">[Kick]</button>-->
            <label>@user.Name</label>
        </li>
    }
</ul>

@code {
    // Parameter로 받을 변수 선언
    [Parameter]
    public List<UserData> Users { get; set; }
}
```

참고로 [Parameter]로 지정해주려면 프로퍼티 형식으로 선언해주어야 한다.   
또한 UserData를 사용하기 위해 맨 위에 @using BlazorApp.Data;를 선언해주고 있다.

**User.razor**
```html
@page "/user"

@using BlazorApp.Data;

<h3>Online Users</h3>

<!-- ShowUser 붙여넣기 Parameter로 _users 전달 -->
<ShowUser Users="_users"></ShowUser>

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
    List<UserData> _users = new List<UserData>();

    string _inputName;

    protected override void OnInitialized()
    {
        _users.Add(new UserData() { Name = "define_chan" });
        _users.Add(new UserData() { Name = "Rookiss" });
        _users.Add(new UserData() { Name = "Dongglee" });
    }

    void AddUser()
    {
        _users.Add(new UserData() { Name = _inputName });
        _inputName = "";
    }

    void KickUser(UserData user)
    {
        _users.Remove(user);
    }
}
```

Parameter로 넘겨줄 때는   
```html
<!-- ShowUser 붙여넣기 Parameter로 _users 전달 -->
<ShowUser Users="_users"></ShowUser>
```
이런 형식으로 전달해주면 된다.

위와 같이 코드를 수정하였다면 실행이 잘 되는 것을 볼 수 있다.

![Parameter](/assets/img/posts/webserver/Parameter.png){: w="800"}
_Parameter_

다음으로 현재 테스트를 위해 ShowUser.razor에서 [Kick] 버튼 부분을 주석처리하였는데 [Kick] 버튼을 사용하려면 ShowUser.razor에다 KickUser() 함수를 정의해야한다.   
만약 User.razor에서 KickUser 기능을 사용하고 싶다면 ShowUser.razor 내에 KickUser 함수를 호출할 수 있어야 할 것이다.   
이때 @ref를 사용하면 다른 Razor Component에 있는 함수들도 사용할 수 있다.

## **@ref**

먼저 User 목록에 대해서는 ShowUser.razor에서 관리하는 것이 좋아보이니 ShowUser.razor에 AddUser, KickUser 함수 등을 정의해놓자.   

그 다음 User.razor에서 ShowUser.razor에 정의되어 있는 AddUser를 사용해보자.

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
            <label>@user.Name</label>
        </li>
    }
</ul>

@code {
    [Parameter]
    public List<UserData> Users { get; set; }

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
    }
}
```

**User.razor**
```html
@page "/user"

@using BlazorApp.Data;

<h3>Online Users</h3>

<!-- ShowUser 붙여넣기 _showUser와도 연동-->
<ShowUser Users="_users" @ref="_showUser"></ShowUser>

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
    List<UserData> _users = new List<UserData>();
    ShowUser _showUser;

    string _inputName;

    void AddUser()
    {
        // ShowUser 내 AddUser 함수 호출
        _showUser.AddUser(new UserData() { Name = _inputName });
        _inputName = "";
    }
}
```

실행시켜보면 Add a User와 Kick 기능이 잘 동작하는 것을 볼 수 있다.

마지막으로 지금은 부모 Component 쪽에서 자식 Component 쪽 함수를 사용하고 있는데 반대로도 가능하다.   
새로운 기능은 아니고 아까 Parameter 기능과 C#에서의 델리게이트를 활용한 방법이다.

## **EventCallback**

부모 Component 쪽에서 Parameter로 함수를 전달하면 자식 Component 쪽에서 Callback 형식으로 호출이 가능하다.   

간단하게 테스트를 해보자.

ShowUser.razor에서 델리게이트 형식으로 Parameter를 선언해놓고 KickUser()가 실행되었을 때 호출해주자.

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
            <label>@user.Name</label>
        </li>
    }
</ul>

@code {
    [Parameter]
    public List<UserData> Users { get; set; }

    // 함수를 받을 Parameter 추가
    [Parameter]
    public Action CallbackTest { get; set; }

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

        CallbackTest.Invoke(); // Parameter로 받은 함수 호출
    }
}
```
**User.razor**
```html
@page "/user"

@using BlazorApp.Data;

<h3>Online Users</h3>

<!-- CallbackTestFunc도 Parameter로 전달 -->
<ShowUser Users="_users" CallbackTest="CallbackTestFunc" @ref="_showUser"></ShowUser>

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
        _inputName = "CallbackTest"; // _inputName 변경

        StateHasChanged(); // 수동 UI 갱신
    }
}
```

여기서 중요한 점은 CallbackTestFunc()가 호출되어 _inputName이 변경되어도 UI가 자동으로 갱신이 안돼 수동으로 UI 갱신을 해주고 있다는 것이다.   

확인해보고 싶으면 StateHasChanged() 부분을 주석처리하고 실행해보자.

이처럼 StateHasChanged()를 호출해 수동으로 UI를 갱신해주어도 되지만 다른 방법도 있다.

ASP.NET Core에서 제공하는 EventCallback을 사용하는 것이다.

EventCallback 사용법은 Action과 매우 비슷하지만 다른 점은 StateHasChanged()를 수동으로 호출할 필요가 없다.

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
            <label>@user.Name</label>
        </li>
    }
</ul>

@code {
    [Parameter]
    public List<UserData> Users { get; set; }

    // Action 대신 EventCallback 사용
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

        CallbackTest.InvokeAsync(null); // Parameter로 받은 함수 호출
    }
}
```
**User.razor**
```html
@page "/user"

@using BlazorApp.Data;

<h3>Online Users</h3>

<!-- CallbackTestFunc도 Parameter로 전달 -->
<ShowUser Users="_users" CallbackTest="CallbackTestFunc" @ref="_showUser"></ShowUser>

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
        _inputName = "CallbackTest"; // _inputName 변경

        //StateHasChanged(); // EventCallback 사용시 호출 안해줘도 됨.
    }
}
```

실행시켜보면 StateHasChanged()를 호출하지 않았음에도 UI가 잘 갱신되는 것을 볼 수 있다.
