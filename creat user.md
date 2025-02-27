
# the method to Update Password，Create User，Grant user administrator rights，Remove user administrator rights in rear

### swich mastodon:
```
sudo - mastodon
```
### Navigate to your Mastodon directory:
```
cd /home/mastodon/live
```
### Run the Rails console to interact with your Mastodon application:
```
sudo -u mastodon RAILS_ENV=production bundle exec rails console
```
----------------------------------------------
## update password

### Find the admin user and set the new password. Once in the Rails console, you can use the following commands to update the password:
```
user = User.find_by(email: "admin@yourdomain.com") # Replace with the admin email
user.password = "newpassword" # Set your new password
user.password_confirmation = "newpassword" # Confirm the new password
user.save
```
----------------------------------------------
## Grant user administrator rights

### find user
```
user = User.find_by(email: "admin@yourdomain.com")
```
### set user admin
```
user.update(admin: true)
```
### again check
```
user.admin?  # if the user is admin , it will return true
```
----------------------------------------------
## Remove user administrator rights
```
user.update(admin: false)
```
----------------------------------------------
## Create User
### create a user and set him admin
```
user = User.create(
  email: "user@example.com",           
  password: "yourpassword",            
  password_confirmation: "yourpassword",  
  admin: true                          
)
```
### check it
```
user = User.find_by(email: "user@example.com")  
user.admin?  
```
----------------------------------------------
### Exit the Rails console:
```
exit
```
### return root
```
exit
```
### Restart Mastodon services to apply the changes (if necessary):
```
sudo systemctl restart mastodon-web
sudo systemctl restart mastodon-sidekiq
sudo systemctl restart mastodon-streaming
```
