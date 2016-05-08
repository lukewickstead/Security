#OWASP Top 10 #10 - Unvalidated Redirects and Forwards #

## What Is It? ##

If a user sees a domain they trust, they are more than likely to feel secure. Attackers use this false sense of security to launch their malicious site via means such as XSS. 

## How Can I Protect Against It? ##

### Implement A White List ###

A white list is simply a list of known good values which can be used. Consider implementing a registry of valid domains and pages which your site is allowed to redirect to and then validate against this before redirecting.

```
if(!IsTrustedUrl(url))
{
	throw new ApplicationException("URL is not trusted");
}

Response.Redirect(url);
```

### Referrer Checking ###

Where redirects are only permitted to be made from your domain and to your domain, a referrer check can be used to validate this, Preventing your site being used as a redirect proxy.

```
var referrer = Request.UrlReferrer;

if(referrer == null || referrer.Host != Request.Url.Host)
{
	throw new ApplicationException("Referrer is not the same site")'
}

Response.Redirect(url);
```

The Request.UrlReferrer will be null if the page requesting the redirect was not from our domain.

**Note** it will also be null if HTTPS is used.