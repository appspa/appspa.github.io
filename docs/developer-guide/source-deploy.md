---
sidebar_label: "源代码"
---
# 源代碼部署


### 运行前准备

* 安装 MongoDB (3.6)
* 安装 Nodejs


1.clone 下载代码 `git clone https://github.com/appspa/app-space.git`

2.运行server端

```bash
#进入项目根目录
cd server
npm install  #安装依赖
npm start

=============>>out
...
...
App is listening on 8081.
数据库连接成功
=============>>end
```

3.编译运行前端页面

```Bash
cd client
npm install
npm run build #正式环境可以用该命令编译静态文件交给nginx
npm run start  #本地运行可以使用该命令

============>>out
...
...
 DONE  Compiled successfully in 18546ms                                                
 I  Your application is running here: http://0.0.0.0:8081
============>>end
```

4.使用浏览器打开进入http://localhost:8081即可进入应用




### 项目配置说明

**前端配置**

参见 client/config.js
```javascript
const config = {
  domain: process.env.APP_SPA_DOMAIN || "http://192.168.1.10:8081",
  // basePath: process.env.APP_SPA_PATH || "/x/app/",
};

export default config;

```

**后端配置**

参见 server/config.js

```javascript
//需要修改配置可以修改config.js文件,也可以在部署的时候导出环境变量
//比如 export APP_SPA_DOMAIN=https://127.0.0.1:8085

const common = {
    //baseUrl应用请求的url地址,比如https://appspace
    baseUrl: process.env.APP_SPA_DOMAIN || "https://127.0.0.1:8081", 
    port: process.env.APP_SPA_PORT || "8085", //server运行的端口
    apiPrefix: 'api',
    secret: process.env.APP_SPA_SECRET || "secretsecret", //secret
    //数据库用户 (没有开启mongodb用户认证的可以不填写)
    dbUser: process.env.APP_SPA_DBUSER || undefined,  
    //数据库密码 (没有开启mongodb用户认证的可以不填写)
    dbPass: process.env.APP_SPA_DBPWD || undefined,  
    dbName: process.env.APP_SPA_DB_NAME || "app_space", //数据库名称
    dbHost: process.env.APP_SPA_DB_HOST || "localhost", //数据库地址
    dbPort: process.env.APP_SPA_DB_PORT || "27017", //数据库端口
	
    //邮件相关配置 用于找回密码和邀请团队成员发送邮件
    emailService: process.env.APP_SPA_EMAIL_SERVICE || "qq", 
    emailUser: process.env.APP_SPA_EMAIL_USER || "", 
    emailPass: process.env.APP_SPA_EMAIL_PASS || "",

    //是否允许用户注册,为否则后端注册接口不可用
    allowRegister: process.env.APP_SPA_ALLOW_REGISTER || true, 

    //是否开启ldap 默认是false 如果公司没有ldap服务可以不用理会
    openLdap: process.env.APP_SPA_ALLOW_LDAP || false, 
    ldapServer: process.env.APP_SPA_LDAP_URL || "",  //ldap server url
    ldapUserDn: process.env.APP_SPA_LDAP_USERDN || "", //ldap管理员dn 管理员用户名
    ldapBindCredentials: process.env.APP_SPA_LDAP_CREDENTIALS || "", //ldap管理员密码
    ldapBase: process.env.APP_SPA_LDAP_BASE || "" //ldap base

};
```





### 正式环境部署 nginx配置(注意请使用https部署,否则iOS会出现无法安装的问题)

可以按照项目根目录的 app_space_nginx.conf 文件进行配置

```bash
server{
  listen 80;
  server_name appspace;

  #root目录为项目根目录的client/dist目录下,前端静态页面
  root /home/ubuntu/appspace/client/dist;
  index index.html;

  location / {
      try_files $uri $uri/ @router;
      index index.html;
  }

  location /api/ {  #把以api打头的接口转发给后端server
    proxy_pass http://127.0.0.1:8085; #这里端口修改为后端服务运行的端口
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
  client_max_body_size 208M; #最大上传的ipa/apk文件大小
}
```


