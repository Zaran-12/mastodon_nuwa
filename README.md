


# 本地部署 Mastodon - 一个去中心化的社交平台(非docker方式）

## 开始之前

准备一个域名和证书

- 域名：`yourDomain.com`

如果只是想本地跑一下，也行

- 修改hosts：`127.0.0.1 yourDomain.com`

没有域名用localhost也可以

### 安装系统依赖 (Ubuntu/Debian为例)

```
sudo apt update && sudo apt upgrade -y
sudo apt install -y \
  git curl build-essential libssl-dev libreadline-dev \
  zlib1g-dev libidn11-dev libicu-dev libpq-dev redis-server \
  imagemagick ffmpeg libprotobuf-dev protobuf-compiler

```

### 安装Ruby和Node.js

```

git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
rbenv install 3.0.6
rbenv global 3.0.6

```

### 安装Node.js 16.x和Yarn：

```

curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g yarn

```

### 配置PostgreSQL

```

sudo -u postgres psql
# 在PostgreSQL控制台执行：
CREATE USER mastodon CREATEDB;
ALTER USER mastodon WITH PASSWORD 'newpassword';
\q

```

### 克隆Mastodon仓库并安装依赖

```

git clone https://github.com/mastodon/mastodon.git
cd mastodon
git checkout v4.1.4  # 使用稳定版本
bundle config deployment 'true'
bundle config without 'development test'
bundle install -j$(nproc)
yarn install --pure-lockfile

```

### 生成Mastodon配置文件

```

cp .env.production.sample .env.production
nano .env.production

```

修改LOCAL_DOMAIN=yourDomain.com
DB_HOST=localhost
DB_USER=mastodon
DB_NAME=mastodon
DB_PASS=your_password  # 与PostgreSQL用户密码一致
REDIS_HOST=localhost

### 初始化数据库

```
gem install rails
```

```

RAILS_ENV=production bundle exec rails db:setup
```

### 这里可能遇见一些问题

1.ArgumentError:scret_key_base for production environment must be a type of String.            这是因为缺少密钥，通过运行以下几个代码来生成密钥，生成的密钥填在.env.production的对应位置

```

RAILS_ENV=production rails secret
npm install -g web-push
web-push generate-vapid-keys

```

2.Redis::CannotConnectError: Error connecting to Redis on localhost:6379 
Redis连接不上，需要自己临时搭建一个redis，参考：https://blog.csdn.net/weixin_40165163/article/details/103184001
搭建完后在一个独立的终端运行redis-server，不要关闭
安装完了之后卸载redis

```

sudo apt-get remove --purge redis-server

```

3.ActiveRecord::ProtectedEnvironmentError ，表示你在生产环境中运行了一个破坏性操作（如重建数据库），而 Rails 会对生产环境进行保护，避免这种操作。

```

DISABLE_DATABASE_ENVIRONMENT_CHECK=1 RAILS_ENV=production bundle exec rails db:setup

```

### 编译静态资源

```

RAILS_ENV=production bundle exec rails assets:precompile

```

### 这里可能遇见一些问题

1.code: 'ERR_OSSL_EVP_UNSUPPORTED'

```

export NODE_OPTIONS=--openssl-legacy-provider
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
unset NODE_OPTIONS
node -v

```

### 恭喜你🎉成功搭建，现在运行下面的命令，每一个命令都在一个单独的终端运行

```

# Web服务器 (端口3000)
RAILS_ENV=production bundle exec rails s -p 3000 -b 0.0.0.0
# Sidekiq任务队列
RAILS_ENV=production bundle exec sidekiq
# Streaming服务 (端口4000)
RAILS_ENV=production node ./streaming

```

### 创建管理员账户

```
RAILS_ENV=production bundle exec rails mastodon:webpush:generate_vapid_key
RAILS_ENV=production bundle exec rails mastodon:setup
# 按提示输入邮箱和密码，获取管理员权限

```

### 关掉代理访问

```

http://yourDomain.com

```

或者你用的localhost

```

localhost：3000

```
