## Installation of Moodle 3.10 on CentOS 7 with Nginx, Postgresql and PHP

### 1. Install nginx
* `yum -y update`
* `yum install -y epel-release`
* `yum install nginx`

* `systemctl start nginx`
* `systemctl enable nginx`

###  Nginx directories:
*  Config dir: /etc/nginx
*  Root dir: /usr/share/nginx/html

###   Nginx configuration:
*  /etc/nginx/nginx.conf

  ```
  [http]
  #Stop nginx from displaying server tokens
  server_tokens off;

  [server]
  server_name  moodle-tom.ssystems.de;
  index   index.php;
  root         /usr/share/nginx/html/;


  # Only allow GET, HEAD & POST HTTP requests
  if ($request_method !~ ^(GET|HEAD|POST)$ ){
    return 405;
  }

  # Configuration for the use with moodle
  location ~ [^/]\.php(/|$) {
    fastcgi_split_path_info  ^(.+\.php)(/.+)$;
    fastcgi_index            index.php;
    fastcgi_pass             127.0.0.1:9000;
    include                  fastcgi_params;
    fastcgi_param   PATH_INFO	$fastcgi_path_info;
    fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
  }
```
### 2. Change ssh port to 22022:
* Edit port in /etc/ssh/sshd_config to 22022
* `firewall-cmd --add-port 22022/tcp --permanent`
* `firewall-cmd --reload`
* `semanage port -a -t ssh_port_t -p tcp 22022`
* `systemctl restart sshd`
* `firewall-cmd --remove-service ssh --permanent`
* `firewall-cmd --reload`

### 3. Install fail2ban
* `yum install fail2ban`
* `systemctl enable fail2ban`
* `touch /etc/fail2ban/jail.local`
* Paste the following code
```
[DEFAULT]
# Ban hosts for one hour:
bantime = 3600

# Override /etc/fail2ban/jail.d/00-firewalld.conf:
banaction = iptables-multiport

[sshd]
enabled = true

[nginx-http-auth]
enabled = true

```
* `systemctl start fail2ban`

### 4. Install certbot for nginx and getting a certificate:
* `yum install certbot-nginx`
* `certbot --nginx -d moodle-tom.ssystems.de`

### 5. Install postgresql using the official postgresql repo:
* Exclude postgresql from CentOS Base Repository
* `nano /etc/yum.repos.d/CentOS-Base.repo`

```
...
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

exclude=postgresql*

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
exclude=postgresql*
...
```
* `yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm`

* `yum install postgresql96 postgresql96-contrib postgresql96-libs postgresql96-server`

* `/usr/pgsql-9.6/bin/postgresql96-setup initdb`

* `systemctl start postgresql-9.6`
* `systemctl enable postgresql-9.6`

* Postgresql dir: `/var/lib/pgsql/9.6/`

* PostgreSQL Port: 5432

* Log into PostgreSQL to create moodle database & moodle user
* `postgres=# CREATE USER moodleuser WITH PASSWORD 'yourpassword';`
* `postgres=# CREATE DATABASE moodle WITH OWNER moodleuser;`

* Paste the following into `/var/lib/pgsql/9.6/data/pg_hba.conf`

```
# TYPE      DATABASE    USER            CIDR-ADDRESS        METHOD
host        moodle      moodleuser      127.0.0.1/32        password
```

### 6. Install PHP 7.4
`yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm`

```
yum install php74-php php74-php-bcmath php74-php-cli php74-php-common php74-php-devel php74-php-fpm
php74-php-gd php74-php-gmp php74-php-intl php74-php-json php74-php-mbstring php74-php-mysqlnd php74-php-opcache
php74-php-pdo php74-php-pgsql php74-php-process php74-php-pspell php74-php-soap php74-php-sodium php74-php-xml
php74-php-xmlrpc php74-php-pecl-zip
```
* PHP config file: `/etc/opt/remi/php74/php.ini`

### 7. Install Git
* `yum install git`
* `git config --global user.name "Tom Forsthoevel"`
* `git config --global user.email "tom.forsthoevel@protonmail.com"`

### 8. Install Moodle 3.10
* `git clone https://github.com/moodle/moodle.git`
* `git checkout MOODLE_310_STABLE`
* Copy the Moodle file into `/usr/share/nginx/html`
```
# chown apache /path/to/moodle
# cd /path/to/moodle/admin/cli
# sudo -u www-data /usr/bin/php74 install.php
# sudo -u www-data /usr/bin/php74 install_database.php --agree-license --adminpass=password
# chown -R root /path/to/moodle
```

* Add cronjob for moodle:
`# crontab -u apache -e`
```
* * * * * /usr/bin/php74  /usr/share/nginx/html/admin/cli/cron.php >/dev/null
```
