# 配置NLog

## 配置文件
在项目目录下新增一个NLog.config<br/>
按照公司Nlog配置修改,内容如下:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.nlog-project.org/schemas/NLog.xsd NLog.xsd"
      autoReload="true"
      throwExceptions="false"
      internalLogLevel="Off" internalLogFile="c:\temp\nlog-internal.log">

  <!-- optional, add some variables
  https://github.com/nlog/NLog/wiki/Configuration-file#variables
  -->
  <variable name="myvar" value="myvalue"/>

  <!--
  See https://github.com/nlog/nlog/wiki/Configuration-file
  for information on customizing logging rules and outputs.
   -->
  <!--[變數] 文字樣板 -->
  <!--<variable name="Layout" value="${longdate} | ${level:uppercase=true} | ${message}"/>-->
  <variable name="Layout" value="${longdate}|${level:uppercase=true}[${threadid}]${message}|${exception:format=ToString, StackTrace}"/>

  <!--[變數] 檔案位置 -->
  <variable name="LogTxtLocation" value="${basedir}/Logs/${level:uppercase=true}_${date:format=yyyy-MM-dd-HH}"/>
  <variable name="ScheduleLogTxtLocation" value="${basedir}/Logs/Schedule/${level:uppercase=true}_${date:format=yyyy-MM-dd-HH}"/>
  <variable name="MESwsLogTxtLocation" value="${basedir}/Logs/MESws/${level:uppercase=true}_${date:format=yyyy-MM-dd-HH}"/>
  <targets>

    <!--
    add your targets here
    See https://github.com/nlog/NLog/wiki/Targets for possible targets.
    See https://github.com/nlog/NLog/wiki/Layout-Renderers for the possible layout renderers.
    -->

    <!-- Write events to a file with the date in the filename. 寫入目標 -->
    <target name="File" xsi:type="File" fileName="${LogTxtLocation}.log" layout="${Layout}"
            archiveFileName="${LogTxtLocation}.{#####}.log"
            archiveAboveSize="10240000"
            archiveNumbering="Sequence" />
    <target name="ScheduleFile" xsi:type="File" fileName="${ScheduleLogTxtLocation}.log" layout="${Layout}"
            archiveFileName="${ScheduleLogTxtLocation}.{#####}.log"
            archiveAboveSize="10240000"
            archiveNumbering="Sequence" />
    <target name="MESwsFile" xsi:type="File" fileName="${MESwsLogTxtLocation}.log" layout="${Layout}"
            archiveFileName="${MESwsLogTxtLocation}.{#####}.log"
            archiveAboveSize="10240000"
            archiveNumbering="Sequence" />
  </targets>
  
  <rules>
    <!-- add your logging rules here -->

    <!--
    Write all events with minimal level of Debug (So Debug, Info, Warn, Error and Fatal, but not Trace)  to "f"
    <logger name="*" minlevel="Debug" writeTo="f" />
    -->
    <logger name="Microsoft.*" maxlevel="Info" final="true" />
    <logger name="*" levelmin="Trace" writeTo="File"/>
  </rules>
</nlog>
```
## 项目配置
打开Program.cs
*nuget添加依赖 `NLog.Web.AspNetCore`*<br/>
*注册*
```C#
//Nlog注册
builder.Logging.ClearProviders();
builder.Host.UseNLog();
```

## 在项目中使用
*依赖注入进Controller*
```C#
private readonly ILogger<T> _logger;
public Controller(ILogger<T> logger)
{
        _logger = logger;
}
```
*日志记录的函数*<br/>
可以使用插值表达式,也可以直接记录
```C#
 var log = "123123123";
_logger.LogTrace("{log}",log);
_logger.LogInformation("{log}",log);
_logger.LogDebug("{log}",log);
_logger.LogWarning("{log}",log);
_logger.LogError("{log}",log);
```