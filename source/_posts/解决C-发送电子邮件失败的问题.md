---
title: '解决C#发送电子邮件失败的问题'
date: 2017-10-18 08:21:15
categories: 技术相关
tags: [C#]
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/mail.png" width="20%" height="20%">

本文记录了在 C# 环境下，如何利用 SMTP 邮箱服务发送邮件，以 QQ 邮箱为例
<!-- more -->

# 实现过程

- 配置 App.Config 文件
    - 在项目中添加`System.Configuration`程序集的引用
    - 在 App.Config 文件中添加键值，如下（只需要关注 appSettings 标签内的内容）

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <configSections>
    <section name="entityFramework" type="System.Data.Entity.Internal.ConfigFile.EntityFrameworkSection, EntityFramework, Version=6.0.0.0, Culture=neutral, PublicKeyToken=*************" requirePermission="false" />
  </configSections>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
  </startup>
  <appSettings>
  <!--邮箱-->
    <add key="MANAGER_MAIL_NUM" value="123456789@qq.com" />
    <!--邮箱密码-->
    <add key="MANAGER_MAIL_PASSWORD" value="邮箱密码。QQ邮箱需要许可码" />
    <!--邮件显示名-->
    <add key="MANAGER_MAIL_NAME" value="发送邮件使用的用户名" />
    <!--QQ邮箱对应的SMTP服务器-->
    <add key="SmtpClient_HOST" value="smtp.qq.com"/>
  </appSettings>
</configuration>
```

- EmailHelper.cs 文件内容

```C#
using System;
using System.Text;
using System.Net.Mail;
using static System.Configuration.ConfigurationManager;

namespace View_Spot_of_City.UIControls.Helper
{
    public static class EmailHelper
    {
        public static bool SendEmail(string mail, string title, string content)
        {
            MailMessage message = new MailMessage();
            {
                message.To.Add(mail);
                message.From = new MailAddress(AppSettings["MANAGER_MAIL_NUM"], AppSettings["MANAGER_MAIL_NAME"], Encoding.UTF8);
                message.Subject =title;
                message.SubjectEncoding = Encoding.UTF8;
                message.Body = content;
                message.BodyEncoding = Encoding.UTF8;
                message.IsBodyHtml = false;
                message.Priority = MailPriority.Normal;
            }

            SmtpClient smtp = new SmtpClient();
            {
                smtp.Host = AppSettings["SmtpClient_HOST"];
                smtp.EnableSsl = true;
                smtp.UseDefaultCredentials = false;
                smtp.Credentials = new System.Net.NetworkCredential(AppSettings["MANAGER_MAIL_NUM"], AppSettings["MANAGER_MAIL_PASSWORD"]);
            }
            object userState = message;
            try
            {
                smtp.SendAsync(message, userState);
                return true;
            }
            catch(Exception ex)
            {
                Console.Write(ex.Message);
                return false;
            }
        }
    }
}
```

# 注意

若使用 QQ 邮箱，输入密码为许可码，需要在 QQ 邮箱中打开 SMTP 服务

- 打开 QQ 邮箱网页版
- 点击左上角"设置"，并在导航栏中点击账户标签
- 开启 SMTP 服务并获得许可码
    ![开启 SMTP 服务](http://home.cs-tao.cc/github-content/contents/blog/image/WpfSendMail.png)
