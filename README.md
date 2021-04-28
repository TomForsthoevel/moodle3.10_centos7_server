## Installation of Moodle 3.10 on CentOS 7 with Nginx, Postgresql and PHP

### 1. Install & configure Nginx
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
