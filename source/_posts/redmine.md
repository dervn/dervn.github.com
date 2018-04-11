---
title: Redmine
date: 2018-03-12
tags:
  - redmine
---

Redmine升级和安装操作记录

## 备份
1. 备份3.0.1文件，包括
    * 数据库SQL
    * files目录中的上传文件
    * config目录下配置文件
    * plugins目录下所有文件

## Windows安装包执行恢复数据步骤
1. 执行exe文件 (windows)  https://downloads.bitnami.com/files/stacks/redmine/3.4.4-2/bitnami-redmine-3.4.4-2-windows-installer.exe
2. 创建新数据库 redmine_new
```
CREATE DATABASE redmine CHARACTER SET utf8;
CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'my_password';
GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
```

3. redmine主目录执行db升级命令
```
set RAILS_ENV=production
bundle exec rake db:migrate
```

4. 重启服务

## 操作步骤
* 下载最新安装包 3.3.6 http://www.redmine.org/releases/redmine-3.3.6.tar.gz (md5: 103bcfc7a0603815130fba8626c97661)，解压文件到安装目录
* 将插件文件放到plugins目录
* 进入安装目录，执行 如下命令
```
bundle install --path vendor/bundle --without development test
```
* 当前linux系统安装Mysql
* 创建数据库
```
CREATE DATABASE redmine336 CHARACTER SET utf8mb4;
CREATE USER 'redmine336'@'localhost' IDENTIFIED BY 'abcd.1234';
GRANT ALL PRIVILEGES ON redmine336.* TO 'redmine336'@'localhost';
```
* 执行3.0.1版本的备份SQL,文件 bitnami_redmine.sql
* 修改config/database.yml，修改production节点下数据库配置为当前设置
* 执行db迁移命令  
```
bundle exec rake db:migrate RAILS_ENV=production
```
* 执行插件DB迁移命令， 【如新环境无新插件可不执行】
```
bundle exec rake redmine:plugins:migrate RAILS_ENV=production
```
* 修改config/configuration.yml文件中邮件配置
```
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      address: smtp.163.com
      port: 25
      authentication: :login
      domain: smtp.163.com
      user_name: ucfdev@163.com
      password: 7788jira
      enable_starttls_auto: false
```

* 执行命令
```
bundle exec rake generate_secret_token
```

* LINUX文件系统写权限
```
mkdir -p tmp tmp/pdf public/plugin_assets
sudo chown -R redmine:redmine files log tmp public/plugin_assets
sudo chmod -R 755 files log tmp public/plugin_assets
```

* 拷贝备份文件目录 files到新环境下
* 启动测试 ，访问http://localhost:3000/
```
bundle exec rails server webrick -e production -b 0.0.0.0 -p 3001
```
## 其他参考
* Redmine 下载地址 http://www.redmine.org/projects/redmine/wiki/Download
* Redmine 安全问题 http://www.redmine.org/projects/redmine/wiki/Security_Advisories
* Redmine 安装步骤 http://www.redmine.org/projects/redmine/wiki/RedmineInstall
* Redmine 升级安装  http://www.redmine.org/projects/redmine/wiki/RedmineUpgrade
* 安装包下载地址 https://bitnami.com/stack/redmine/installer
    * WINDOWS https://bitnami.com/redirect/to/179030/bitnami-redmine-3.4.4-2-windows-installer.exe
    * LINUX https://bitnami.com/redirect/to/179023/bitnami-redmine-3.4.4-2-linux-x64-installer.run
* 手动部署步骤 https://www.cnblogs.com/colder219/p/7158294.html
* CentOS7 安装包大致步骤 http://www.cr173.com/html/50478_1.html
* 邮件配置 https://www.jianshu.com/p/e220924d17b6
 Install Redmine with Nginx, Puma, and MariaDB/MySQL on Ubuntu 14.04 https://blog.rudeotter.com/install-redmine-with-nginx-puma-and-mariadbmysql-on-ubuntu-14-04/
* Nginx 通过 passenger-install-nginx-module 安装的 nginx 和直接安装的 nginx 有什么区别？ https://ruby-china.org/topics/16303
* Redmine 性能优化方案 https://www.cnblogs.com/ToDoToTry/p/4462609.html
* MYSQL ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.10.210' (111) 解决方法 https://www.cnblogs.com/zihanxing/p/7049244.html
* Runs server as a Daemon -d
```
/usr/local/rvm/gems/ruby-2.3.6/bin/bundle exec /usr/local/rvm/gems/ruby-2.3.6/bin/rails server webrick -e production -b 0.0.0.0 -p 80 -d 
```
* Nginx 反向代理
```
server {
    listen 80;
    server_name redmine.xxx.com;
    access_log  logs/redmine_access.log main;
    error_log  logs/redmine_error.log;
    location / {
        proxy_pass http://localhost:3000;
    }
}
```

* 找到占用端口 lsof -i tcp:3000  
    * lsof -i:3000
    * netstat -anp | grep 80
    * 杀死进程 kill -9 92372 【92372是pid】
* rvm use 2.3.6 --default # 设施rvm默认使用的ruby版本
* Redmine 3.3.x does not support Ruby 2.4. Please use Ruby 2.3. Ruby 2.4 will be supported in upcoming Redmine 3.4.0
* 检查配置文件，你看到的不一定就是正确的格式
```
RE: occurred Error when start redmine /usr/local/ruby/lib/ruby/2.1.0/psych.rb:370:in `parse': (<unknown>): mapping values are not allowed in this context at line 20 column 22 (Psych::SyntaxError) - 由 Leonel Iturralde 在 将近 3 年 之前添加
I think there is a mistake in your yaml files. For example: configuration.yml, database.yml
Check your yaml files.
```
* Specified key was too long; max key length is 767 bytes  【类似问题需要检查系统要求的数据库版本，创建数据库的时候的编码方式，ruby软件版本】
```
INNODB utf8 VARCHAR(255)
INNODB utf8mb4 VARCHAR(191)
because 767 / 4 ~= 191 , and 767 / 3 ~= 255
The number of allowed characters just depends on your character set. UTF8 may use up to 3 bytes per character, utf8mb4 up to 4 bytes, and latin1 only 1 byte. Thus for utf8 your key length is limited to 255 characters, since 3*255 = 765 < 767.
```
* sudo su - #切换到root账户
    * su redmine #使用某个账户
* SecureCRT    sz rz  【sudo spctl --master-disable】
    * Session LOG文件配置  ../Logs/%H/%Y-%M-%D_%h%m%s.log
* mysqldump -h132.72.192.432 -P3307 -uroot -p htgl>d:\htgl.sql;
* Redmine开启LDAP认证 http://www.pfeng.org/archives/580
* Redmine LDAP配置 http://www.redmine.org/projects/redmine/wiki/redmineldap
