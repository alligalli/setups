# Installing Git and Apache HTTP Server

## Update and upgrade packages

It's good to keep things up to date.

```
sudo apt update
sudo apt upgrade
```

Install the packages we will need for the setup.

```
sudo apt install git apache2 apache2-utils
```

## Configure Apache HTTP Server for Git

```
sudo a2enmod env cgi alias rewrite
sudo mkdir /var/www/git
sudo nano /etc/apache2/sites-available/git.conf
```

---

*git.conf*

```
<VirtualHost *:80>
  ServerAdmin webmaster@localhost
 
  SetEnv GIT_PROJECT_ROOT /var/www/git
  SetEnv GIT_HTTP_EXPORT_ALL
  ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
 
  Alias /git /var/www/git
 
  <Directory /usr/lib/git-core>
    Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
    AllowOverride None
    Require all granted
  </Directory>
 
  DocumentRoot /var/www/html
 
  <Directory /var/www>
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Require all granted
  </Directory>
  
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

---

```
sudo a2dissite 000-default.conf
sudo a2ensite git.conf
sudo systemctl restart apache2
sudo nano /usr/local/bin/git-create-repo.sh
```
  
---
  
*git-create-repo.sh*

```
#!/bin/bash
 
GIT_DIR="/var/www/git"
REPO_NAME=$1
 
mkdir -p "${GIT_DIR}/${REPO_NAME}.git"
cd "${GIT_DIR}/${REPO_NAME}.git"
 
git init --bare &> /dev/null
touch git-daemon-export-ok
cp hooks/post-update.sample hooks/post-update
git config http.receivepack true
git update-server-info
  
chown -Rf www-data:www-data "${GIT_DIR}/${REPO_NAME}.git"
  
echo "Git repository '${REPO_NAME}' created in ${GIT_DIR}/${REPO_NAME}.git"
```

---

```
sudo chmod +x /usr/local/bin/git-create-repo.sh
sudo git-create-repo.sh test
git clone http://SERVER-IP-ADDRESS/git/test.git
cd test/
echo "Hello World" > hello
git add .
git commit -m "initial commit"
git push origin
```

## Configure User Authentication

```  
sudo nano /etc/apache2sites-available/git.conf
```
  
---

*git.conf*

```
<VirtualHost *:80>
  
  ...
 
  <LocationMatch /git/.*\.git>
    AuthType Basic
    AuthName "Git Verification"
    AuthUserFile /etc/apache2/git.passwd
    Require valid-user
  </LocationMatch>
  
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

---

```
sudo htpasswd -c /etc/apache2/git.passwd USERNAME
# remove -c to add other users to the db
sudo systemctl restart apache2
```
  
## Add git user for SSH access

Create a new git user and disable change it's default shell (bash) to a non login one (git-shell).

```
sudo useradd git
# ensure that '/usr/bin/git-shell' is listed in the /etc/shells
sudo usermod -s /usr/bin/git-shell git
```

Add a necessary folder for the git account in it's home directory.

```
cd /home/git/
mkdir git-shell-commands
chmod 755 git-shell-commands
```

Create a symbolic link to from the /var/www/git directory to the git user home directory.

```
sudo ln -s /var/www/git /home/git
```

Add the public keys of the team members (id_rsa.pub) to git’s .ssh/authorized_keys file.

## Enable firewall rules with UFW

Simple set of firewall rules using UFW

```
sudo ufw allow ssh
sudo ufw allow http
sudo ufw enable
```

## The End!

You can now access your repositories like this:
```
# SSH
ssh://git@SERVER-IP-ADDRESS/~/git/example.git
```

```
# HTTP
http://SERVER-IP-ADDRESS/git/test.git
```