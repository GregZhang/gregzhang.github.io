---
title: CentOS安装GitLab
date: 2017-03-16 10:11:48
tags:
  - linux
  - GitLab
categories: 
  - 运维
---

# 环境

Requirements

软件	版本
CentOS	6.6
Python	2.6
Ruby	2.1.5
Git	1.7.10+
Redis	2.0+
MySQL	 
GitLab	7-8-stable
GitLab Shell	v2.6.0
yum源

为了提高软件安装速度，将yum源设置为阿里云开源镜像
```
$ cd /etc/yum.repos.d
$ wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```
# 必要软件包
```bash
$ yum -y install libicu-devel patch gcc-c++ readline-devel zlib-devel libffi-devel openssl-devel make autoconf automake libtool bison libxml2-devel libxslt-devel libyaml-devel zlib-devel openssl-devel cpio expat-devel gettext-devel curl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker
```
# 安装Git
```bash
// 查看当前git版本
$ git --version

// 如果小于1.7.10则先卸载
$ yum remove git

// 下载最新的git并安装
$ wget -O git-src.zip https://github.com/git/git/archive/master.zip
$ unzip git-src.zip
$ cd git-src
$ make prefix=/usr/local all
$ make prefix=/usr/local install
$ ln -fs /usr/local/bin/git* /usr/bin/
```
# 安装Ruby环境
```bash
$ mkdir /tmp/ruby && cd /tmp/ruby
$ curl --progress ftp://ftp.ruby-lang.org/pub/ruby/ruby-2.1.5.tar.gz | tar xz
$ cd ruby-2.1.5
$ ./configure --disable-install-rdoc
$ make && make install

$ ln -s /usr/local/bin/ruby /usr/bin/ruby
$ ln -s /usr/local/bin/gem /usr/bin/gem
$ ln -s /usr/local/bin/bundle /usr/bin/bundle

// 设置ruby gem源为淘宝
$ gem source -r https://rubygems.org/
$ gem source -a https://ruby.taobao.org/

$ gem install bundler --no-ri --no-rdoc
```
# 安装MySQL及初始化GitLab库
```bash
$ yum install mysql mysql-devel mysql-server -y
$ /etc/init.d/mysqld start

// 登录mysql创建gitab的帐号和数据库
mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'gitlab';
mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';

//测试是否可以用git帐号登录数据库
sudo -u git -H mysql -u gitlab -p -D gitlabhq_production
```

# 安装Redis
```bash
$ yum install epel-release
$ yum -y install redis
$ redis-server &
```

# 添加git帐号并允许sudo
```bash
$ useradd --comment 'GitLab' git
$ echo "git ALL=(ALL)       NOPASSWD: ALL" >>/etc/sudoers
```
# 安装GitLab
```bash
$ /home/git
$ sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 7-8-stable gitlab
$ cd /home/git/gitlab
$ sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml

// 编辑git路径, gitlab的host:port
$ vim config/gitlab.yml
// bin_path: /usr/local/bin/git
// host: localhost
// port: 80 

// 给文件夹添加相应的权限
$ chown -R git log/
$ chown -R git tmp/
$ chmod -R u+rwX  log/
$ chmod -R u+rwX  tmp/

// 创建必要的文件夹,以及复制配置文件
$ sudo -u git -H mkdir /home/git/gitlab-satellites
$ sudo -u git -H mkdir tmp/pids/
$ sudo -u git -H mkdir tmp/sockets/
$ sudo chmod -R u+rwX  tmp/pids/
$ sudo chmod -R u+rwX  tmp/sockets/
$ sudo -u git -H mkdir public/uploads
$ sudo chmod -R u+rwX  public/uploads
$ sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
$ sudo -u git -H cp config/initializers/rack_attack.rb.exampleconfig/initializers/rack_attack.rb

// 配置数据库连接信息
$ sudo -u git cp config/database.yml.mysql config/database.yml
$ sudo -u git -H vim  config/database.yml
$ vim config/database.yml
// production:
//     username: gitlab
//     password: "gitlab"
```

# 安装GitLab-Shell
```bash
$ cd /home/git
$ sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-shell.git -b v2.6.0
$ cd gitlab-shell/
$ sudo -u git -H cp config.yml.example config.yml

// 编辑配置文件, 设置gitlab_url, redis-cli, log-level...
$ vim config.yml
// gitlab_url: "http://localhost/"
// /usr/bin/redis-cli

// 安装git-shell
$ sudo -u git -H ./bin/install
```

# 安装需要ruby的gems
```bash
$ cd /home/git/gitlab
$ sudo -u git -H bundle install --deployment --without development test postgres aws
```

# 初始化数据库(创建GitLab相关表)
```bash
$ sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production

Administrator account created:
login.........root
password......5iveL!fe
```

# 安装启动文件以及日志切割文件
```bash
$ cp lib/support/init.d/gitlab /etc/init.d/gitlab
$ cp lib/support/init.d/gitlab.default.example /etc/default/gitlab
$ cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
```

# 设置git帐号信息
```
$ sudo -u git -H git config --global user.name "Troy Zhang"
$ sudo -u git -H git config --global user.email "troyz@synnex.com"
$ sudo -u git -H git config --global core.autocrlf input
```

# 安装Nginx
```bash
$ yum -y install nginx
$ vim /etc/nginx/nginx.conf
```

```
user              root git;
worker_processes  2;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
# GITLAB
# Maintainer: @randx
# App Version: 5.0

upstream gitlab {
  server unix:/home/git/gitlab/tmp/sockets/gitlab.socket;
}

server {
  listen *:80 default_server;         # e.g., listen 192.168.1.1:80; In most cases *:80 is a good idea
  server_name YOUR_SERVER_FQDN;     # e.g., server_name source.example.com;
  server_tokens off;     # don't show the version number, a security best practice
  root /home/git/gitlab/public;

  # Set value of client_max_body_size to at least the value of git.max_size in gitlab.yml
  client_max_body_size 5m;

  # individual nginx logs for this gitlab vhost
  access_log  /var/log/nginx/gitlab_access.log;
  error_log   /var/log/nginx/gitlab_error.log;

  location / {
    # serve static files from defined root folder;.
    # @gitlab is a named location for the upstream fallback, see below
    try_files $uri $uri/index.html $uri.html @gitlab;
  }

  # if a file, which is not found in the root folder is requested,
  # then the proxy pass the request to the upsteam (gitlab unicorn)
  location @gitlab {
    proxy_read_timeout 300; # https://github.com/gitlabhq/gitlabhq/issues/694
    proxy_connect_timeout 300; # https://github.com/gitlabhq/gitlabhq/issues/694
    proxy_redirect     off;

    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_set_header   Host              $http_host;
    proxy_set_header   X-Real-IP         $remote_addr;
    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;

    proxy_pass http://gitlab;
  }
}
}
```

# 更改权限，启动nginx
```bash
$ nginx -t
$ chown -R git:git /var/lib/nginx/
$ /etc/init.d/nginx start
```

# 检测当前环境
```bash
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
```

# 拉取gitlab静态资源文件
```bash
$ sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
```

# 启动gitlab
```bash
$ /etc/init.d/gitlab start
```

# 检测各个组件是否正常工作
```bash
$ sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```
