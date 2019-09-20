Install package & activate apache mods  
```
apt-get install letsencrypt python-letsencrypt-apache
a2enmod rewrite
a2enmod proxy_connect
a2enmod ssl
a2enmod proxy_http
a2enmod headers
a2enmod proxy_wstunnel
a2enmod cgi
```
Create & enable apache FQDN:  
```
echo "ServerName `cat /etc/hostname`" >> /etc/apache2/conf-available/fqdn.conf
a2enconf fqdn 
```
After DNS configured & don't miss to open firewall(Azure for example) on 443 port before !! )  
```
letsencrypt certonly -d sub.mydomain.com
```

Edit apache SSL configuration   
```properties
  ServerName sub.mydomain.com
  SSLProxyCheckPeerCN off
  SSLProxyCheckPeerExpire off
  SSLProxyCheckPeerName off
  SSLProxyEngine On
  SSLProxyVerify none
  ProxyPreserveHost On
  ProxyPass /myapp http://127.0.0.1:8080/myapp
  ProxyPassReverse /myapp http://127.0.0.1:8080/myapp
  RequestHeader set X-Forwarded-Proto https
  SSLCertificateFile /etc/letsencrypt/live/submydomain.com/fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/sub.mydomain.com/privkey.pem
  Include /etc/letsencrypt/options-ssl-apache.conf
  # Custom log file for SSL
  ErrorLog ${APACHE_LOG_DIR}/ssl.error.log
  CustomLog ${APACHE_LOG_DIR}/ssl.access.log combined

  # Parts websocket
  #RewriteEngine on
  #RewriteCond %{HTTP:UPGRADE} ^WebSocket$ [NC]
  #RewriteCond %{HTTP:CONNECTION} ^Upgrade$ [NC]
  #RewriteRule .* ws://127.0.0.1:8092%{REQUEST_URI} [P]
```
And restart Apache .....   

Create an auto renew script for let's encrypt letRenew.sh 
```bash
#!/bin/sh
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
/usr/bin/letsencrypt renew
service apache2 restart
```
Transform to executable  
```
chmod +x letRenew.sh
```
Edit crontab   
```bash
crontab -e
```
and add to list 
```conf
30 2 * * 1 /root/letRenew.sh >> /var/log/le-renew.log
```


  

