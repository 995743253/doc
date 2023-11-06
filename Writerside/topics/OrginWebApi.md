# 跨域配置

前面的配置可以使用服务端正常调用WebApi,但是在前端调用会出现跨域问题

## 项目配置允许跨域
打开Program.cs <br/>
*注册跨域规则*
```C#
//跨域,允许所有源
builder.Services.AddCors(corsOptions =>
{
    corsOptions.AddPolicy("cors", p => p
        .AllowAnyMethod()
            .SetIsOriginAllowed(x=>true)
            .AllowAnyHeader()
            .AllowCredentials()
    );
});
```
*使用定义的跨域规则*
```C#
app.UseCors("cors");
```
重新部署后,前端可正常发送http请求,不会被浏览器的同源策略所限制

## 前端调用
```Javascript
 $http.get('http://127.0.0.1:28888/api/XXX', {params: {}})
               .success((data) => {})
               .error((data) => {});
 $http.post('http://127.0.0.1:28888/api/XXX', {})
                .success((data) => {})
               .error((data) => {});
 $http.delete('http://127.0.0.1:28888/api/XXX', {})
                .success((data) => {})
               .error((data) => {});      
 ....
```