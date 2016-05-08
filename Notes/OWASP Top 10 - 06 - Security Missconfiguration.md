# OWASP Top 10 #6 - Security Misconfiguration#

## What Is It? ##

Security misconfiguration is anything which is considered insecure due to the settings or configuration upon a server.

This can be anything from debug settings being turned on or even out of date third party frameworks being used.

## Custom Errors ##

Leaking of information about the implemented technology stack could be an invitation for hackers to probe a system further. With regards to an SQL injection attack, error messages can be used to view details about the database schema such as table names and then further be used to display the contents of a database table.

Error messages displayed to a user should contain information useful to the user only and nothing else. They should report to the user the minimal amount information for them to react to the error. 

Any information such as details of the error or the trace stack should be hidden from the user.

We can disable the standard .NET  'yellow' error message which contains the trace stack and the details of the error on a site basis. This is done within the customErrors node of the web.config

```
<customErrors mode="On" defaultRedirect="Error.aspx" />
```

The mode attribute can contain the following values:

- On
	- Errors are reported in full. This is the default value.
- Off
	- Errors are not reported. The user is redirected to the page defined within the defaultRedirect attribute
- RemoteOnly
	- Users viewing the site on their local host are reported the errors in full
	- Users not viewing the site on their local host are redirected to the page defined within the defaultRedirect attribute.

If no defaultRedirect is provided the user is presented with a http response code of 500. This in itself is bad as it highlights our site as a potential site worthy of more probing by a hacker. The status code is easily read by batch scripts probing sites.

Providing a defaultRedirect presents the user with the page defined by the attribute along with a http status code of 302 (temporary redirect). This unfortunately creates a query string parameter of aspxerrorpath set to the relative URL of the page which caused the error. This can also be easily used by batch scripts to determine if a page is an error page.

We can redirect to the error page without the indicating that there was an error by changing the RedirectMode from ResponseRedirect (default) to ResponseRewrite.

```
<customErrors mode="On" defaultRedirect="Error.aspx" RedirectMode="ResponseRewrite" />
```

With this set, the contents of the error page is simply set to the response of the request without a redirect. The http status code is set to 200 which is the OK code. The hacker would need to read the contents of the page to try and determine if the page is an error page. This in itself is harder and slower. Anything we can do to slow down the hacker reduces the amount of hacking they can do on our site.

## Disabling Trace ##

Trace information allows collecting, viewing and analysis of debug information that is collected and cached by ASP.NET. The information can be viewed at the bottom of individual pages or within the trace viewer which is accessed by calling the Trace.axd page from the route directory of the web site.

Like any debug information it contains details of implemented technology and potentially sensitive information that might be logged by accident. There are many instances of security issues where personal information such as credit card details or user accounts used in database connection strings which are logged as part of the debugging.

Tracing should be disabled within the web.config for all production environments.

```
<trace enabled="false" />
```

## Keep Frameworks/Third Party Code Up To Date ##

All third party frameworks should be kept up to date to ensure that you are protected from any security issues which have been fixed within them.

With the use of NuGet this can be achieved very easily.

## Encrypting Sensitive Parts Of The Web.Config ##

Sensitive data such as user accounts, passwords and private keys that are contained within the web.config should be encrypted.

If they are accidentally exposed to the user during an error while accessing a page or logged into an audit table and viewed by someone who should not see them, their details will not be readable.

It is also a good idea to reduce the amount of people who have access to sensitive information.

You can encrypt a node of a web.config at the visual studio command prompt by the following:

```
aspnet_regiis -site "Site Name In IIS" -app "/" -pe "connectionStrings"
```

This will update the web.config replacing the sensitive information with an encrypted version.

The file can be decrypted with the following command:

```
aspnet_regiis -site "Site Name In IIS" -app "/" -pd "connectionStrings"
```

This process uses a key which is created based upon the servers MAC address and as such is unique to the server and therefore can only be decrypted upon the web server.

## Using Config Transforms To Ensure Secure Configurations ##

It can be very easy to leave debug settings such as non custom errors and tracing within a web.config.

Transforms for debug and release can be used to ensure that only settings for the required release are used when that release target is used.

Use Web.Release.Config to transform the Web.Config for release:

- Remove debug attributes
- Add custom errors
- Remove trace

These transforms are only run during a site publish.

## Retail Mode##

Retail mode can be used as a back stop to ensure that no web.config setting which is considered unsafe in a production environment is actually enabled. For example in retail mode, turning off custom errors or turning on tracing will have no effect.

This setting can be set in the web.config though it is recommended to set it within the machine.config of the web server.

```
<deployment retail="true" />
```