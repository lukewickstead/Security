# OWASP Top 10 #5 - Cross Site Request Forgery (CSRF)#

## What Is It? ##

Before we talk about CSRF it is important to understand that all cookies created by a domain are sent back to that domain during page requests regardless which domain the page originated from. This includes cookies which contain authentication session cookies.

Hackers can simply change the action attribute of a form to be that of the domain/URL they are trying to breach. An unsuspecting site would only ensure that the authentication session token is valid. It would therefore be possible to use the users open session to perform malicious actions such as saving or reading data from a database.

## How Can I Protect My Self From It? ##

Protection from CSRF attacks has a simple solution. Send another cryptically secure token to the user along with the authentication session token. This token is sent both on the page being generated as well as a cookie.

Upon receiving a post back from a page, the server simply reads the CSRF token from the page and the cookie and ensures that they are identical. 

As only a page generated from the site would contain the token, any fake page posting back to the server will be missing the correct CSRF token.

### MVC ###

The MVC helper method AntiForgeyToken generates a  CSRF token and persists it in a cookie and a hidden input element on the page.


```
using (Html.BeginForm())
{
  @Html.AntiForgeyToken()
}
```

Validation against a CSRF attack is made by decorating the controller's action method with the ValidateAntiForgeryToken attribute.

```
[HttpPost]
[Authorise]
[ValidateAntiForgeryToken]
public ActionResult Index(McModel bigMac)
{
}

```

If a CSRF attack is caught a System.Mvc.HttpAntiForgeryException is thrown and the user is presented with a 500 status code.

### ASP.NET Web Forms ###

A page written in ASP.NET web forms would traditionally need to manually perform the CSRF token, its persistence and checks to ensure they are identical when the page is posted back.

However we can now make use of the AntiForgery class.

https://msdn.microsoft.com/en-us/library/system.web.helpers.antiforgery%28v=vs.111%29.aspx

Within the aspx page a call to the GrtHtml method would generate the CSRF token which is then persisted within a cookie and an input element upon the page.

```
<%= System.Web.Helpers.AntiForgery.GetHtml() %>
```

Validation against a CSRF attack would then be performed during a page post back with the Validation method.

```
protected void Page_Load(object sender, EventArgs e)
{
	if (IsPostBack) {
 		AntiForgery.Validate();
	}
}
```

### Securing Cookies ###

#### Http Only Cookies ####

Using AntiForgery.GetHtml or @Html.AntiForgeyToken should create cookies which are secured for access by the server only and not via a client side script. 

However where a more manual approach has been made to CSRF protection it is important to ensure that the CSRF token cannot be read from by a client side script. If this is not the case then a malicious page could read the token and simply add it to its form data before posting back to the server; circumventing our protection.

Cookies can be set as HttpOnly via a property during their creation:

```
var cookie = new HttpCookie("MyCookies", DateTime.Now.ToString());
cookie.HttpOnly = true;
```

Cookies can be set as HttpOnly globally for a website via the web.config:

```
<httpCookies domain="String" httpOnlyCookies="true" />
```

**Note** is it probably wise to set httpOnlyCookies to true regardless of your CSRF approach. Any information stored within a cookie which could be readable on the client side is probably a bad idea.

### SSL Only Cookies ###

Where cookies contain sensitive information such as CSRF or authentication session token, it would be wise to enforce their transportation via SSL or HTTPS, as such preventing people from spying on the network traffic and therefore stealing them.

This can be done globally for a site within the web.config:

```
<httpCookies domain="String" httpOnlyCookies="true" requireSSL="true" />
```

However this approach might not always be possible. We can conditionally determine when to enforce SSL communication for cookies by checking the forms authentication settings and the connection for their status of secure connections. The individual cookie can then then be secured for SSL communication only via the Secure property.

```
var cookie = new HttpCookie("MyCookies", DateTime.Now.ToString());
cookie.HttpOnly = true;

if( FormsAuthentication.RequireSSL && Request.IsSecureConnection) {
	cookies.Secure = true;
}
```