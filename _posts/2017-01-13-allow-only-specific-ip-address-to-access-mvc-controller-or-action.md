---
layout: post
published: true
title: Allow only specific ip address to access MVC Controller or Action
date: '2017-12-02'
---
When you are developing a back-end web application, you might want to restrict access only to people who are working in the company which is maintaining the content or any other operations in the back-end and allowing public access only for the public, usually read-only content.

This means you still have to leave some parts of your application public and for some you need to restrict access for the people outside of a network (certain IP range).

MVC makes it really easy to achieve this. As you know, any controller or action inside a controller can have attributes which define behavior of it. So basically, all you have to do is to write your own attribute and change behavior of controller or action inside it.

In the following code is a simple example how to make your own custom attribute.

To make allowed IP addresses more manageable, we are going to store them in web.config file as comma separated values as following

```xml
<appSettings>  
  <add value="127.0.0.1,192.168.0.1,192.168.0.104" key="AllowedIPAddresses" />  
</appSettings>
```

And now to do the magic with custom attribute. In case that IP address of the client machine which initiated request is not valid, 403 status code (Forbidden) will be returned back to client in response.

```csharp
namespace AddressAccessFiltering  
{
    public class AuthorizeIPAddressAttribute : ActionFilterAttribute  
    {  
        public override void OnActionExecuting(ActionExecutingContext context)  
        {  
            string ipAddress = HttpContext.Current.Request.UserHostAddress;  
  
            if (!IsIpAddressAllowed(ipAddress.Trim()))  
            {  
                context.Result = new HttpStatusCodeResult(403);  
            }  
  
            base.OnActionExecuting(context);  
        }  
  
        private bool IsIpAddressAllowed(string IpAddress)  
        {  
            if (!string.IsNullOrWhiteSpace(IpAddress))  
            {  
                string[] addresses = Convert.ToString(ConfigurationManager.AppSettings["AllowedIPAddresses"]).Split(',');  
                return addresses.Where(a => a.Trim().Equals(IpAddress, StringComparison.InvariantCultureIgnoreCase)).Any();  
            }  
            return false;  
        }  
    }  
}  
```

```
### Note
This simple example only check the list of IP addresses listed in config file. If you want check IP range you will have to extend the logic of IP address validation. 
```

Now, to apply this validation you just need to add attribute to a controller in case you want to check request to all actions in a controller

```csharp
[AuthorizeIPAddress]  
public class AddressFilteredController : ApiController
{  
}  
```

In case you want to secure only one action, apply attribute to only that specific action

```csharp
public class PubllicController : ApiController
    {
        [AuthorizeIPAddress]
        public JsonResult GetAccountDetails(int Id)
        {
            throw new NotImplementedException("IP Address is authorized for method GetAccountDetails");
        }
    }
```

This becomes really handy in case you have Web API which is public, but you want to expose only certain methods to public users and allow only specific for "in house" user which come from specific network.
