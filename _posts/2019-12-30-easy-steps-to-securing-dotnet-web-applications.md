---
layout: single
title:  "Easy Steps to Security DotNet Web Applications"
date:   2019-12-30 12:00:00 -0500
categories: appsec software dotnet 
author: XanderK
---

# Easy Steps to Securing Your Web Application

A developer's guide for ASP.NET and ASP.NET Core

Frequently, performing some very simple steps can greatly increase an application's security posture. From stopping your web server from leaking information to an attacker, to adding additional security headers to your application, this guide will explain the basic steps that will stop most attacks before they start.

## Cleaning up web server headers

The first step to hardening web applications built on the .NET platform is stopping the web server from providing the four standard headers ASP.NET populates in every response. These headers, with various values, are found in every default ASP.NET web project:

- **Server** : Microsoft-IIS/10.0
- **X-AspNet-Version** : 4.0.30319
- **X-AspNetMvc-Version** : 4.0
- **X-Powered-By** : ASP.NET

You might read that and ask, "why do these headers matter?" These headers give a potential attacker information about the platform serving the web application. This includes what version of IIS and Windows, what .NET framework version, and what version of MVC. This is valuable information and allows an attacker to employ tailored strategies to target a smaller set of potential vulnerabilities based on the information provided.

Removing these headers is very easy and forces the attacker to target a wider range of potential vulnerabilities, thus slowing them down.

### Server

The _Server_ header identifies what web server is serving the content for this request and is set by the webserver or reverse proxy directly. This should be removed from all responses. For IIS, this can be done either via code or the web.config.

In ASP.NET Framework, this can be done via the HttpApplication class implemented in your project. By default, this if found in the _Global.asax.cs_ file. Simply remove the "Server" header from the response in the _Application\_BeginRequest()_ method.

    protected void Application_BeginRequest() {  
        Response.Headers.Remove("Server");  
    }  

Alternatively, in IIS 10.0 and above, you can control this via the web.config by adding the following setting.

    <system.webServer>  
      <security>  
        <requestFiltering removeServerHeader="true"/>  
      </security>  
    </system.webServer>  

In ASP.NET Core, Kestrel must be instructed to not add the _Server_ header. This is done in the _CreateDefaultBuilder_ call usually found in _program.cs__._

    WebHost.CreateDefaultBuilder(args)  
           .UseKestrel(c => c.AddServerHeader = false)  
           .UseStartup<Startup>()  
           .Build();  

This only removes the HTTP header from Kestrel. If you are hosting Kestrel from within IIS, the header will likely still be added by IIS. If this is the case, you will still need make the above web.config change. If Kestrel is running on anything else, like Nginx, the web.config will not be necessary.

### X-AspNet-Version

The _X-AspNet-Version_ header identifies the version of the .NET Framework running the ASP.NET application and is added by the framework itself, but only in ASP.NET Framework. It is simple to remove by adding the following to your web.config.

**Note:**  **ASP.NET Core does not add this header.**

    <system.web>  
      <httpRuntime targetFramework="4.7.2" **enableVersionHeader="false"** />  
    </system.web>  

### X-AspNetMvc-Version

Like the _X-AspNet-Version_ header, the _X-AspNetMvc-Version_ header identifies the version of MVC running on the web application. This header requires us to edit the HttpApplication class to remove. Like above, by default, this if found in the _Global.asax.cs_ file. In the _Application\_Start()_ method we must instruct MVC to disable the header.

**Note:**  **ASP.NET Core does not add this header.**

    protected void Application_Start() {  
        // initialization code  
        MvcHandler.DisableMvcResponseHeader = true;  
    }  

### X-Powered-By

_X-Powered-By_ introduces us to an element in the web.config file with which many people aren't familiar. The _httpProtocol_ element contains a child element for _customHeaders_. Here, we can add our own headers (which we will be doing later) and remove existing headers in the response. If the application is being hosted by IIS, either directly or via Kestrel, remove the _X-Powered-By_ header by adding this to the web.config:

    <system.webServer>  
      <httpProtocol>  
        <customHeaders>  
          <remove name="X-Powered-By"/>  
        </customHeaders>  
      </httpProtocol>  
    </system.webServer>  

If the _X-Powered-By_ header is being added in an ASP.NET Core application, it is either because it is being hosted by IIS or due to a framework being used within the application. If the application is hosted by IIS, follow the instructions above for modifications to the web.config. Otherwise, it is likely do to another framework dependency and will need to be removed via middleware. Kestrel and ASP.NET Core do not use this header directly.

## Cleaning up your development settings

Another source of valuable information to an attacker is in the form of settings used during development that aren't properly turned off or removed when deployed. In an application that is hosted by IIS, the web.config needs to be adjusted before deployment to a public target. This is frequently accomplished through a _transform_. These are some important transform elements to include in your Web.Release.config file.

    <system.web>  
      <compilation xdt:Transform="RemoveAttributes(debug)" />  
      <customErrors xdt:Transform="SetAttributes(mode)" mode="On" />  
      <trace xdt:Transform="InsertIfMissing"/>  
      <trace xdt:Transform="Remove"/>  
    </system.web>  

    <system.webServer>  
      <handlers xdt:Transform="InsertIfMissing" xdt:SupressWarnings="true">  
        <remove xdt:Transform="InsertIfMissing" name="TraceHandler-Integrated"/>  
        <remove xdt:Transform="InsertIfMissing" name="TraceHandler-Integrated-4.0"/>  
      </handlers>  
    </system.webServer>  

This will ensure _debug_ mode is turned off for the application as well as turning on _customErrors_ to prevent leaking of information through error pages.

## Add security headers

Once the information leak has been plugged, it's time to start adding headers which control the allowed behaviors for browsers. This includes considerations like XSS protection, iFrame behaviors, where content is allowed to be loaded from and how it's able to behave, and others. All these behaviors can be dictated by headers sent by the application and obeyed by the major web browsers.

These headers are not a substitution for secure coding practices but are the front line in a layered defense.

- **Content-Security-Policy**
- **X-Frame-Options**
- **X-XSS-Protection**
- **Referrer-Policy**
- **X-Content-Type-Options**

These headers are set in the system.webServer�httpProtocol�customHeaders section of the web.config.

    <system.webServer>  
      <httpProtocol>  
        <customHeaders>  
          <add name="Content-Security-Policy"  
               value="default-src 'self' https://intlcdn.net http://intlcdn.net;  
               script-src 'self' https://intlcdn.net http://intlcdn.net;  
               style-src 'self' https://intlcdn.net http://intlcdn.net;  
               img-src 'self' https://intlcdn.net http://intlcdn.net"/>  
          <add name="X-Frame-Options" value="DENY">  
          <add name="X-XSS-Protection" value="1; mode=block" />  
          <add name="Referrer-Policy" value="no-referrer" />  
          <add name="X-Content-Type-Options", "nosniff" />  
        </customHeaders>  
      </httpProtocol>  
    </system.webServer>  

Alternatively, in the case of an ASP.NET Core application, these can be set using middleware.

    app.Use(async (context, next) => {  
        context.Response.Headers.Add("Content-Security-Policy",  
          "default-src 'self' https://intlcdn.net http://intlcdn.net; " +  
          "script-src 'self' https://intlcdn.net http://intlcdn.net; " +  
          "style-src 'self' https://intlcdn.net http://intlcdn.net; " +  
          "img-src 'self' https://intlcdn.net http://intlcdn.net;" +  
        );  
  
        context.Response.Headers.Add("X-Frame-Options", "deny");  
        context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");  
        context.Response.Headers.Add("Referrer-Policy", "no-referrer");  
        context.Response.Headers.Add("X-Content-Type-Options", "nosniff");  
        await next();  
    });  

### Content-Security-Policy

_Content-Security-Policy_ is a very extensive header that can have several different directives applied to it. Each directive includes the directive name followed by a space delimited list of values and terminated with a semicolon.

The example above shows four separate categories being set and allowing sources from 'self' (the web application itself), http://intlcdn.net and https://intlcdn.net. There are a large number of options that can be set with _Content-Security-Policy_ and following the most restrictive settings the application can support.

Determining what settings to include can be daunting, but _-report-only_ can be appended to the header to test your settings. Violations will appear in the Developer Tools Console for areas on the site that would have been blocked if in enforcement mode. This can also be used with a _report-uri_ directive to direct all violations to an endpoint that can log them if testing needs to be longer term.

A full reference can be found at [https://content-security-policy.com/](https://content-security-policy.com/).

### X-Frame-Options

Browsers support the ability to display one webpage inside another by using iframes. The _X-Frame-Options_ header allows an application to tell the browser what iframe behavior it will allow. Options include:

- **deny**  never allow this content to be displayed in an iframe
- **sameorigin**  only allow this content to be displayed in an iframe of another page on the same origin as the content being presented.
- **allow-from uri**  allow this content to be displayed in an iframe only on the specified uri.

Without specifying this header, another site can create an iframe of specific dimensions and transparency to trick a user to click on a button on another site. This is called _clickjacking_.

### X-XSS-Protection

Cross-site scripting or XSS is a type of injection attack that utilizes malicious scripts to cause a website to act outside of its normal behavior. The _Content-Security-Policy_ header will prevent most XSS attacks if 'unsafe-inline' is not included. However, for older browsers, _X-XSS-Protection_ serves as the mechanism to instruct the browser what to do when an XSS attack is detected. Options include:

- **0**  Disables XSS filtering
- **1**  Enables XSS filtering and if an attack is detected, the browser will sanitize the page
- **1; mode=block**  Enables XSS filtering and if an attack is detected, the browser will prevent the rendering of the page
- **1; report=**** uri**  Only available in Chromium, Enables XSS filtering and if an attack is detected, the browser will sanitize the page and report the violation

### Referrer-Policy

The _referer_ header (purposely misspelled) provides the information of what site the browser was on when the request was made. This can result in privacy violations by providing too much information to the server and thus captured by any agent between the browser and server. To avoid this, the _Referrer-Policy_ (spelled correctly) is provided with one of the following directives:

- **no-referrer** The referer header will be omitted entirely.
- **no-referrer-when-downgrade** - The URL is sent as a referrer when the protocol security level is the same, but is omitted when sending to a less secure destination
- **origin** - Only send the origin as the referrer in all cases.
- **origin-when-cross-origin** - Send a full URL when performing a same-origin request, but only send the origin of the document for other cases.
- **same-origin** - A referrer will be sent for same-site origins, but cross-origin requests will contain no referrer information.
- **strict-origin**  The origin is sent as a referrer when the protocol security level stays the same, but is omitted when sending to a less secure destination
- **strict-origin-when-cross-origin**  The URL is sent when performing a same-origin request, when the protocol security level stays the same, but omitted when sending to a less secure destination
- **unsafe-url** - Send a full URL when performing a same-origin or cross-origin request.

### X-Content-Type-Options

Browsers have the ability to provide a "best guess" at MIME type in the case where the _content-type_ header is missing, or the browser detects it could be incorrect. This is called MIME type sniffing and can lead to browsers transform non-executable MIME types into executable MIME types. This should always be set to _nosniff_. This helps prevent script files from being marked as sources for IMG tags and thus being executed on load.

### Strict-Transport-Security

HTTP Strict Transport Security tells a browser to register this site as only ever being available over HTTPS. When a response includes this header, the browser registers this site internally and any request to HTTP for this site is immediately replaced with HTTPS for a time period determined by the _max-age_ directive. This helps mitigate man-in-the-middle attacks.

In ASP.NET hosted on IIS add the following custom header in the web.config:

    <system.webServer>  
      <httpProtocol>  
        <customHeaders>  
          <add name="Strict-Transport-Security", "max-age=31536000" />  
        </customHeaders>  
      </httpProtocol>  
    </system.webServer>  

In ASP.NET Core there is a built-in middleware that can be called using the following:

    app.UseHsts();  

## Sessions, Cookies, and Default Documents

### Sliding Expiration

_Sliding expiration_ is a setting which instructs the framework to issue a new cookie with a new expiration any time a request is processed which is more than halfway through the expiration window.

If this is set to _true_, it is conceivable to never have to re-authenticate a user. If the user's cookie, browser, or computer were to be compromised this would allow an attacker to never have to know the user's credentials and remain able to act on behalf of the user.

In ASP.NET this is controlled on the _Authentication->Forms_ element in the _web.config_.

    <system.web>  
      <authentication mode="Forms">  
        <forms defaultUrl="~/"  
               loginUrl="~/openid"  
               name="cookieName"  
               cookieless="UseCookies"  
               slidingExpiration="false"/>  
      </authentication>  
    </system.web>  

In ASP.NET Core, this is achieved by setting the _SlidingExpiration__property_ on the Authentication.Cookies object.

    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)  
        .AddCookie(options => {  
        Options.SlidingExpiration = false;  
    });  

This setting should always be set to _false_.

### HttpOnly Cookies

If not specified otherwise, all cookies on a site are able to be accessed by JavaScript running on a page from the same domain. This allows any attacker to inject custom JavaScript and read the cookies passed between browser and server. These cookies often contain sensitive data and should not be able to be read. To prevent this from being possible, setting called _httpOnly_ needs to be set.

In ASP.NET, this is done in the _web.config_.

    <system.web>  
      <httpCookies httpOnlyCookies="true"/>  
    </system.web>  

In ASP.NET Core, if cookie authentication is used, this is done by default.

    app.UseCookieAuthentication();  

Or, if you wish to set a cookie manually, you can do that as middleware.

    app.Use((context, next) => {  
        context.Response.Cookies.Append(  
            "CookieKey",  
            "CookieValue",  
            new CookieOptions {  
                HttpOnly = true  
            });  
        await next();  
    });  

### SessionState Cookie

Any time an application requires the ability to track a user across multiple requests, session state is used. This will create a unique session id and store it in a cookie. By default, this cookie is called _ASP.NET\_SessionId_. This makes it trivial to determine that the site is running ASP.NET. This is one more piece of information that should be hidden from any potential attacker.

In ASP.NET this can be changed in the _web.config_.

    <system.web>  
      <sessionState cookieName="s" cookieless="UseCookies" />  
    </system.web>  

In ASP.NET Core, this can be changed in the _Startup_ class.

    public void ConfigureServices(IServiceCollection services) {  
        services.AddSession(options => {  
            options.Cookie.HttpOnly = true;  
            options.Cookie.IsEssential = true;  
            options.Cookie.Name = "s";  
        });  
    }  

    public void Configure(IApplicationBuilder app, IHostingEnvironment env) {  
        app.UseSession();  
    }  

If it is unnecessary for your application to track a user across multiple requests, it's best to completely disable session state tracking. In ASP.NET Core, session state is turned off by default, but in ASP.NET it can be turned off in the _web.config_.

    <system.web>  
      <sessionState mode="Off"/>  
    </system.web>  

### AntiForgery Cookie

Anti-forgery is a mechanism that helps mitigate the risk of _cross-site request forgery._ By writing a unique value to a cookie and the same value to a form element, the server-side code can validate that the values match and confirm that the request hasn't been forged by another site. By default, the antiforgery cookie is named _AntiForgeryToken_ in ASP.NET Framework and _.AspNetCore.Antiforgery._ **xxxxxxx** in ASP.NET Core. Leaving these to be the default values would allow potential attackers to determine the site is running ASP.NET. This should be hidden from an attacker.

In ASP.NET this can be changed in the _web.config_.

    <system.web>  
      <sessionState cookieName="s" cookieless="UseCookies" />  
    </system.web>  

In ASP.NET Core, this can be changed in the _Startup_ class.

    public void ConfigureServices(IServiceCollection services) {  
        services.AddSession(options => {  
            options.Cookie.HttpOnly = true;  
            options.Cookie.IsEssential = true;  
            options.Cookie.Name = "s";  
        });  
    }  
  
    public void Configure(IApplicationBuilder app, IHostingEnvironment env) {  
        app.UseSession();  
    }  

### Default Documents

ASP.NET has a list of documents that are served, in a priority order, if no other route is provided. Examples of these files are:

- default.htm
- default.asp
- default.aspx
- default.html
- index.htm
- index.html
- iisstart.htm

Generally speaking, if the application has the proper routes set up, default documents are unnecessary. If default documents are enabled, they can provide an opportunity for an attacker to take over a part of a site.

ASP.NET Core does not have default document handling turned on by default.

To turn off default documents in ASP.NET is a simple change to the _web.config_ file.

    <system.webServer>  
      <defaultDocument enabled="false"/>  
    </system.webServer>  