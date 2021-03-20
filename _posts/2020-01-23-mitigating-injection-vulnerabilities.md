---
layout: single
title:  "Mitigating Injection Vulnerabilities"
date:   2020-01-23 12:00:00 -0500
categories: appsec software injection XSS SQLi 
author: XanderK
---

# Mitigating Injection Vulnerabilities

## What is an Injection Vulnerability?
Any vulnerability allowing an attacker to “inject” content into code, HTML, database queries, etc. is considered an injection vulnerability. The two exploits of injection vulnerabilities that will be covered here are Cross-Site Scripting (XSS) and SQL Injection (SQLi). XSS injects script content into the HTML to be rendered by the browser and SQLi injects content into a database query to modify the behavior of the query executed by the database engine.

Exploiting these vulnerabilities can generate effects which range from website defamation, credential harvesting, session hijacking, unauthorized transactions, data exfiltration, data modification, and so much more.

## Trust 
There are several different areas where injection vulnerabilities are addressed, but mitigation starts with the design of the application and establishment of trust boundaries. A trust boundary is a boundary which, when crossed, changes the level of trust. These are areas where the design needs to evaluate the data crossing the boundary and if additional data validation is required. 

As an example, an internal service which can only be called by the presentation layer may not add additional data validation to the parameters it receives. However, an API endpoint, which may be called from many different applications, would need strict data validation performed. This is because the API cannot assume validation has been done sufficiently by the calling application, whereas the internal service is dedicated to the presentation layer.

Any ingress into the application is a trust boundary and any data should be untrusted unless there is sufficient reason to trust it. This includes both execution paths and data paths. If data is being pulled from an external source, that is also a trust boundary being crossed and requires additional data validation to be performed.

## Data Validation Strategies
### Reactive Validation
Data validation performed on data crossing a trust boundary from a trust partner is referred to as reactive. It is reactive because the agent performing the data validation has no control over the data coming in. It is forced to react to what is being presented.  

Examples of this include escaping strings, parameterizing SQL queries, validating data ranges, etc.

### Proactive Validation
Proactive Data Validation, on the other hand, refers to making design decisions that result in the simplification of data validation concerns when data crosses a trust boundary to a trust partner. 

An example of this is Indirect Selection. Simply stated, it is the pattern of representing a limited number of options as indexes instead of strings. This simplifies the data validation and makes it more for an attacker to manipulate.

### Client-side vs Server-side
It is important to understand the difference between client-side and server-side data validation and the roles they play. 

Client-side validation refers to any data validation conducted by the client, prior to sending the data to the server. This can take place in JavaScript code in a web application, or a package from NuGet, NPM, Ruby Gem, pip, etc. It is important to understand that client-side validation exists for the benefit of usability and provides no security function. Because it is happening on the client, it can be bypassed. Client-side validation takes place prior to sending data.

Server-side validation is the other side of the partnership. When the server receives data, it must perform validation on it to ensure security and data quality standards. The server must assume that no data validation has taken place on the data being received.

## Filtering Input
Filtering user input is not as simple as it sounds. There are many ways input can be encoded, escaped, or otherwise obfuscated that translate to malicious strings. Another challenge is the number of combinations that can lead to XSS or SQL injection.

There are two strategies to implement filtering: Blacklisting and Whitelisting.
-	Blacklisting is explicitly blocking or sanitizing specific characters or character combinations. This requires the use of a carefully crafted set of values to identify.
-	Whitelisting involves identifying the only input considered to be valid and rejecting all other input. This option provides the most future-proofed solution as any new attack methods will be automatically ineffective if the input doesn’t match the whitelist.

When considering the best strategy for filtering input, several factors need to be considered:
-	Is the field a free-form text field?
-	Are there a limited number of valid choices?
-	Is the value intended to be a number?
-	Will excluding certain characters have an impact on valid data?

Blacklisting characters and strings can get very difficult to maintain as new threats emerge. It can also be difficult to prevent bypass attacks by encoding differences. However, blacklisting may be the best option if your input fields must accept a wide range of data values. 

Whitelisting involves restricting the input to a defined list of values. This is the preferred option when it’s possible. By only accepting a limited set of values, as the threat landscape changes, the filtering strategy doesn’t have to change.  
Implementation

There are several viable strategies available when deciding how filtering is going to be implemented in your application architecture. To understand the impact of each strategy, it’s important to know when a strategy is triggered in the request pipeline and what the framework being used provides the development team.

## Request Pipeline Stages
There are several areas during an HTTP request where filtering of data can take place. These include the web server, middleware, model binding, and the endpoint of the application itself.
### Web Server
Requests are frequently filtered initially at the web server, before being routed to an application. This can include areas such as denial-of-service attack mitigation, HTTP header, or filtering out suspected XSS and SQLi content in the body of the request. The benefit of applying rules at the server level is that you avoid having to keep track of each area where rules need to be applied. However, this also means that exceptions to those rules become difficult to implement. 
### Middleware
Another area where filtering can be applied is using middleware. Middleware is a term referring to code which executes between the web server and the web application in the request pipeline. This code can execute before the request reaches the application code, after the application has handled the request, or both. This provides the opportunity to filter data being presented to the application as input and being returned to the client as output. The drawback to this is that it can become difficult to handle field-level rules for input filtering. If the endpoints of an application need to have different validation rules, those rules would have to be added to the middleware as an exception. 
### Data Annotations/Property Decorators
Model validation takes place in different frameworks in different ways. Most frameworks either have a built-in model validation mechanism, or one is available through a third-party package. The most flexible and reusable forms involve the use of property decorators or data annotation attributes. Applying these decorators to properties sets the rules that determine the validity of the property value. 
### Application Endpoint
The last place that filtering and sanitization can take place is within the application endpoint itself. This is certainly flexible but does not create a centralized and repetitive way to apply filtering. Filtering at this level creates code that is not maintainable and does not follow the principle of DRY (Don’t Repeat Yourself). As filtering rules need adjusted, each endpoint will have to be modified to support the change needed.

## Conclusion
Model validation with data annotations provides the best solution for data validation for frameworks that support it. For applications that can’t utilize built-in model validation, the same pattern should be implemented to provide the most flexible and maintainable validation mechanism.

## Additional Model Validation Information
### ASP.NET
https://docs.microsoft.com/en-us/aspnet/mvc/overview/getting-started/introduction/adding-validation 
### ASP.NET Core
https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-3.1
### NodeJS
https://github.com/xpepermint/contextablejs
https://github.com/chriso/validator.js
https://www.npmjs.com/package/iz
### Ruby
https://api.rubyonrails.org/classes/ActiveModel/Validations.html

Please reach out to the Application Security Team for additional help.