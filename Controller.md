// Decompiled with JetBrains decompiler
// Type: Microsoft.AspNetCore.Mvc.Controller
// Assembly: Microsoft.AspNetCore.Mvc.ViewFeatures, Version=2.1.1.0, Culture=neutral, PublicKeyToken=adb9793829ddae60
// MVID: 398FFC85-F043-4882-8DB1-98F1D9A058A5
// Assembly location: C:\Program Files\dotnet\sdk\NuGetFallbackFolder\microsoft.aspnetcore.mvc.viewfeatures\2.1.1\lib\netstandard2.0\Microsoft.AspNetCore.Mvc.ViewFeatures.dll

using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc.Filters;
using Microsoft.AspNetCore.Mvc.ModelBinding;
using Microsoft.AspNetCore.Mvc.ViewFeatures;
using Microsoft.AspNetCore.Mvc.ViewFeatures.Internal;
using Microsoft.Extensions.DependencyInjection;
using Newtonsoft.Json;
using System;
using System.Threading.Tasks;

namespace Microsoft.AspNetCore.Mvc
{
  /// <summary>A base class for an MVC controller with view support.</summary>
  public abstract class Controller : ControllerBase, IActionFilter, IFilterMetadata, IAsyncActionFilter, IDisposable
  {
    private ITempDataDictionary _tempData;
    private DynamicViewData _viewBag;
    private ViewDataDictionary _viewData;

    /// <summary>
    /// Gets or sets <see cref="T:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary" /> used by <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> and <see cref="P:Microsoft.AspNetCore.Mvc.Controller.ViewBag" />.
    /// </summary>
    /// <remarks>
    /// By default, this property is intiailized when <see cref="T:Microsoft.AspNetCore.Mvc.Controllers.IControllerActivator" /> activates
    /// controllers.
    /// <para>
    /// This property can be accessed after the controller has been activated, for example, in a controller action
    /// or by overriding <see cref="M:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting(Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext)" />.
    /// </para>
    /// <para>
    /// This property can be also accessed from within a unit test where it is initialized with
    /// <see cref="T:Microsoft.AspNetCore.Mvc.ModelBinding.EmptyModelMetadataProvider" />.
    /// </para>
    /// </remarks>
    [ViewDataDictionary]
    public ViewDataDictionary ViewData
    {
      get
      {
        if (this._viewData == null)
          this._viewData = new ViewDataDictionary((IModelMetadataProvider) new EmptyModelMetadataProvider(), this.ControllerContext.ModelState);
        return this._viewData;
      }
      set
      {
        if (value == null)
          throw new ArgumentException(Resources.ArgumentCannotBeNullOrEmpty, nameof (ViewData));
        this._viewData = value;
      }
    }

    /// <summary>
    /// Gets or sets <see cref="T:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary" /> used by <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" />.
    /// </summary>
    public ITempDataDictionary TempData
    {
      get
      {
        if (this._tempData == null)
        {
          HttpContext httpContext = this.HttpContext;
          ITempDataDictionaryFactory dictionaryFactory;
          if (httpContext == null)
          {
            dictionaryFactory = (ITempDataDictionaryFactory) null;
          }
          else
          {
            IServiceProvider requestServices = httpContext.RequestServices;
            dictionaryFactory = requestServices != null ? requestServices.GetRequiredService<ITempDataDictionaryFactory>() : (ITempDataDictionaryFactory) null;
          }
          this._tempData = dictionaryFactory?.GetTempData(this.HttpContext);
        }
        return this._tempData;
      }
      set
      {
        if (value == null)
          throw new ArgumentNullException(nameof (value));
        this._tempData = value;
      }
    }

    /// <summary>Gets the dynamic view bag.</summary>
    public object ViewBag
    {
      get
      {
        if (this._viewBag == null)
          this._viewBag = new DynamicViewData((Func<ViewDataDictionary>) (() => this.ViewData));
        return (object) this._viewBag;
      }
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object that renders a view to the response.
    /// </summary>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewResult View()
    {
      return this.View((string) null);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object by specifying a <paramref name="viewName" />.
    /// </summary>
    /// <param name="viewName">The name or path of the view that is rendered to the response.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewResult View(string viewName)
    {
      return this.View(viewName, this.ViewData.Model);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object by specifying a <paramref name="model" />
    /// to be rendered by the view.
    /// </summary>
    /// <param name="model">The model that is rendered by the view.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewResult View(object model)
    {
      return this.View((string) null, model);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object by specifying a <paramref name="viewName" />
    /// and the <paramref name="model" /> to be rendered by the view.
    /// </summary>
    /// <param name="viewName">The name or path of the view that is rendered to the response.</param>
    /// <param name="model">The model that is rendered by the view.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewResult View(string viewName, object model)
    {
      this.ViewData.Model = model;
      return new ViewResult()
      {
        ViewName = viewName,
        ViewData = this.ViewData,
        TempData = this.TempData
      };
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object that renders a partial view to the response.
    /// </summary>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual PartialViewResult PartialView()
    {
      return this.PartialView((string) null);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object by specifying a <paramref name="viewName" />.
    /// </summary>
    /// <param name="viewName">The name or path of the partial view that is rendered to the response.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual PartialViewResult PartialView(string viewName)
    {
      return this.PartialView(viewName, this.ViewData.Model);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object by specifying a <paramref name="model" />
    /// to be rendered by the partial view.
    /// </summary>
    /// <param name="model">The model that is rendered by the partial view.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual PartialViewResult PartialView(object model)
    {
      return this.PartialView((string) null, model);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object by specifying a <paramref name="viewName" />
    /// and the <paramref name="model" /> to be rendered by the partial view.
    /// </summary>
    /// <param name="viewName">The name or path of the partial view that is rendered to the response.</param>
    /// <param name="model">The model that is rendered by the partial view.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.PartialViewResult" /> object for the response.</returns>
    [NonAction]
    public virtual PartialViewResult PartialView(string viewName, object model)
    {
      this.ViewData.Model = model;
      return new PartialViewResult()
      {
        ViewName = viewName,
        ViewData = this.ViewData,
        TempData = this.TempData
      };
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> by specifying the name of a view component to render.
    /// </summary>
    /// <param name="componentName">
    /// The view component name. Can be a view component
    /// <see cref="P:Microsoft.AspNetCore.Mvc.ViewComponents.ViewComponentDescriptor.ShortName" /> or
    /// <see cref="P:Microsoft.AspNetCore.Mvc.ViewComponents.ViewComponentDescriptor.FullName" />.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewComponentResult ViewComponent(string componentName)
    {
      return this.ViewComponent(componentName, (object) null);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> by specifying the <see cref="T:System.Type" /> of a view component to
    /// render.
    /// </summary>
    /// <param name="componentType">The view component <see cref="T:System.Type" />.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewComponentResult ViewComponent(Type componentType)
    {
      return this.ViewComponent(componentType, (object) null);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> by specifying the name of a view component to render.
    /// </summary>
    /// <param name="componentName">
    /// The view component name. Can be a view component
    /// <see cref="P:Microsoft.AspNetCore.Mvc.ViewComponents.ViewComponentDescriptor.ShortName" /> or
    /// <see cref="P:Microsoft.AspNetCore.Mvc.ViewComponents.ViewComponentDescriptor.FullName" />.</param>
    /// <param name="arguments">
    /// An <see cref="T:System.Object" /> with properties representing arguments to be passed to the invoked view component
    /// method. Alternatively, an <see cref="T:System.Collections.Generic.IDictionary`2" /> instance
    /// containing the invocation arguments.
    /// </param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewComponentResult ViewComponent(string componentName, object arguments)
    {
      return new ViewComponentResult()
      {
        ViewComponentName = componentName,
        Arguments = arguments,
        ViewData = this.ViewData,
        TempData = this.TempData
      };
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> by specifying the <see cref="T:System.Type" /> of a view component to
    /// render.
    /// </summary>
    /// <param name="componentType">The view component <see cref="T:System.Type" />.</param>
    /// <param name="arguments">
    /// An <see cref="T:System.Object" /> with properties representing arguments to be passed to the invoked view component
    /// method. Alternatively, an <see cref="T:System.Collections.Generic.IDictionary`2" /> instance
    /// containing the invocation arguments.
    /// </param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.ViewComponentResult" /> object for the response.</returns>
    [NonAction]
    public virtual ViewComponentResult ViewComponent(Type componentType, object arguments)
    {
      return new ViewComponentResult()
      {
        ViewComponentType = componentType,
        Arguments = arguments,
        ViewData = this.ViewData,
        TempData = this.TempData
      };
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.JsonResult" /> object that serializes the specified <paramref name="data" /> object
    /// to JSON.
    /// </summary>
    /// <param name="data">The object to serialize.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.JsonResult" /> that serializes the specified <paramref name="data" />
    /// to JSON format for the response.</returns>
    [NonAction]
    public virtual JsonResult Json(object data)
    {
      return new JsonResult(data);
    }

    /// <summary>
    /// Creates a <see cref="T:Microsoft.AspNetCore.Mvc.JsonResult" /> object that serializes the specified <paramref name="data" /> object
    /// to JSON.
    /// </summary>
    /// <param name="data">The object to serialize.</param>
    /// <param name="serializerSettings">The <see cref="T:Newtonsoft.Json.JsonSerializerSettings" /> to be used by
    /// the formatter.</param>
    /// <returns>The created <see cref="T:Microsoft.AspNetCore.Mvc.JsonResult" /> that serializes the specified <paramref name="data" />
    /// as JSON format for the response.</returns>
    /// <remarks>Callers should cache an instance of <see cref="T:Newtonsoft.Json.JsonSerializerSettings" /> to avoid
    /// recreating cached data with each call.</remarks>
    [NonAction]
    public virtual JsonResult Json(object data, JsonSerializerSettings serializerSettings)
    {
      if (serializerSettings == null)
        throw new ArgumentNullException(nameof (serializerSettings));
      return new JsonResult(data, serializerSettings);
    }

    /// <summary>Called before the action method is invoked.</summary>
    /// <param name="context">The action executing context.</param>
    [NonAction]
    public virtual void OnActionExecuting(ActionExecutingContext context)
    {
    }

    /// <summary>Called after the action method is invoked.</summary>
    /// <param name="context">The action executed context.</param>
    [NonAction]
    public virtual void OnActionExecuted(ActionExecutedContext context)
    {
    }

    /// <summary>Called before the action method is invoked.</summary>
    /// <param name="context">The action executing context.</param>
    /// <param name="next">The <see cref="T:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate" /> to execute. Invoke this delegate in the body
    /// of <see cref="M:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync(Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext,Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate)" /> to continue execution of the action.</param>
    /// <returns>A <see cref="T:System.Threading.Tasks.Task" /> instance.</returns>
    [NonAction]
    public virtual async Task OnActionExecutionAsync(ActionExecutingContext context, ActionExecutionDelegate next)
    {
      if (context == null)
        throw new ArgumentNullException(nameof (context));
      if (next == null)
        throw new ArgumentNullException(nameof (next));
      this.OnActionExecuting(context);
      if (context.Result != null)
        return;
      this.OnActionExecuted(await next());
    }

    /// <inheritdoc />
    public void Dispose()
    {
      this.Dispose(true);
    }

    /// <summary>
    /// Releases all resources currently used by this <see cref="T:Microsoft.AspNetCore.Mvc.Controller" /> instance.
    /// </summary>
    /// <param name="disposing"><c>true</c> if this method is being invoked by the <see cref="M:Microsoft.AspNetCore.Mvc.Controller.Dispose" /> method,
    /// otherwise <c>false</c>.</param>
    protected virtual void Dispose(bool disposing)
    {
    }
  }
}
