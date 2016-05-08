# OWASP Top 10 #4 - Insecure Direct Object References #

## Direct Object Reference ##

A direct object reference is a key or id which is used identify a piece or set of data. The data would typically be a record within a a table of a database.

They can be:

- Patterns such as incrementing integers/ids
- Natural keys such as a username or email address.
- Discoverable data such as a national insurance number or a social security number.

## What Is it? ##

An insecure direct object reference is when a user accesses a direct object reference when they are not permitted to. 

This can be from an unsecured page and an unauthenticated user. It can also come from a secured page and authenticated user, where there is simply insufficient checks that they can view the data they have requested.

Quite often the user guesses or enumerates a direct object reference either manually or by an automated script.

If the data should only be visible to a subset of people then any requests to that data should be validated that they are permitted to see that data. Simply relying on not displaying links to the user is not secure enough.

## How Can I Protect Against It? ##

### Validate Access ###

The first point to address is to  validate that the user has permission to access the data.

For example; is the person permitted to see the financial records of the bank account in question?

Quite often the validation checks are placed around populating the UI to navigate to the data but no checks are placed around querying the data.

### Indirect Reference Maps ###

A normal approach to view database records on a web page is to expose the table fields and their direct object references on the URL

```
https://www.MySite/Foo.aspx?id=1
```

Here an attacker can simply increment the id to try and view other peoples Foo which are simple incrementing ids or identity indexes generated automatically by the database.

An indirect reference provides an abstraction between what is publicly observable in the URL and what is used to uniquely identify a record within a database.

A dictionary is used to store a map between the id and a randomly generated cryptically secure identifier. We then pass these secure identifier to the user as part of URLs which they can request.

```
https://www.MySite/Foo.aspx?id=ASWERDFMJSDGJSERSASKAWEOSDSDF
```

These keys should also be mapped to the user's session to protect against the ids being leaked:

The following code shows how to generate a cryptically safe unique id using the RNGCryptoServiceProvider class.

```
using System.Security.Cryptography;

using( var rng = new RNGCryptoServiceProvider() )
{
	map[id] = HttpServerUtility.UrlTokenEncode(rng.GetBytes(new byte[32]));
}

// Store the map within the users session
HttpContext.Current.Session["Map"] = map;
```

In the example above it would be best to only create one instance of the RNGCryptoServiceProvider class regardless of how many direct object references are being made. Consider creating a class which implements IDisposable which calls Dispose upon the instance of RNGCryptoServiceProvider within it's Dispose method.

### Obfuscation via non-discoverable surrogate keys###

A surrogate key is a set of fields which when used together can uniquely identify a record. These are harder to guess than a single column which has a distinctive pattern such as an identify index which increments by one each time. 

Surrogate keys would require more space to index and are also slower, it is a trade off between performance and security.

A random unique identifier such as a GUID is harder to guess but does not perform as well as an integer when used to search upon. Also for tables such as logs and audits which are used to generate a high number records, the indexes will become fragmented and as such will have more overhead for de-fragmenting them regularly.