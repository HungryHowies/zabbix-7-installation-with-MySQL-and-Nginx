# zabbix-7-installation-with-MySQL-and-Nginx

Login as root

wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb
dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb
apt update
apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
apt install mysql-server
systemctl start mysql.service
systemctl enable mysql.service
mysql -uroot -p
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
mysql -uroot -p
set global log_bin_trust_function_creators = 0;
quit;

Edit file 

vi /etc/zabbix/zabbix_server.conf

Set password.

DBPassword=password

Save and Exit file.

Certificate 

Change directory.
cd /etc/zabbix
```
openssl req -newkey rsa:4096 \
-x509 \
-sha256 \
-days 3650 \
-nodes \
-out 192.168.200.103.crt \
-keyout 192.168.200.103.key
```
Edit file. 

vi /etc/zabbix/nginx.conf

Uncomment and set 'listen' and 'server_name' directives.

listen 8080;
server_name 192.168.200.103;

Add the following line under the block called  location ~ [^/]\.php(/|$)
listen 443 ssl; # managed by me
ssl_certificate /etc/zabbix/192.168.200.103.crt; # managed by me
ssl_certificate_key /etc/zabbix/192.168.200.103.key; # managed by me
Example:
```
       location ~ [^/]\.php(/|$) {
                fastcgi_pass    unix:/var/run/php/zabbix.sock;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_index   index.php;

                fastcgi_param   DOCUMENT_ROOT   /usr/share/zabbix;
                fastcgi_param   SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;
                fastcgi_param   PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;

                include fastcgi_params;
                fastcgi_param   QUERY_STRING    $query_string;
                fastcgi_param   REQUEST_METHOD  $request_method;
                fastcgi_param   CONTENT_TYPE    $content_type;
                fastcgi_param   CONTENT_LENGTH  $content_length;

                fastcgi_intercept_errors        on;
                fastcgi_ignore_client_abort     off;
                fastcgi_connect_timeout         60;
                fastcgi_send_timeout            180;
                fastcgi_read_timeout            180;
                fastcgi_buffer_size             128k;
                fastcgi_buffers                 4 256k;
                fastcgi_busy_buffers_size       256k;
                fastcgi_temp_file_write_size    256k;
        }
        listen 443 ssl; # managed by me
        ssl_certificate /etc/zabbix/192.168.200.103.crt; # managed by me
        ssl_certificate_key /etc/zabbix/192.168.200.103.key; # managed by me

}
```
Save and Exit

https://communityhub.nullstack.com/link/20#bkmrk-test%C2%A0-site-file
 
Test Site file

nginx -t

Start Zabbix server and agent processes and make it start at system boot.
systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm
systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm
