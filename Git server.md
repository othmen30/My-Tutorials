# Setup a Git Server with SSH and Apache HTTP Server

#### Do all of this on your server!

## Update and upgrade packages

```
sudo apt update
sudo apt upgrade

sudo apt install git apache2 apache2-utils
```

## Configure Apache HTTP Server for Git

First you have to enable Apache's modules and to create a new configuration file on apache's directory (i.e git.conf).

```
sudo a2enmod env cgi alias rewrite

sudo mkdir /var/www/git
sudo nano /etc/apache2/sites-available/git.conf
```
***IMPORTANT NOTE: we have create the git folder on "/var/www/", all the code below have this PATH too, if you create your git folder on "/home"; Change the PATH "/var/www" to "/home" when you see them !!!***


Copy/Paste this code below to your new "git.conf" file:

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

Disable the "000-default.conf" of Apache and Enable the new "git.conf" file.

```
sudo a2dissite 000-default.conf
sudo a2ensite git.conf
```

Restart the Apache service to take the change.

```
sudo systemctl restart apache2
```

Now we will add a script to easily create a new repository:

```
sudo nano /usr/local/bin/git-create-repo.sh
```

Copy/Paste the code below to your new "git-create-repo file" script:
  
---
  
*git-create-repo*

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

Make the script executable and try it (It will create a git repository in /var/www/)

```
sudo chmod +x /usr/local/bin/git-create-repo.sh
sudo git-create-repo.sh test
```

Test that git functionalities are working by cloning the remote repo on your local machine, create a text file and push it to the server.

```
git clone http://SERVER-IP-ADDRESS/git/test.git
```

```
cd test/
echo "Hello World" > hello

git add .
git commit -m "initial commit"
git push origin
```

## Configure User Authentication

To add proper authentication for our users we need to edit our Apache configuration file again.

```  
sudo nano /etc/apache2sites-available/git.conf
```

Add the following to your git.conf file:

---

*git.conf*

```  
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

To add new users to the git.passwd db we can use the *htpasswd* command.

```
sudo htpasswd -c /etc/apache2/git.passwd USERNAME
```
# remove -c to add other users to the db


And restart the Apache service again, because we changed the configurations.

```
sudo systemctl restart apache2
```
  
## Add git user for SSH access

Create a new git user and disable it's default shell (bash) to a non login one (git-shell).

```
# ensure that '/usr/bin/git-shell' is listed in the /etc/shells; If not, add it!
cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
/usr/bin/git-shell
```

```
sudo useradd -m git
sudo usermod -s /usr/bin/git-shell git

# remember to setup a good password
sudo passwd git
```

```
# add your admin account to the git group
sudo usermod -a -G git YOURUSERADMIN
```

Add a necessary folder for the git account in it's home directory.

```
cd /home/git/
mkdir git-shell-commands
chmod 755 git-shell-commands
```

Create a symbolic link to from the /var/www/git directory to the git user home directory, so when you modify something on tour /home/git, it's will be modify in /var/www/git as well.

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

## The End

You can now access your repositories like this:

```
# SSH
git clone ssh://git@SERVER-IP-ADDRESS/~/git/test.git
```

```
# HTTP
git clone http://SERVER-IP-ADDRESS/git/test.git
```
