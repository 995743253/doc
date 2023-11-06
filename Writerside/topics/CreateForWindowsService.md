# WebApi注册为Windows服务

## 一.创建WebApi
创建asp.net core项目,模板选webapi,框架选.net6以上

## 二.配置
打开Program.cs
### 配置Kestrel
*显式的配置使用asp.net core的内置服务器Kestrel*
```C#
builder.WebHost.ConfigureKestrel(serverOptions =>
{
serverOptions.Limits.MaxConcurrentConnections = 100;
});
```
Options可以不配置,仅使用默认设置. <br/>
*监听Url* <br/>
Kestrel无法反向代理,所以需要显式配置监听所有Url,端口可改
```C#
builder.WebHost.UseUrls("http://*:28888");
```
*禁用Https(选用)*<br/>
没有证书,可以不使用这个中间件
```C#
// app.UseHttpsRedirection();
```
### 配置Windows服务
*nuget添加依赖 Microsoft.Extensions.Hosting.WindowsServices*<br/>
这个库目前仅支持`.net6` 或者`.NETFramework 4.6.2`以上的框架

*注册*
```C#
#region 配置webapi可打包为Windows服务

var options = new WebApplicationOptions
{
    Args = args,
    ContentRootPath = WindowsServiceHelpers.IsWindowsService() 
        ? AppContext.BaseDirectory : default
};

#endregion
var builder = WebApplication.CreateBuilder(options);
//注册Windows服务
builder.Host.UseWindowsService();
```
## 三.发布生成自包含应用
发布到文件夹,发布配置文件中,按如下配置
1. 部署模式 <br/>
选择独立部署,框架依赖需要本地有.net runtime
2. 目标运行时 <br/>
根据客户电脑判断,win7最好选择win-x86,win10可以选择win-x64

## 四.安装api windows服务
1. 使用SC工具注册 <br/>
```
sc create "Digihua.PrintService" binPath= "{安装路径}\win-x86\publish\WebApi.exe" start=auto
```
2. 使用bat脚本注册 <br/>
安装
``` Shell
@echo off
>nul 2>&1 "%SYSTEMROOT%\system32\cacls.exe" "%SYSTEMROOT%\system32\config\system"
if '%errorlevel%' NEQ '0' (
goto UACPrompt
) else ( goto gotAdmin )
:UACPrompt
echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
"%temp%\getadmin.vbs"
exit /B
:gotAdmin
if exist "%temp%\getadmin.vbs" ( del "%temp%\getadmin.vbs" )
@echo.服务启动......  
@echo off  
set "current_dir=%~dp0"
@sc create DigihuaWeightApi binPath= "%current_dir%\win-x86\publish\WeightApi.exe"  
@net start DigihuaWeightApi  
@sc config DigihuaWeightApi start= AUTO  
@echo off  
@echo.启动完毕！  
@pause
```
卸载
```Shell
@echo off
>nul 2>&1 "%SYSTEMROOT%\system32\cacls.exe" "%SYSTEMROOT%\system32\config\system"
if '%errorlevel%' NEQ '0' (
goto UACPrompt
) else ( goto gotAdmin )
:UACPrompt
echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
echo UAC.ShellExecute "%~s0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
"%temp%\getadmin.vbs"
exit /B
:gotAdmin
if exist "%temp%\getadmin.vbs" ( del "%temp%\getadmin.vbs" )
@echo.服务删除  
@echo off  
@net stop DigihuaWeightApi
@sc delete DigihuaWeightApi 
@echo off  
@echo.删除结束！  
@pause****
```