---
layout: post
title: HangFire dashboard in azure
subtitle: Each post also has a subtitle
gh-repo: padraigmyers/padraigmyers.github.io
gh-badge: [star, fork, follow]
tags: [csharp,azure,hangfire]
comments: true
---


Create an identity for the azure app

```
az webapp identity assign --resource-group hangfire-test --name hangfire-ui
```

Remove the existing connection string that would have been created by default when the app was deployed
```
az webapp config connection-string delete --resource-group hangfire-test --name hangfire-ui --setting-names MyDbConnection
```

Create a firewall rule for the app's ip address to allow it to access the db (easiest way to find this out is to just run that app, it will complain about this firewall rule not being in place when it runs)  
_Note: the start and the end ip address will be the same ip address, as you are only allowing your app service to access the db_
```
az sql server firewall-rule create --resource-group hangfire-test --name myip --server hangfire-sql2 --start-ip-address ??? --end-ip-address ???
```  

On the db run the following commands to allow the app to be able to access and modify the hangfire db
```sql
create user [hangfire-ui-local] from external provider;
alter role db_datareader add member [hangfire-ui-local];
alter role db_datawriter add member [hangfire-ui-local];
alter role db_ddladmin add member [hangfire-ui-local];
```