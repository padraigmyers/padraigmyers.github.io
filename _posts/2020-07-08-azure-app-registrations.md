---
layout: post
title: eili5 Azure App Registrations
subtitle: Authenticating against azure without passwords
gh-repo: padraigmyers/padraigmyers.github.io
gh-badge: [star, fork, follow]
tags: [csharp][azure][auth]
comments: true
---

When you want to access something in Azure you can just log on with your username and password, and that authenticates you.

When you have an application that needs to do the same thing, you can use your username and password again, but this leave you with the problem of where to store it securely. If you store it in config then it can easily be leaked, and is probably in plain text, you can encrypt it, but again this leave the problem of who do you handle sharing the encryption keys.

The solution in Azure is to use [App Registrations](https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app).  
Think of an App Registration as an application's version of a **_user_**.  
The app reach's out to Azure and _proves_ (details below) that is it that application, it then gets a token (similar to an OAuth token) that allow the app to call out to other services in azure that it has access to (e.g. Key Vault, App Configuration, SQL Database etc).

The app has a number of ways to prove it is that app, one of the most useful way is via a SSL certificate. An SSL certificate can be created, put into the certificate store on the machine where the application runs, and also uploaded to the **_Certificates & Secrets_** section of the App Registration in Azure.  

When the app wants to get at some service on azure it calls out the app service with the tenantId and clientId of the app registration (to identify the app registration in azure) and also passes the certificate, to prove that it is allowed to use that app registration.  
It then gets a token and can call the app in question.  

Most Azure SDK's have helper methods that do all this for you. You just create a ClientSecretCredentials object using the tenantId, clientId and secret and pass that though when calling the certificate.
However, below is an example of calling it manually to get back a token for a SQL Server connection 

```cs
const string clientId = "?????";
const string tenantId = "?????";
const string thumbprint = "????";
const string certificateStore = "CurrentUser";

var authConnectionString = $"RunAs=App;AppId={clientId};TenantId={tenantId};CertificateStoreLocation={certificateStore};CertificateThumbprint={thumbprint}";

return new SqlConnection("Server=tcp:???.database.windows.net,1433;Database=???")
{
    AccessToken = new AzureServiceTokenProvider(authConnectionString)
        .GetAccessTokenAsync("https://database.windows.net/")
        .GetAwaiter()
        .GetResult()
};
```
