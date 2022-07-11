---
title: sourcetree破解，跳过atlassian账号登陆sourcetree
index_img: /img/cover/05.jpg
categories:
  - 工具类
tags:
  - sourcetree
abbrlink: 563089cb
date: 2018-09-21 17:52:22
---

1. **找到本地账户sourceTree目录**
    ![](1.png)
2. **找到对应的accounts.json文件，如果没有新建一个.**
3. **编辑accounts.json  内容为：**
    ```json
    [
      {
        "$id": "1",
        "$type": "SourceTree.Api.Host.Identity.Model.IdentityAccount, SourceTree.Api.Host.Identity",
        "Authenticate": true,
        "HostInstance": {
          "$id": "2",
          "$type": "SourceTree.Host.Atlassianaccount.AtlassianAccountInstance, SourceTree.Host.AtlassianAccount",
          "Host": {
            "$id": "3",
            "$type": "SourceTree.Host.Atlassianaccount.AtlassianAccountHost, SourceTree.Host.AtlassianAccount",
            "Id": "atlassian account"
          },
          "BaseUrl": "https://id.atlassian.com/"
        },
        "Credentials": {
          "$id": "4",
          "$type": "SourceTree.Model.BasicAuthCredentials, SourceTree.Api.Account",
          "Username": "",
          "Email": null
        },
        "IsDefault": false
      }
    ]
    ```