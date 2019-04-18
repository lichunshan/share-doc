# 1. pm2自动部署前端项目
## 1.1. 将本地和远端服务器公钥添加git服务器
整个自动部署流程都需要免密
## 1.2. 配置ssh免密登录服务器
### 1.2.1. 本机生成公钥
`ssh-keygen -t rsa -C "xxxxx@xxxxx.com"`

![](_v_images/_1555555352_9571.png)

>说明：出现输入密码提示时，直接回车默认不设置密码。

### 1.2.2. 将公钥存储到远程主机
`ssh-copy-id -i path/to/my/key your_username@server.com`

例如：`ssh-copy-id -i .ssh/id_rsa.pub git@12.56.224.61`
>说明：    
-i 后跟的是本地公钥路径  
该命令会将公钥存储到远程主机`.ssh/authorized_keys`文件中  
也可以手动添加 public key 到服务器上的 `~/.ssh/authorized_keys`。  
>>ssh 对目录的权限有要求，代码中要设置下新生成的config文件权限才行。～目录权限是750，~/.ssh 的是700， ~/.ssh/* 的是600，~/.ssh/config 是700  
`$chmod 600 .ssh/authorized_keys`

### 1.2.3. 开启远程主机上的ssh公钥认证登录功能
检查ssh服务的配置文件——`/etc/ssh/sshd_config`
```
RSAAuthentication yes    # 这行一定要取消注释的（删掉#号）
PubkeyAuthentication yes    # 我的服务器没这行，不添加似乎也能用
AuthorizedKeysFile .ssh/authorized_keys    # 我的服务器没这行，不添加似乎也能用
```
重启ssh服务

`systemctl restart sshd`
## 1.3. 配置pm2自动部署
### 1.3.1. 在项目根目录下新建`ecosystem.json`文件

```
{
  "apps" : [
  {
    "name": "m_chasing",
    "script": "app.js"
  }
  ],
    "deploy": {
      "production": {
        "user": "root",
        "host": ["144.34.159.83"],
        "port": "28389",
        "ref": "origin/master",
        "repo": "git@git.chasing-innovation.com:3000:lichunshan/m_chasing.git",
        "path": "/WWW/production",
        "post-deploy": "npm install && pm2 startOrRestart ecosystem.json --env production"
      },
      "test": {
        //ssh用户
        "user": "root",
        //服务器ip
        "host": ["67.230.191.105"],
        //ssh端口
        "port": "28391",
        //git远程分支
        "ref": "origin/master",
        //git仓库
        "repo": "git@github.com:lichunshan/m_chasing.git",
        //服务器部署路径
        "path": "~/te",
        //安装后置任务
        "post-setup": "yarn install;npm run pm2:install",
        //部署后置任务
        "post-deploy": "npm run pm2:update"
      }
    }
  }
  
```
>说明：  
由于 pm2 是用来部署 node 代码的，需要提供一个 js 文件用来执行，上面配置文件制定了项目根目录下的 `app.js`  
`app.js`里面写一行`console.log('app is running');`即可。  
>>`package.json`中包含的启动脚本  
```
"scripts": {
    "dev": "cross-env BUILD_ENV=dev nuxt",
    "build": "nuxt build",
    "start_test": "cross-env BUILD_ENV=dev nuxt start",
    "start_product": "cross-env BUILD_ENV=product nuxt start",
    "generate": "nuxt generate",
    "lint": "eslint --ext .js,.vue --ignore-path .gitignore .",
    "precommit": "npm run lint",
    "pm2:install": "npm run build && pm2 start npm --name 'chasing_test' -- run start_test",
    "pm2:update": "npm run build && pm2 reload all",
    "test:unit": "jest"
  }
```

### 1.3.2. 首次部署

`$ pm2 deploy test setup`

上面命令setup完成后会执行`yarn install;npm run pm2:install`

首次部署后 pm2 会在执行文件夹（配置文件中的path) 生成三个文件夹

* current - 当前版本代码，可以配置为 nginx 指向，也是 git repo
* shared - 里面有log 和pid 等信息
* source - git 拉下来的代码

### 1.3.3. 持续集成
`$ pm2 deploy test exec "git pull"`

`$ pm2 deploy test update`

### 1.3.4. 一些部署命令
```
首次部署命令
pm2 deploy test setup

非首次部署命令
pm2 deploy test update

回退一个版本
pm2 deploy test revert 1

远程执行服务器命令
pm2 deploy test exec "pm2 reload all"
```

