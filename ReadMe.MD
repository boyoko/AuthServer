### 创建Authorization Server


1. 建立asp.net core 项目使用空模板
---------------------------------

2. 打开nuget, 搜索 Identityserver4, 引用到项目中
-----------------------------------
3. 打开startup.cs, 编辑Configure方法 ,添加代码：
-----------------------------------------
 > ```
 >	app.UseIdentityServer();
 > ```
4. Startup.cs,编辑ConfigureServices方法, 这里不仅要把IdentityServer注册到容器中, 还要至少对其配置三点内容:
---------------------------------------------------
 - 哪些API可以使用这个authorization server.

 - 那些客户端Client(应用)可以使用这个authorization server.

 - 指定可以使用authorization server授权的用户.

首先需要把上面这些做成一个配置文件,建立Configuration/InMemoryConfiguration.cs:
```
namespace AuthServer.Configuration
{
    public class InMemoryConfiguration
    {
        public static IEnumerable<ApiResource> ApiResources()
        {
            return new[]
            {
                new ApiResource("Album", "相册")
            };
        }

        public static IEnumerable<Client> Clients()
        {
            return new[]
            {
                new Client
                {
                    ClientId = "Album",
                    ClientSecrets = new [] { new Secret("secret".Sha256()) },
                    AllowedGrantTypes = GrantTypes.ResourceOwnerPasswordAndClientCredentials,
                    AllowedScopes = new [] { "Album" }
                }
            };
        }

        public static IEnumerable<TestUser> Users()
        {
            return new[]
            {
                new TestUser
                {
                    SubjectId = "1",
                    Username = "asdf",
                    Password = "123456"
                }
            };
        }
    }
}
```

**ApiResources: 这里指定了name和display name, 以后api使用authorization server的时候, 这个name一定要一致, 否则就不好用的.**

**Clients: Client的属性太多了, 这里就指定几个. 其中ClientSecrets是Client用来获取token用的. AllowedGrantType: 这里使用的是通过用户名密码和ClientCredentials来换取token的方式. ClientCredentials允许Client只使用ClientSecrets来获取token. 这比较适合那种没有用户参与的api动作. AllowedScopes: 这里只有Album**

**Users: 这里的内存用户的类型是TestUser, 只适合学习和测试使用, 实际生产环境中还是需要使用数据库来存储用户信息的, 例如接下来会使用asp.net core identity. TestUser的SubjectId是唯一标识.**


5. 回到StartUp的ConfigureServices, 添加以下代码：

```
services.AddIdentityServer()
                .AddDeveloperSigningCredential()
                .AddTestUsers(InMemoryConfiguration.Users().ToList())
                .AddInMemoryClients(InMemoryConfiguration.Clients())
                .AddInMemoryApiResources(InMemoryConfiguration.ApiResources());
``` 
6. 启动服务，使用PostMan 验证：

|   请求方式  |   调用地址                                 | 参数          |  值
|-----       | ------------------------------------------|---------------|-----
|POST       | http://localhost:5000/connect/token       | client_id     | Album
|          |                                           | client_secret | secret
|           |                                           | grant_type    | client_credentials


**grant_type如果使用password , 需要添加 username 和 password 这两个参数，使用正确的用户名和密码来获取token**
  
   参数  |   值
-------- | ----------
username  |  asdf
password  |  123456

7. 添加UI
 - 在项目根目录，打开Powershell，输入命令,就把UI下载到项目了.
	```
	iex ((New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/IdentityServer/IdentityServer4.Quickstart.UI/release/get.ps1'))
	```
 - 由于有wwwroot下很多静态文件, 所以asp.net core 需要启用服务静态文件的功能: 修改Startup的Configure方法:
	```
		app.UseStaticFiles();
		app.UseMvcWithDefaultRoute();
	```
 - 在ConfigureServices里面注册MVC:
	```
	services.AddMvc();
	```
8. 运行程序，并使用设定的用户进行登录

                                                        
