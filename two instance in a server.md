# You want to run two Mastodon instances on the same server, for example:
- One is nuwasocial.com
- One is libra2.org (for example)

Both instances run simultaneously without affecting each other.

## 🔧 Detailed steps
1️⃣ Create a directory for the second instance:
```
cd /home/mastodon/
cp -r live nuwa
```
2️⃣ Modify .env.production configure file
come into the new instance directory:
```
cd /home/mastodon/nuwa/
nano .env.production
```
The following contents are mainly modified:
```
LOCAL_DOMAIN=nuwasocial.com
SECRET_KEY_BASE=
OTP_SECRET=
VAPID_PRIVATE_KEY=
VAPID_PUBLIC_KEY=
DB_NAME=mastodon_nuwa
```
You can generate a new key with the command
Generate SECRET_KEY_BASE and OTP_SECRET,you need to run it twice:
```
RAILS_ENV=production bundle exec rake secret
# the first give SECRET_KEY_BASE
# the second give OTP_SECRET
```
Generate VAPID public/private keys (for Web Push notifications):
```
RAILS_ENV=production bundle exec rake mastodon:webpush:generate_vapid_key
```
3️⃣ Create new database
Suppose you originally used the mastodon database and now you create a new one:
```
sudo -u postgres createdb mastodon_nuwa -O mastodon
```
Then initialize the database:
```
RAILS_ENV=production bundle exec rails db:setup
```
Then compile the static resource:
```
#delete the old file
rm -rf public/assets public/packs
rm -rf tmp/cache
#give the file permission
sudo chown -R mastodon:mastodon /home/mastodon/nuwa
#compile
RAILS_ENV=production bundle exec rake assets:precompile
```
4️⃣ Create new systemd service file
```
cd /etc/systemd/system/
cp mastodon-web.service mastodon-web-nuwa.service
cp mastodon-sidekiq.service mastodon-sidekiq-nuwa.service
cp mastodon-streaming.service mastodon-streaming-nuwa.service
```
Then edit each file separately, changing the working directory to /home/mastodon/nuwa:
```
sudo nano /etc/systemd/system/mastodon-web-nuwa.service
sudo nano /etc/systemd/system/mastodon-sidekiq-nuwa.service
sudo nano /etc/systemd/system/mastodon-streaming-nuwa.service
#you need to change the location
WorkingDirectory=/home/mastodon/nuwa
```
5️⃣ Reload the systemd
```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
```
Then restart the new instance:
```
sudo systemctl start mastodon-web-nuwa.service
sudo systemctl start mastodon-sidekiq-nuwa.service
sudo systemctl start mastodon-streaming-nuwa.service
```
6️⃣ Config Nginx virtual host
```
sudo nano /etc/nginx/sites-available/nuwasocial.com
sudo ln -s /etc/nginx/sites-available/nuwasocial.com /etc/nginx/sites-enabled/nuwasocial.com
```
About the context of nuwasocial.com , you can follow the original configuration. But you need to modify some details:
```
server {
   if ($host = nuwasocial.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

  listen 80;
  listen [::]:80;
  server_name nuwasocial.com;
  root /home/mastodon/nuwa/public;
  location /.well-known/acme-challenge/ {
      root /var/www/html;
      allow all;
  }
  location / { return 301 https://$host$request_uri; }
 # location / { try_files $uri @proxy;}

}
```
```
upstream backend_nuwa {
    server unix:/home/mastodon/nuwa/tmp/sockets/puma.sock;
}
```
```
location / {
    proxy_pass http://backend_nuwa;
    ...
}
```
```
upstream streaming_nuwa {
    server 127.0.0.1:4000 fail_timeout=0;
}
```
```
location ^~ /api/v1/streaming {
    ...
    proxy_pass http://streaming_nuwa;
    ...
}
```
```
proxy_cache_path /var/cache/nginx_nuwa levels=1:2 keys_zone=CACHE_NUWA:10m inactive=7d max_size=1g;
```
```
proxy_cache CACHE_NUWA;
```
Then test it
```
sudo nginx -t
sudo systemctl reload nginx
```

7️⃣ Obtaining an HTTPS certificate
```
sudo certbot --nginx -d nuwasocial.com
```

## You don't need to modify SMTP in .env.production , it can adapt the old . If you persue perfect , you can do it following me.
1.

































