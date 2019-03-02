<div id="Top"></div>
###ASP.NET CORE 基础
[基础介绍](https://www.cnblogs.com/qinxingnet/p/5837590.html)
> MVC
> > Model:业务逻辑和业务数据
> View:数据
> Controller:接收请求

> 请求流程：接收请求->Controller->传递请求数据->Model->返回处理请求->Controller->呈现数据->View->返回结果

#####Startup.cs解析：（启动配置文件）
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using CoreBB.Web.Models;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;

namespace CoreBB.Web
{
    public class Startup
    {
        // 此方法由运行时调用。使用此方法将服务添加到容器中。
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();//加载Mvc组件
            services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)//加载身份验证服务集合扩展组件:基于Cookie的身份验证处理程序
                .AddCookie(option =>
                {
                    option.LoginPath = "/User/LogIn";//默认登录路径/User/LogIn
                    option.AccessDeniedPath = "/Error/AccessDenied";//当访问被拒绝时的路径(未经授权的访问将被重定向到/Error/AccessDenied)无权访问资源的授权用户
                    option.Cookie.SecurePolicy = CookieSecurePolicy.Always;//安全始终标记为真
                });
            services.AddDbContext<CoreBBContext>();//将CoreBBContext类注册到ASP.NET Core的依赖注入容器
        }

        // 此方法由运行时调用。使用此方法配置HTTP请求管道。
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseExceptionHandler("/Error/Index");//使用异常处理程序扩展(默认的错误处理路径是/Error/Index)内部异常，显示相应的错误信息给用户
            app.UseStaticFiles();//允许使用静态文件（根文件夹是wwwroot）
            app.UseAuthentication();//启用身份验证功能组件
            app.UseMvcWithDefaultRoute();//启用带默认路径的Mvc方法组件（默认的URL路由模板是{controller=Home}/{action=Index}/{id?}）
        }
    }
}
```
> 控制器：
> > 是一个类
> 命名以Conttroller结尾
> 如果控制器命名不以Controller结尾，需要类上添加特性“[Controller]”，此时框架也会认为它是个控制器类
> [NonController]：控制器前面有这个特性时，它就不再是控制器了
> 需要继承Controller类

> 用户请求如何映射到控制器方法上的：
> > 默认路由规则：域名/{Controller}/{Action}/{id?}

> 如果控制器中的方法没有对应的视图：return Content("content");
> 有对应视图：return View();
```
using Microsoft.AspNetCore.Mvc;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            return Content("Han");
            //return View();
        }
    }
}
```
> 数据形式：
> > QueryString(查询字符串：通过地址栏，键值对形式)url问号后面的字符串
> Form(表单)
> Cookie
> Session
> Header

#####数据接收的两种方式：（两个对象）
- HttpRequest
- HttpContext
- 还有一种数据绑定的方式也可以（绑定到控制器的参数上）


> 接收数据：HttpRequest:他是用户请求对象
> HttpRequest是一个对象
> HttpRequest提供获取请求数据的属性：Cookie,Headers,Query,Forms
> HttpRequest是无法获取Session数据的
> 方法中直接写：var str=Request.Query("key");Console.WriteLine(str);
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            var str=Request.Query["name"];

            //Request.Form[""];//获取表单数据
            //Request.Cookies[""];//获取Cookie数据

            return Content($"Hello {str}");
        }
    }
}
```
> URL:https://localhost:44380/test/index?name=han
> 若不传入也不会出错

-----
> HttpContext:是用户请求上下文对象：
> 提供Session属性获取Session对象：
> > Session.Set设置（Set只允许设置byte[]类型）（HttpContext.Session.SetString)
> Session.Remove移除
> Session.TryGetValue获取数据（HttpContext.Session.getString）
> > <span style="background:yellow">使用Session的前提：在Startup.cs文件中添加并启用Session
> 在ConfigureServices()方法中添加：services.AddSession();//将Session服务添加到容器中
> 在Configure()方法中添加：app.UseSession();//运行时启用Session（不能放在app.UseMvc()下面）</span>
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            HttpContext.Session.SetString("name", "abcdef");

            HttpContext.Session.SetInt32("age", 32);//设置
            HttpContext.Session.Remove("age");//移除

            string str=HttpContext.Session.GetString("name");//获取

            int? age = HttpContext.Session.GetInt32("age");//此时获取不到了
            //var age = HttpContext.Session.GetInt32("age");

            return Content(str+"---"+age);
        }
    }
}
```

-----
> 数据绑定：
> 把用户请求的数据绑定到控制器方法的参数上
> 支持简单类型和自定义类型
> 绑定规则时请求数据名称和参数名称一致
> - 查询字符串Key名称和参数一致
> - Form表单名称和参数一致.....
> 
> <span style="color:red">数据的来源还是从URL的问号后面的键值对获取</span>
> 模型类也可以传递数据（它也是从url中获取数据的）
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index(string name)//从url中获取数据
        {
            return Content($"Hello {name}");
        }

        public IActionResult Say(TestModel model)//从url中获取数据
        {
            return Content($"nihao {model.Name}");
        }
    }

    public class TestModel//可以写入到TestModel.cs模型类中
    {
        public string Name { get; set; }
    }
}
```

----
> 1. 如果同一个数据在多个数据源里都出现了，但是只想取特定的数据源的数据怎么办？
> 上句：即在多个地方（例如url和session都有name的赋值）都出现了同名的参数值，但我只想获取其中某一个
> 2. 是不是所有的数据都可以自动绑定到方法参数上？
**框架有默认优先级进行获取 **

####解决上述问题：特性
- 控制器特性:当前定义的类作为一个控制器或者当前控制器作为一个普通类：[Controller]   [NonController]
- 方法参数特性：
> - FromHeader,（Headers）
> - FromRoute(路由数据),
> - FromForm(请求体),
> - FromBody(表单数据),
> - FromQuery(查询字符串),
> - FromServices(服务注册)

- 特性参数：
- 通过特性修饰参数来影响绑定逻辑
- 灵活可扩展
- **可自定义特性**

只从url中获取name值的来源
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            return View();
        }

        public IActionResult Say([FromQuery] string name)//从url中获取数据
        {
            return Content($"nihao {name}");
        }
    }
}
```
Test.index.cshtml
```
<form action="/test/say?name=bbb" method="post"><!--此处url中name值是特性指定获取的对象-->
    <input type="text" name="name"/><br>
    <input type="submit" value="提交" />
</form>
```
> - 从远程服务绑定数据
> - 从缓存绑定数据
> - Anywhere  Anydata

----
> 函数签名：
>>  函数：
> - 名称：用于映射
> - 参数：用于接收数据
> - 返回值：用于返回结果

#####结果表达形式：（返回值类型）
动作结果接口：IActionResult
- 具体实现：
- JsonResult：返回Json结构数据
- RedirectResult：跳转到新地址
- FileResult：返回文件
- ViewResult：返回视图页面
- ContentResult：文本内容
| 类型|     实例化对象|   封装方法(框架提供的简便方法)|
| :-------- | --------:| :------: |
| Json结果|   JsonResult|  Json(Object)|
|跳转|RedirectResult|Redirect(URL)|
|文件|FileResult|File()|
|视图|ViewResult|View()|
|文本|ContentResult|Content("")|
实例化对象实例:
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            JsonResult result = new JsonResult(new { name = "zhangsan" });//需要实例化对象后再使用
            return result;
        }
    }
}
```
结果:
```
{"name":"zhangsan"}
```
封装方法(简化实例化的复杂度):
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            return Json(new { name = "zhanfsan" });//直接使用了
        }
    }
}
```
> - 默认的视图查找路径:views/{Controller}/{Action}.cshtml
> - 指定视图查找路径:View("视图路径");
> > return View("~/Views/Home/a.cshtml");

> 如果方法逻辑中涉及到I/O操作的话,最好返回值类型(结果形式):Task< IActionResult>
> > public Task< IActionResult> Test(){........}

-----
> 视图引擎:Razor:将c#和html的整合
> > 1. 数据传递
> > 2.  数据绑定
#####1. 数据传递
|ViewData|ViewBag|TempData|Model|Session|Cache|
| :-------- | --------:| :------: |

| ViewData|     ViewBag|
| :-------- | --------:|
| 键值对|   动态类型| 
|索引器|它底层是对ViewData的封装|
|支持任意类型|动态属性|

> 数据绑定:
>> Razor语法:
> @表示c#代码块或语句
> @* 注释 *@
> c#代码可与html混合

设置值:
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            ViewData["name"] = "zhangsan";

            ViewBag.name2 = "lisi";

            return View();
        }
    }
}
```
显示值:
```
<p>@ViewData["name"]</p>
<p>@ViewBag.name2</p>
```
设置列表数据:
```
namespace myConsole8.Controllers
{
    [Controller]
    public class Test:Controller
    {
        public IActionResult Index()
        {
            //传递列表数据（数据库中获取）
            List<string> str = new List<string>();
            str.Add("one");
            str.Add("one2");
            str.Add("one3");
            str.Add("one4");
            str.Add("one5");
            ViewBag.str = str;//将列表数据绑定到ViewBag中

            return View();
        }
    }
}
```
循环输出:
```
<ul>
@foreach (var item in ViewBag.str)//循环输出数据
{
   <li>@item</li>
}
</ul>
```
> 表单与数据绑定
> >表单的使用:
> >> 1. 传统Html方式
> >>2. HtmlHelper
```
<form action="/test/checkuser" method="post"><!--1-->
   Name: <input type="text" name="name"/><br>
   Password:<input type="password" name="password"/><br>

    <br>
    Name:@Html.TextBox("name")<br><!--2-->
    Password:@Html.Password("password")<br>

   <input type="submit" value="提交" />
</form>
```
Test.CheckUser方法获取数据:
```
public IActionResult CheckUser(string name,string password,string name2,string password2)
        {
            return Content($"{name}+{password}+{name2}+{password2}");
        }
```
设置默认值:
- Test.Index方法在呈现视图之前设置默认数据
```
public IActionResult Index()
        {
            ViewData["name"] = "zhangsan";//设置数据
            return View();
        }
```
- 显示数据:
- value="@ViewBag.name"
- 或者value="@ViewData["name"]"
```
Name: <input type="text" name="name" value="@ViewBag.name"/><br>
```
- Select数据初始化:
```
public IActionResult Index()
        {
            List<int> list = new List<int>();
            list.Add(1);
            list.Add(11);
            list.Add(111);
            list.Add(1111);
            list.Add(11111);
            ViewBag.list = list;

            return View();
        }
```
```
<select>
    @foreach (var i in ViewBag.list)
    {
        <option value="">@i</option>
    }
</select>
```

-----
#####HtmlHelper方式:
- 提供方法的重载:@Html.TextBox(name,value,format,htmlattributes)
- 四个值的含义:表单名称,表单的值,格式化的方式,扩充html的属性
```
Name:@Html.TextBox("name2",(string)ViewBag.name,null,new { style="color:red"})<br><!--2-->
```
> 特定类型:dynamic
#####使用Model的方式传递数据
> 视图中的Model:
> - 视图提供Model属性
> - Model是一个动态类型,类型取决于传递数据类型
> 
> 通过View(Object)传递模型数据
> 强类型视图:（确定类型）
> - 强类型Model是一个设计时就能确定具体数据类型,而不是运行时确定
> - 强类型Model能提供设计时的智能提示
> - 强类型Model能方便数据绑定及数据验证
> 
> 定义: 
> - 使用@model关键字
> - 必须在视图第一行定义
> - 要在@model后面跟着数据类型名称
> - Model属性的类型就是@model标识的类型
> 

```
@model myConsole8.Models.TestModel;
```
实例:
Models.TestModel.cs
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace myConsole8.Models
{
    public class TestModel//必须是public
    {
        public string Name { get; set; }
        public int Telphone { get; set; }
    }
}
```
Controllers.Test.cs
```
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using myConsole8.Models;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace myConsole8.Controllers
{
    [Controller]
    public class Test : Controller
    {
        public IActionResult Index()
        {
            TestModel model = new TestModel {//实例化模型对象并传入相应数据
                Name = "zhangsan",
                Telphone = 1234567890
            };

            return View(model);//将model以参数形式传递进去
        }
    }
}
```
Views.Test.Index.cshtml
```
@model myConsole8.Models.TestModel;
@{
    ViewData["Title"] = "Index";
}

<h2>Index</h2>

@Html.TextBoxFor(m => m.Name)<!--表单生成的HtmlHelper方式-->

<table border="1">
    <tr>
        <td>@Model.Name</td>
        <td>@Model.Telphone</td>
    </tr>
</table>
```

-----
> 表单标题名称统一管理实例：（利于后期维护管理）
> DisplayNameAttribute和Html.LabelFor
#####在TestModel.cs模型文件的相应字段上方添加:[DisplayName("名称")]
```
namespace myConsole8.Models
{
    public class TestModel//必须是public
    {
        //改此一处，所有的名称都更改了
        [DisplayName("用户名")]//引入名称空间using System.ComponentModel;
        public string Name { get; set; }

        [DisplayName("电话")]
        public int Telphone { get; set; }
    }
}
```
#####视图文件Index.cshtml:@Html.LabelFor(m=>m.字段名)
```
<table border="1">
    <tr>
        <td>@Html.LabelFor(m=>m.Name)</td>
        <td>@Html.TextBoxFor(m=>m.Name)</td>
    </tr>
    <tr>
        <td>@Html.LabelFor(m=>m.Telphone)</td>
        <td>@Html.TextBoxFor(m=>m.Telphone)</td>
    </tr>
</table>
```

------
#####某些内容在多个页面需要共享的实现
- 母版页
- 部分视图
######母版页
- 实现多个视图内容的共享，减少编码量
- 便于系统维护，共享部分的内容修改，改一处，全部都自动更改了。
> 使用方法：
> - 和普通视图创建方法一样
> - 在@renderbody()输出子页面内容
> - 在子页面中使用Layout属性指定母版页
> - 用@rendersection()现了子页面内容的个性化，这样可以达到共享内容最大化

- 母版页代码:mby.cshtml(放在Views文件夹内就可以了)
```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
</head>
<body>
    <p>头部内容</p>

    <div>@RenderBody()</div>

    <p>尾部内容</p>
</body>
</html>
```
- 子页内容：
```
@{
    ViewData["Title"] = "Privacy";
    Layout = "~/Views/shared/mby.cshtml";
}

<p>这是子页面内容</p>
```
> 内容隔离：@RenderSection()
> 在母版页相应位置：@RenderSection("name1")
> 在子页面相应位置处：@Section name1{ < p>隔离内容部分< /p> }
######母版页
```
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title></title>
</head>
<body>
    <p>头部内容</p>

    @RenderSection("section1")

    <div>@RenderBody()</div>

    @RenderSection("section2")

    <p>尾部内容</p>
</body>
</html>
```
######子页面
```
@{
    ViewData["Title"] = "Privacy";
    Layout = "~/Views/shared/mby.cshtml";
}

@section section1{ 
    <p>这是区域1内容</p>
}

<p>这是子页面内容</p>

@section section2{ 
    <p>这是区域2内容</p>
}
```
#####母版页数据绑定
> 在页面顶部用一个变量获取数据，然后在页面中调用该变量即可
- 设置数据：控制器渲染页面前：ViewData["name"]=12345;（一般是数据库获取）
- 页面获取：开头处：@{ var a= ViewBag.name; }
- 调用处：@a  即可

-----
#####部分视图
- 可以实现视图内容共享
- 可以在视图上通过<span style="background:yellow">Html.Partial("部分视图的路径“)</span>引入部分视图
- 可以直接作为视图结果（PartialView("路径")）返回(即控制器方法中作为返回值使用)<=局部刷新时会用到它（需要结合Ajax）
```
<li>@Html.Partial("~/Views/Shared/abc.cshtml")</li>
```
```
return PartialView("~/views/shared/abc.cshtml");
```
> 母版页和部分视图的区别：
> - 母版页不能独立使用，部分试图可以（PartialView("局部视图路径")）
> - 母版页用于内容共享，部分视图则倾向于页面的局部呈现，如局部刷新

-----
#####多个页面共享数据：TempData,Cache,Session

| TempData（临时数据）|     Cache（缓存） |   Session   |
| :-------- | --------:| :------: |
| 视图级别    |   应用程序级别|  会话级别|
|只允许消费一次|服务器端保存|服务器端保存|
|可多次赋值|可设置有效期||
|键值对形式|键值对形式|键值对形式|
#####TempData视图间共享数据
- 和ViewData使用方法一样（只限于使用方法）
- 但和上面有很大区别：
 > - TempData只能在一个视图使用一次（消费一次，刷新页面后数据就没了）
 > - ViewData只能在定义的控制器对应的视图中使用，而TempData可以跨视图使用
 > - [详细区别引用](https://www.cnblogs.com/EasonWu/p/beginner-aspnet-mvc-various-ways-of-passing-data-in-mvcapps.html)
 
- 刷新后还想要显示值，那就在显示代码下面再进行一次赋值操作了
控制器方法：AbcController.cs
```
public class AbcController : Controller
    {
        public IActionResult Index()
        {
            ViewData["name1"] = "zhangxan";
            TempData["name2"] = "lisi";
            return View();
        }
        public IActionResult Index2()
        {
            return View();
        }
    }
```
视图Index.cshtml
```
<body>
    <p>@ViewBag.name1</p>
    <a href="/abc/index2">index2</a>
</body>
```
视图Index2.cshtml
```
<body>
    <p>@ViewData["name1"]</p>
    <p>@TempData["name2"]</p><!--第一次显示lisi以后都显示zhaoliu-->
    @{ 
        TempData["name2"] = "zhaoliu";//从第一次刷新开始，他的值都是这个了（每次刷新都是一次新的赋值过程）
    }
</body>
```
#####Session:
> - 通过HttpContext.Session获取到Session对象（先实例化它，再使用它）
> - Session.Set方法添加数据
> - 索引器方式获取对应key的数据
```
public IActionResult Index()
        {
            //var session = HttpContext.Session;
            ISession session = HttpContext.Session;//建立对象

            session.SetString("name", "woshiren");//用对象找到对应方法来添加数据

            var name=session.GetString("name");//获取数据，并赋值到一个变量中

            return Content(name);//展示数据

           // return View();//也可以到视图中展示
        }
```
如果要在视图中展示数据，就将获取数据赋值给ViewBag.key即可
```
public class AbcController : Controller
    {
        public IActionResult Index()
        {
            //var session = HttpContext.Session;
            ISession session = HttpContext.Session;//建立对象

            session.SetString("name", "woshiren");//用对象找到对应方法来添加数据

            //var name=session.GetString("name");//获取数据，并赋值到一个变量中

            ViewBag.name = session.GetString("name");//如果要在视图中展示数据

            // return Content(name);//展示数据

            return View();
        }
        public IActionResult Index2()
        {
            return View();
        }
    }
```






----
<a href="#Top">Top</a>


















