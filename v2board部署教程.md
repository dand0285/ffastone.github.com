## 环境要求

这里我们推荐使用aaPanel作为环境搭建的入门首选，机器的内存最好是1G，512M是最低要求配置。

本文教程是将 **aaPanel** 作为环境进行配置，部署机器由 [Moack](https://www.moack.co.kr/dedicated.php) 提供。



## 部署

### 1.配置aaPanel



```shell
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

安装完成后我们登陆 aaPanel 进行环境的安装。

选择使用LNMP的环境安装方式勾选如下信息

☑️ Nginx 1.17

☑️ MySQL 5.6

☑️ PHP 7.3

以上环境版本号均为最低要求，选择 Fast 快速编译后进行安装。

### 2.安装Redis

aaPanel 面板 > App Store > 找到PHP 7.3点击Setting > Install extentions > redis 进行安装。

### 3.解除被禁止的函数

aaPanel 面板 > App Store > 找到PHP 7.3点击Setting > Disabled functions 将 `putenv` `proc_open` `pcntl_alarm ` `pcntl_signal`从列表中删除。

### 4.添加站点

aaPanel 面板 > Website > Add site。

```
Domain：填入你指向服务器的域名
Database：MySQL
PHP version：PHP-73
```

Submit 完成创建。

完成创建后访问站点目录删除目录下的所有文件，.user.ini 文件需要在 btPanel 下单独点击 Del 删除。

添加完成后编辑添加的站点 > Site directory > Running directory 选择 /public 保存。

添加完成后编辑添加的站点 > URL rewrite 填入伪静态信息。

```nginx
location /downloads {
}

location / {  
	try_files $uri $uri/ /index.php$is_args$query_string;  
}

location ~ .*\.(js|css)?$
{
    expires      1h;
    error_log off;
    access_log /dev/null; 
}
```

到此站点添加完成。

### 5.安装V2Board

通过SSH登录到服务器后访问站点路径如：/www/wwwroot/js.xxx.me。

以下命令都需要在站点目录进行执行。

执行命令从 Github 克隆到当前目录。

```shell
# 更新网站到1.02版本并更新数据库
git fetch --all && git reset --hard origin/master && git pull origin 1.0.3 && php artisan v2board:update

# 1.0.1 为当前 V2Board 版本号
git clone -b 1.0.1 https://github.com/v2board/v2board.git ./
```

执行命令下载 composer.phar 到当前目录。

```shell
wget https://getcomposer.org/download/1.9.0/composer.phar
```

执行命令进行包安装。

```shell
php composer.phar install
```

安装过程中报错或者无法继续安装的请分配 swap，如何分配 swap 请查阅 google。

复制.env.example文件为.env。

```shell
# domain.com 请更改为站点域名且路径必须存在
cp .env.example .env
```

打开 .env 文件，修改数据库信息并保存。

```shell
DB_HOST=数据库地址
DB_PORT=3306
DB_DATABASE=数据库名
DB_USERNAME=数据库用户名
DB_PASSWORD=数据库密码
```

保存后请重新给予目录权限

```shell
# domain.com 请更改为站点域名且路径必须存在
chown -R www ../js.xxx.me
```

执行命令进行面板的安装。

```shell
php artisan v2board:install
```

输入管理员账号密码，至此一切就绪，可以访问你的面板了。

### 6.配置定时任务

aaPanel 面板 > Cron。

```shell
# domain.com 请更改为站点域名且路径必须存在

Type of Task：Shell Script
Name of Task：v2board
Period：N Minutes 1 Minute
Script content：php /www/wwwroot/js.xxx.me/artisan schedule:run
```

根据上述信息添加每1分钟执行一次的定时任务。

### 7.启动队列服务

队列服务将会应用在邮件发送等场景，请务必保证队列服务在后台运行正常。

你可以使用 nohup 让其在后台运行，但是 nohup 无法保证队列服务不会退出。 使用 nohup 方式你需要在站点目录下执行如下命令：

```shell
nohup php artisan queue:work > queue.log 2>&1 &
```

如果你想让队列服务长期保持稳定的在后台运作，你需要使用第三方软件如 supervisor 的 buff 加持。

如何使用 supervisor 在 google 你会得到想要的答案。
