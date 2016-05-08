# OWASP Top 10 #3 - Broken Authentication & Session Management #

How to secure .NET websites from broken authentication and session management security attacks.

## Sessions In A Stateless Protocol ##

To understand broken authentication attacks such as session hijacking it is important to understand how a user is logged in and kept logged in within the stateless protocol HTTP.

HTTP is a stateless protocol, each request is independent of every other request. Any knowledge of previous requests and their contents is maintained outside of the bounds of the protocol.

ASP.NET maintains session data either on the client side in browser cookies or on the server side in memory or within a database and ties the data to the user via session which is identified by unique identifier.

Web sites which require a user to log in make use of a unique identifier or session token which is created during then authentication process, associated or linked to the users account and then passed between the users web browser and the web server via a cookie.

## What Is It? ##

If the authentication session token is discovered by a potential attacker, it can be used to steal the users session. This can be by either a session fixation or a session hijack.

Session hijacking is when an attacker gets hold of a session identifier and uses this to gain access to a users session and hence impersonates them.

Session fixation is when a user who is logged in to a site overrides their session identifier set in a cookie by passing in another session identifier within the query string; thus pretending to be someone else with potentially elevated user access.

Session ids can be passed between client and server by a query string parameter within a URL as well as cookies and form data.

## How Can I Protect Myself From It? ##

### Do Not Persist Session Tokens In The URL ###

Seeing as URLs are logged, shared and retrievable from a browser's history; the first line of defence is not to persist session tokens in URLs, you should favour cookies. 

**Note** that a default form post action is GET; I have fixed many security issues which were caused simply by forgetting to set the action to POST.

### Persist Session Tokens In Cookies  ###

To get .NET to persist the session token within cookies you can set the cookieless attribute of the sessionState node to UseCookies within the web.config.

```
<system.web>
  <sessionState cookieless="UseCookies" />
```

Other options of the variable are:

- UseUri
	- Always use the URL regardless of device support
- UreCookies (default)
	- Always use cookies regardless of device support
- UseDeviceProfile
	- ASP.NET determines if cookies are supported in the browser and falls back to URL if not.
- AutoDetect
	- ASP.NET determines if cookies are enabled in the browser and falls back to the URL if not.

**Note** there is a difference between enabled and supported.

**Note** you should perhaps question whether users who are using browsers with no cookie support or with cookies disabled should be allowed to use your site.

### Secure Cookies ###

All cookies should be set to HttpOnly; this prevents the browser from reading the contents and as such reduces the chance of their contents being read by malicious code.

Cookies which contain session data should be configured to be served over SSL connections only.

## Use ASP.NET Membership Provider For Authentication ##

Whenever security is concerned, it is important to use tried and tested frameworks in favour of writing your own.

The .NET membership provider is built into ASP and should be favoured. 

It contains the following functionality:

- Automatic schema creation upon first usage
- Default website projects which are set up to allow
	- Creating user accounts
	- User login/log out
	- Session persistence between page requests
- Automatic page/URL protection from non logged in users or users not within a user group
- User groups for a more granular permissions

### Automatically Logout Expired Sessions ###

A session token should only be considered valid for the minimum viable amount of time possible. If a session token is leaked the smaller the time window it is exploitable reduces the chance of the session being hijacked.

You can set the forms login session length in minutes within the web.config.

```
<forms loginUrl="~/Account/Login" timeout="30" />
```

By default the timeout is sliding, it expires after the configured amount of time has passed after the last request has been made.

It is possible to change the timeout to be fixed, expiring after the configured amount of time has passed after the user has logged in.

```
<forms slidingExpiration="false" />
```

### Other Checks ###

- Users credentials should always be stored in a cryptographically secure way
- Enforce users to create passwords with a minimum standard of complexity
- Never send a password by email
 - Always implement a secure password reset process
 	* http://www.troyhunt.com/2012/05/everything-you-ever-wanted-to-know.html