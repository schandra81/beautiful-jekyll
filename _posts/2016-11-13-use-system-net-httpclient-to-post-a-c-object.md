---
layout: post
published: true
title: 'Use System.Net.HttpClient to post a C# Object'
subtitle: 'Use System.Net.HttpClient to post a C# Object'
---
You can use `PostAsJsonAsync` function. But if it is not available, then you might want to obtain json serialized string from object first (in my case it is `argsAsJson`)
```cs
HttpClient client = new HttpClient();
HttpContent contentPost = new StringContent(argsAsJson, Encoding.UTF8, 
"application/json");

await client.PostAsync(new Uri(wsUrl), contentPost).ContinueWith(
(postTask) => postTask.Result.EnsureSuccessStatusCode());
```
