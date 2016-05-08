# OWASP Top 10 #9 - Insufficient Transport Layer Protection #

##  What Is It? ##

Any communication between a client and a server can be intercepted in a number of ways. If the communication is not encrypted then the information can be freely read, worse still it can be manipulated.

### What Is a Secure Connection? ###

HTTPS is a secured version of HTTP. The connection sends data which is encrypted. Third-party organisations are responsible for the certification of the connection and the distributing of the public keys used to decrypt network packets.

Only the owner with the private key can encrypt the data and certify that they are the true originator of the site.

HTTPS is often referred to as TLS and SSL but they are basically the same thing.

### Man In The Middle Attack ###

A man in the middle (MITM) attack is when someone intercepts and potentially manipulates network packets.

This type of attack can be implemented by:

- Physically putting in a cable onto the network
- Intercepting traffic at the ISP level
- Monitoring unprotected traffic at a WiFi hotspot
- Creating a rogue wireless access point

## How Can I Protect My Self From It? ##

It is important that all communication between a server and a client is secure.

### Enforce Forms Authentication To Be Secure ###

Enforce forms authentication to be secure by setting the requireSSL attribute to true upon the authentication node within the web.config.

```
<authentication mode="Forms">
	<forms loginurl = "~/Account/Login" timneout"2880" requireSSL="true" />
</authentication>
```

### Enforce Cookies To Be Secure ###
Cookies can be enforced over a secure connection with the requireSSL attribute of the httpCookies node within the web.config

```
<system.web>
	<httpCookies requireSSL="true" />
</system.web>
```

Cookies can individually be set to secure via the secure property of the cookie during its creation:

```
Response.Cookies.Add(new HttpCookie("MyCookie")
{
	Value = "My value",
       Secure = false
});
```

### Redirect To HTTPS If HTTP Is Detected ###

We can enforce certain pages or resources to always be secure by detecting if the connection is not secure and then redirecting the user to the HTTPS version of the page or resource.

```
if(!Request.IsSecureConnection)
{
	var secureUrl = Request.Url.ToString().Replace("http://", "https://");
    	Response.RedirectPermanent(secureUrl);
}
```

Hooking this into an early page event or even a http module might be a good idea for this code.

MVC has the RequireHttps attribute. Again the user will be redirected to the HTTPS version of the URL if they try to access the site by HTTP:

```
public class MyController 
{	
    [RequireHttps]
	public ActionResult ViewAsSecureOnly() {
    }
}
```

### Mixed Mode HTTPS ##

Mixed mode is when a page is served via HTTPS but some of the resources on the page such as JavaScript libraries or CSS files are served via HTTP.

The browser should warn about the site not being fully secure.

Always serve all content via HTTPS regardless if the information contained is sensitive. HTTPS is not just about encrypting the communication between server and client, it is also about ensuring the authenticity of sender of a resource and therefore the validity of its content.
When resources are served via third party servers such as common frameworks served over CDNs, favour their secure URLs. Most CDNs provide HTTP and HTTPS URLS for their content.

Where content is hosted upon your website use protocol relative URLs. This ensures that the same protocol is used for the resource as was used to request the page.

```
// Protocol specific
<script src="http://MyOtherDomain/Resources/code.js" type="text/javascript"></script>

// Protocol relative
<script src="//MyOtherDomain/Resources/code.js" type="text/javascript"></script>
```

When a browser sees an address which starts with //, it will automatically prefix it when the scheme that the parent page was called with.

### HSTS Headers ###

A page response with HSTS or strict transport security within its header will prevent any additional resources be requested with via HTTP.

This can be set within the customHeaders node of the web.config. Here it will affect all pages served by the server.

```
<system.webServer>
    <httpProtocol>
        <customHeaders>
            <add name="Strict-Transport-Security" value="max-age=31536000"/>
        </customHeaders>
    </httpProtocol>
</system.webServer>
```

Max age is the length of time the header is applicable for in the cache of the client browser and is defined in seconds.

This can also be set on a page by page basis using the AddHeader method of the HTTP response class:

```
HttpContext.Current.Response.AddHeader("Strict-Transport-Security", "max-age=31536000");
```

Unfortunately browser support is currently a little patchy, for example only version 11 of internet explorer supports this setting.