


# 本地部署 Mastodon - 一个去中心化的社交平台(非docker方式）

## 开始之前

准备一个域名和证书

- 域名：`yourDomain.com`

如果只是想本地跑一下，也行

- 修改hosts：`127.0.0.1 yourDomain.com`

没有域名用localhost也可以

### 切换root模式
```
sudo -i
```
### 安装node.js
```
curl -sL https://deb.nodesource.com/setup_16.x | bash -
apt-get install nodejs -y
```
### 安装yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
```
### 安装系统依赖 (Ubuntu/Debian为例)

```
apt update
sudo snap install --classic certbot
sudo apt install libgdbm-dev
apt install -y \
  imagemagick ffmpeg libpq-dev libxml2-dev libxslt1-dev file git-core \
  g++ libprotobuf-dev protobuf-compiler pkg-config nodejs gcc autoconf \
  bison build-essential libssl-dev libyaml-dev libreadline6-dev \
  zlib1g-dev libncurses5-dev libffi-dev libgdbm-dev \
  nginx redis-server redis-tools postgresql postgresql-contrib \
  certbot yarn libidn11-dev libicu-dev libjemalloc-dev
sudo apt install python3-certbot-nginx
```
### 创建一个mastodon用户
```
adduser --disabled-login mastodon
在这里会问你一些问题
Enter the new value, or press ENTER for the default
	Full Name []: Mastodon
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
```
```
sudo usermod -s /bin/bash mastodon
su - mastodon
```
### 安装 rbenv 和 rbenv-build
```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
cd ~/.rbenv && src/configure && make -C src
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec bash
cd ../
source ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
rbenv install 3.0.6
rbenv global 3.0.6
```
### 安装 bundler
```
gem install bundler
exit
```
### 配置 PostgreSQL
```
sudo -u postgres psql
CREATE USER mastodon CREATEDB;
ALTER USER mastodon WITH PASSWORD '123';
\q
```
### 配置 Mastodon
```
su - mastodon
git clone https://github.com/mastodon/mastodon.git live && cd live
git checkout v4.1.4
bundle config deployment 'true'
bundle config without 'development test'
bundle install -j$(nproc)
yarn install --pure-lockfile
```
### 生成配置文件
它将：

创建一个配置文件
预编译静态文件
创建数据库schema
配置文件被保存在.env.production。
```
RAILS_ENV=production bundle exec rake mastodon:setup
```
配置过程如下
```
Domain name: nuwa.social

Single user mode disables registrations and redirects the landing page to your public profile.
Do you want to enable single user mode? No

Are you using Docker to run Mastodon? no

PostgreSQL host: /var/run/postgresql
PostgreSQL port: 5432
Name of PostgreSQL database: mastodon
Name of PostgreSQL user: mastodon
Password of PostgreSQL user: 
Database configuration works! 🎆

Redis host: localhost
Redis port: 6379
Redis password: 
Redis configuration works! 🎆

Do you want to store uploaded files on the cloud? No

Do you want to send e-mails from localhost? No
SMTP server: smtp-relay.brevo.com
SMTP port: 587
SMTP username: 8651ac001@smtp-brevo.com
SMTP password: 
SMTP authentication: plain
SMTP OpenSSL verify mode: none
Enable STARTTLS: auto
E-mail address to send e-mails "from": zaran <bg@nuwa.social>
Send a test e-mail with this configuration right now? no

This configuration will be written to .env.production
Save configuration? Yes

Now that configuration is saved, the database schema must be loaded.
If the database already exists, this will erase its contents.
Prepare the database now? Yes
Running `RAILS_ENV=production rails db:setup` ...


Created database 'mastodon'
Done!

The final step is compiling CSS/JS assets.
This may take a while and consume a lot of RAM.
Compile the assets now? Yes
Running `RAILS_ENV=production rails assets:precompile` ...


yarn install v1.22.22
[1/6] Validating package.json...
[2/6] Resolving packages...
[3/6] Fetching packages...
[4/6] Linking dependencies...
warning Workspaces can only be enabled in private projects.
[5/6] Building fresh packages...
[6/6] Cleaning modules...
Done in 4.25s.
I, [2025-02-23T16:09:15.427547 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/doorkeeper/admin/application-a644908e7bab54fb749be0f59fb64a7480bbf9c4c2b79d4a65791cb7ab4d8730.css
I, [2025-02-23T16:09:15.427832 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/doorkeeper/admin/application-a644908e7bab54fb749be0f59fb64a7480bbf9c4c2b79d4a65791cb7ab4d8730.css.gz
I, [2025-02-23T16:09:15.432482 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/doorkeeper/application-c93dac2ad9d65e3393e0e2c958481e86ef7a5e5b0f6ce406842a7b99b25a4850.css
I, [2025-02-23T16:09:15.432525 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/doorkeeper/application-c93dac2ad9d65e3393e0e2c958481e86ef7a5e5b0f6ce406842a7b99b25a4850.css.gz
I, [2025-02-23T16:09:15.433421 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/pghero/favicon-db10337a56c45eb43c22ff5019546b520fa22c7281d4d385f235cbca67ed26bb.png
I, [2025-02-23T16:09:15.589642 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/pghero/application-a60bf0a452ed064fef3594cf52a4c998712da7c76150f890f4eaa644f59671e4.js
I, [2025-02-23T16:09:15.589725 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/pghero/application-a60bf0a452ed064fef3594cf52a4c998712da7c76150f890f4eaa644f59671e4.js.gz
I, [2025-02-23T16:09:15.593019 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/pghero/application-c31338f656687c1d733bb0f48d40acd076e24060f3dcff83b34870e4ccc2789d.css
I, [2025-02-23T16:09:15.593053 #37664]  INFO -- : Writing /home/mastodon/live/public/assets/pghero/application-c31338f656687c1d733bb0f48d40acd076e24060f3dcff83b34870e4ccc2789d.css.gz
Compiling...
Compiled all packs in /home/mastodon/live/public/packs
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme

Done!

All done! You can now power on the Mastodon server 🐘

Do you want to create an admin user straight away? Yes
Username: admin
E-mail: 11@qq.com
You can login with the password: a94c4753daeae884faa5fa8b085984a8
You can change your password once you login.

```
```
exit
```
### 配置 nginx
```
cp /home/mastodon/live/dist/nginx.conf /etc/nginx/sites-available/mastodon
ln -s /etc/nginx/sites-available/mastodon /etc/nginx/sites-enabled/mastodon
```

编辑 /etc/nginx/sites-available/mastodon

替换 example.com 为你自己的域名
启用 ssl_certificate 和 ssl_certificate_key 这两行，并把它们替换成如下两行（如果你使用自己的证书的话则可以忽略这一步）
```
ssl_certificate     /etc/ssl/certs/ssl-cert-snakeoil.pem;
ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
```
ssl证书可参考我之前发的mastodon_docker
### 配置 systemd 服务
```
cp /home/mastodon/live/dist/mastodon-*.service /etc/systemd/system/

systemctl daemon-reload
systemctl start mastodon-web mastodon-sidekiq mastodon-streaming
systemctl enable mastodon-*

sudo systemctl enable mastodon-web.service
sudo systemctl enable mastodon-sidekiq.service
sudo systemctl enable mastodon-streaming.service

sudo systemctl start mastodon-web.service
sudo systemctl start mastodon-sidekiq.service
sudo systemctl start mastodon-streaming.service

sudo systemctl status mastodon-web.service
sudo systemctl status mastodon-sidekiq.service
sudo systemctl status mastodon-streaming.service
```
### 要停止这些服务，你可以使用 systemctl stop 命令
```
sudo systemctl stop mastodon-web.service
sudo systemctl stop mastodon-sidekiq.service
sudo systemctl stop mastodon-streaming.service
```
### 如果你不希望这些服务在系统启动时自动启动，可以使用 systemctl disable 命令：
```
sudo systemctl disable mastodon-web.service
sudo systemctl disable mastodon-sidekiq.service
sudo systemctl disable mastodon-streaming.service
```

### 下次需要启动这些服务时，可以使用 systemctl start 命令：
```
sudo systemctl start mastodon-web.service
sudo systemctl start mastodon-sidekiq.service
sudo systemctl start mastodon-streaming.service
```

### 如果你想让这些服务在系统启动时自动启动，可以使用 systemctl enable 命令：
```
sudo systemctl enable mastodon-web.service
sudo systemctl enable mastodon-sidekiq.service
sudo systemctl enable mastodon-streaming.service
```


