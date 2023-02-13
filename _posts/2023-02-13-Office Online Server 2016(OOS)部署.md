---
layout:     post
title:      Office Online Server 2016(OOS)部署
subtitle:   OOS部署
date:       2023-01-11
author:     W
header-img: img/post-bg-oos.png
catalog: true
tags:
    - Office
    - OOS
    - 在线编辑
    - 在线预览
---

## 前言

Office Online Server (OOS，下文简写为OOS) 提供基于浏览器的 Word、PowerPoint、Excel在线预览和编辑。

OOS可完美支持复杂的 PPT 与 Excel。同时能保留 office 文档的动态效果或视频。总之，仅除了少数功能缺失外，桌面版拥有的功能和网页版并无区别，且具有自动实时保存与多人协作功能；当你修改了 office 文档时，OOS 会立即自动保存。

## 正文

### 先决条件

- 需要两台服务器，一台作为域控服务器、一台作为OOS转换服务器
- OOS转换服务器必须使用64 位版本的 Windows Server 2012 R2 或Windows Server 2016（仅适用与 Office Online Server 11 月2018或更高版本）

> 建议
>
> OOS转换服务器配置至少6G内存及以上，另外转码会生成大量缓存，因此建议磁盘50G及以上。
>
> 当OOS服务器配置较高时，打开文档将十分快速流畅；配置较低打开则及其缓慢，且有失败的可能性（内存或磁盘不足，无法完成转码工作）。
>
> 推荐使用Windows Server 2016系统部署OOS， 本文也基于Windows Server 2016。

### 注意事项

- 请勿在运行 OOS 的服务器上安装任何其他服务器应用程序。如服务器不足，请在虚拟机上运行OOS。
- 请勿在端口 80（web前端程序 ²）、443 或 809 （转码程序 ³）上安装依赖 Web 服务器 (IIS) 角色的任何服务或角色。否则 OOS 会定期删除这些端口上的 Web 应用程序。
- 请勿安装任何版本的桌面 Office。如果已经安装，在安装 OOS 之前必须将其完全卸载。
- 请勿在域控制器上安装 OOS。OOS 不会在包含 Active Directory 域服务 (AD DS) 的服务器上运行。
- OOS 服务器必须加入 AD 域。

### 安装资源

- .NET Framework 4.5.2
- Visual C++ Redistributable Packages for Visual Studio 2013
- Visual C++ Redistributable for Visual Studio 2015
- Microsoft.IdentityModel.Extention.dll
- Office Online Server 2017.3 更新版本

注：若需要相关资源可联系本人获取

### 搭建域控服务器

1. 打开“服务器管理”，点击“添加角色和功能”
2. 默认点击“下一步” 至 “添加AD域服务器”
3. 添加AD域服务器，点击“添加功能”，并点击“下一步
4. 安装 .NET 3.5，点击下一步，并“安装”
5. 点击“此服务器提升为域控制器”
6. 添加新林（填写你自定义的域名，示例：exmple.com）
7. 下一步，输入密码
8. 其他步骤全部保持默认安装。

域控服务器至此既配置完成。

### 搭建OOS转换服务器

1. 固定IP，修改DNS为域控服务器IP

2. 更改域

   ```
   在计算机右键 => 属性中点击更改设置。随后输入上一步 “添加新林” 中你设定的根域名，在这里示例为：exmple.com
   输入域服务器的账号和密码，点击“确定”，随后重启此服务器
   重启后，打开 Win+R，输入“gpedit.msc”
   找到：计算机配置=>管理模板=>系统=>指定可选组件安装和组件修复的设置，启用它
   打开“PowerShell”，输入“gpupdate / force”，刷新组策略
   ```

3. 安装角色与服务

   ```
   以管理员身份在“PowerShell”里输入：
   Add-WindowsFeature Web-Server,Web-Mgmt-Tools,Web-Mgmt-Console,Web-WebServer,Web-Common-Http,Web-Default-Doc,Web-Static-Content,Web-Performance,Web-Stat-Compression,Web-Dyn-Compression,Web-Security,Web-Filtering,Web-Windows-Auth,Web-App-Dev,Web-Net-Ext45,Web-Asp-Net45,Web-ISAPI-Ext,Web-ISAPI-Filter,Web-Includes,NET-Framework-Features,NET-Framework-45-Features,NET-Framework-Core,NET-Framework-45-Core,NET-HTTP-Activation,NET-Non-HTTP-Activ,NET-WCF-HTTP-Activation45,Windows-Identity-Foundation,Server-Media-Foundation
   ```

4. 安装软件

   ```
   按照下列步骤在服务器上安装全部软件：
   安装NET Framework 4.5.2（NDP452-KB2901954-Web.exe）；
   安装Visual C++ Redistributable Packages for Visual Studio 2013（vcredist_x64.exe）；
   安装Visual C++ Redistributable for Visual Studio 2015（vc_redist.x64.exe）；
   安装Microsoft.IdentityModel.Extention.dll（MicrosoftIdentityExtensions-64.msi）；
   安装Office Online Server 2016（setup.exe）打开镜像运行“setup.exe”默认同意选项以安装；
   安装语言包（cn_office_online_server_language_pack_may_2016_x64_8783021.exe）
   ```

5. 启动服务场

   ```
   以管理员身份在“PowerShell”里输入：
   New-OfficeWebAppsFarm -InternalURL "http://docs.exmple.com" -AllowHttp -EditingEnabled
   # 更改为你的域名。
   ```

6. 验证服务是否可用

   ```
   访问 docs.exmple.com/hosting 显示一个XML文档，即为配置成功。
   访问 docs.exmple.com/op/generate.aspx 即可使用。
   ```

   注意：若访问出现服务器错需要开启配置`Set-OfficeWebAppsFarm -OpenFromUrlEnabled:$true`

