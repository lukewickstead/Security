# OWASP Top 10 #8 - Failure To Restrict URL Access #

## What Is It? ##

Failure to restrict URL access is when someone can view a resources via a URL who is not permitted to see that resource.

This normally comes from the URL being hidden from the UI but no effort has been made to prevent the user viewing the URL when they navigate to it directly via the address bar of the browser.

## How Can I Protect Myself From It? ##

Protection from this is quite simple but quite often overlooked. The first point to note is that simply removing or hiding any reference to the URL from the UI is simply not good enough. Efforts should be made to prevent the user from viewing the URL itself.

###Web.Config Locations ###

ASP.NET allows setting permissions for URLs within the location node of the web.config.

The benefit of adding permissions here rather than manually on each page is that it consolidates all the permissions of the site into one common place.

```
<location path="DangerousPage.aspx">	
    <system.web>    	
        <authorization>
        <deny users="?" />
        <allow roles="Admin,Backoffice" />
        </authorization>       
    </system.web>
</location>    
```

The URL is defined within the path attribute of the location node. Here the path is mapped to the page DangerousPage.aspx which is defined as a relative path from the website route.

The authorization node allows permissions to be set by both denying and allowing. 

We deny any user who is not logged in as denoted by the '?' character of the deny node.

We then add permitted users with the allow roles attribute of the allow node. Here we allow users who are in the role Admin or Backoffice. All other users will be redirected to the configured login page.

**Note** you should always start by denying everyone before then adding in permissions to permitted users.

**Note** the one draw back here is that we set permissions for URLs. Where routing is used such as in MVC,  multiple URLs can be defined for a resource such as a controller. We would need to permission each URL which can be used to access a resource. This abuses the principle of DRY which can in itself lead to bugs and therefore holes in our security.

### The MVC Authorize Attribute ###

The 'Authorize' attribute can be added to a controller action method or a controller class. If used on the latter the permissions are defined for all of the controller's action methods.

In its default form the attribute will ensure that a user is logged in. Where a user is not logged in they will be redirected to the login page.

Setting the attribute on a controller class:

```
[Authorize]
public class MyController {

}
```

Setting the attribute on an action method within a controller class:

```
public class MyOtherController {
	[Authorize]	
	public ActionResult MyActionMethod() {
	}
}
```

The attribute can take a CSV list of user roles to enforce that a user is logged in with a specific role.

```
[Authorize(Roles ="Admin")]
public ActionResult AdminyTypeThingMethod() {

}
```

The attribute can contain a CSV list of user names via the users property. However you should consider only adding permissions to user roles. Allowing permissions at a user level will only create a system which is too granular and therefore harder to test and maintain.

### ASP.NET Membership And Role Provider ###

When thinking about security you should never try and write your own. Always use tried and tested third-party frameworks.

Where user accounts are concerned, ASP.NET has the membership and role provider framework.

https://msdn.microsoft.com/en-us/library/aa354509%28v=vs.110%29.aspx

The framework self initializes the database and everything else required when it is first used.

It provides an industry proved secure framework for managing user accounts, user sessions management, defining permissions of URLs as well as the functionality of user roles. 

**Note** the database schema can only be created if it is given an admin user account within its configuration. The account should be changed to one that has the least number of required privileges permitted to provide a working system immediately afterwards.

The following tables are created:

```
-- User Accounts
select * from dbo.UserProfile

-- Roles
select * from dbo.webpagfes_Role

-- Mapped users to roles via their PK Id``
select * from dbo.webpagfes_UsersInRoles
```


Users can be added to roles via SQL or via code as shown below:

```
using System.Web.Security;
Roles.AddUserToRole("", "Admin");
```

We can the determine if a user is logged in and within a role with the IsInRole method of the User class.:

```
@if (User.IsInRole("Admin"))
{
	<li>@Html.ActionLink("Admin", "Index", "Admin")</li>
}
```