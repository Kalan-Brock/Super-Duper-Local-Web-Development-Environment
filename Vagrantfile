# -*- mode: ruby -*-
# vi: set ft=ruby :

@script = <<SCRIPT
# Fix for https://bugs.launchpad.net/ubuntu/+source/livecd-rootfs/+bug/1561250
if ! grep -q "ubuntu-xenial" /etc/hosts; then
    echo "127.0.0.1 ubuntu-xenial" >> /etc/hosts
fi

echo -e "\n--- $(tput -T xterm setaf 1)One Super Duper Local Web Development Environment coming right up!$(tput -T xterm sgr0) ---\n"

echo -e "--- Installing base packages, Vim, Curl, and Git ---"
apt-get -y install vim curl build-essential python-software-properties git > /dev/null 2>&1

echo -e "\n--- Adding repositories to update our box ---"
add-apt-repository ppa:ondrej/php > /dev/null 2>&1
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6 > /dev/null 2>&1
echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list > /dev/null 2>&1
apt-get -qq update

echo -e "\n--- Installing Apache and PHP 7.2 ---"
apt-get install -y apache2 php7.2 php7.2-bcmath php7.2-bz2 php7.2-cli php7.2-curl php7.2-intl php7.2-json php7.2-mbstring php7.2-opcache php7.2-soap php7.2-sqlite3 php7.2-xml php7.2-xsl php7.2-zip libapache2-mod-php7.2 memcached php-memcached > /dev/null 2>&1

echo -e "\n--- Configuring PHP for development environment ---"
sed -i "s/error_reporting = .*/error_reporting = E_ALL/" /etc/php/7.2/apache2/php.ini
sed -i "s/display_errors = .*/display_errors = On/" /etc/php/7.2/apache2/php.ini
echo 'echo "upload_max_filesize = 64M" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "post_max_size = 10M" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s
echo 'echo "max_execution_time = 600" >> /etc/php/7.2/apache2/conf.d/user.ini' | sudo -s

echo -e "\n--- Configuring Apache for both http and https ---"
echo "<VirtualHost *:80>
    DocumentRoot /var/www/public
    AllowEncodedSlashes On

    <Directory /var/www/public>
        Options +Indexes +FollowSymLinks
        DirectoryIndex index.php index.html
        Order allow,deny
        Allow from all
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>" > /etc/apache2/sites-available/000-default.conf
a2enmod rewrite > /dev/null 2>&1

openssl genrsa -out /etc/ssl/private/apache-selfsigned.key 1024 > /dev/null 2>&1
touch openssl.cnf
cat >> openssl.cnf <<EOF
[ req ]
prompt = no
distinguished_name = req_distinguished_name

[ req_distinguished_name ]
C = US
ST = Chicago
L = Illinois
O = Me Incorporated
OU = Web Development
CN = 192.168.33.10
emailAddress = noreply@me.com
EOF

openssl req -config openssl.cnf -new -key /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.csr > /dev/null 2>&1
openssl x509 -req -days 1024 -in /etc/ssl/certs/apache-selfsigned.csr -signkey /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt > /dev/null 2>&1

echo '<IfModule mod_ssl.c>
          <VirtualHost _default_:443>
              ServerAdmin noreply@me.com
              ServerName 192.168.33.10

              DocumentRoot /var/www/public

              ErrorLog ${APACHE_LOG_DIR}/error.log
              CustomLog ${APACHE_LOG_DIR}/access.log combined

              SSLEngine on
              SSLCertificateFile      /etc/ssl/certs/apache-selfsigned.crt
              SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

              <FilesMatch "\.(cgi|shtml|phtml|php)\$">
                SSLOptions +StdEnvVars
              </FilesMatch>

              <Directory /usr/lib/cgi-bin>
                SSLOptions +StdEnvVars
              </Directory>
          </VirtualHost>
      </IfModule>' > /etc/apache2/sites-available/default-ssl.conf

a2enmod ssl > /dev/null 2>&1
a2enmod headers > /dev/null 2>&1
a2ensite default-ssl > /dev/null 2>&1
sed -i "s/www-data/vagrant/" /etc/apache2/envvars
sudo rm -rf /var/www/html/
service apache2 restart > /dev/null 2>&1

echo -e "\n--- Increasing memcached memory size ---"
sed -i "s/-m 64/-m 1024/g" /etc/memcached.conf
service memcached restart

echo -e "\n--- Installing MySQL ---"
echo "mysql-server mysql-server/root_password password root" | debconf-set-selections
echo "mysql-server mysql-server/root_password_again password root" | debconf-set-selections
apt-get -y install mysql-server > /dev/null 2>&1
mysql -uroot -proot -e "GRANT ALL PRIVILEGES ON * . * TO 'root'@'localhost' IDENTIFIED by 'root'" > /dev/null 2>&1
mysql -uroot -proot -e "CREATE USER 'root'@'%' IDENTIFIED BY 'root'" > /dev/null 2>&1
mysql -uroot -proot -e "GRANT ALL PRIVILEGES ON * . * to 'root'@'%' identified by 'root'" > /dev/null 2>&1
mysql -uroot -proot -e "FLUSH PRIVILEGES" > /dev/null 2>&1
sed -i 's/^bind-address/#bind-address/' /etc/mysql/mysql.conf.d/mysqld.cnf
sed -i 's/^skip-external-locking/#skip-external-locking/' /etc/mysql/mysql.conf.d/mysqld.cnf
sudo service mysql restart > /dev/null 2>&1

echo -e "\n--- Installing Composer ---"
curl --silent https://getcomposer.org/installer | php > /dev/null 2>&1
mv composer.phar /usr/local/bin/composer

echo -e "\n--- Installing NodeJS and NPM ---"
apt-get -y install nodejs > /dev/null 2>&1
apt-get -y install npm > /dev/null 2>&1
sudo ln -s /usr/bin/nodejs /usr/bin/node

echo -e "\n--- Installing Sass, Gulp, Bower, and Yarn ---"
npm install -g sass gulp bower yarn > /dev/null 2>&1

echo -e "\n--- Installing WP CLI ---"
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar > /dev/null 2>&1
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp

echo -e "\n--- Installing Redis ---"
sudo apt-get install -y redis-server > /dev/null 2>&1
sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.old
sudo cat /etc/redis/redis.conf.old | grep -v bind > /etc/redis/redis.conf
echo "bind 0.0.0.0" >> /etc/redis/redis.conf
sudo update-rc.d redis-server defaults > /dev/null 2>&1
sudo /etc/init.d/redis-server start > /dev/null 2>&1

echo -e "\n--- Installing Mailhog ---"
wget --quiet -O /home/vagrant/mailhog https://github.com/mailhog/MailHog/releases/download/v1.0.0/MailHog_linux_amd64
chmod +x /home/vagrant/mailhog
touch /etc/systemd/system/mailhog.service
echo '
[Unit]
Description=MailHog Service
After=network.service vagrant.mount
[Service]
Type=simple
ExecStart=/usr/bin/env /home/vagrant/mailhog > /dev/null 2>&1 &
[Install]
WantedBy=multi-user.target
' > /etc/systemd/system/mailhog.service

sudo systemctl enable mailhog > /dev/null 2>&1
sudo systemctl start mailhog > /dev/null 2>&1

echo -e "\n--- Installing MongoDB ---"
sudo apt-get install -y mongodb-org > /dev/null 2>&1
sudo service mongod start > /dev/null 2>&1
sudo systemctl enable mongod.service > /dev/null 2>&1

echo -e "\n--- Updating all the things... ---"
sudo apt-get update > /dev/null 2>&1
sudo apt-get -y upgrade > /dev/null 2>&1

echo -e "\n--- Setting up bash profile with our custom commands ---\n"

##Backup Script
sudo touch /home/vagrant/backup.sh
cat > /home/vagrant/backup.sh <<- "EOF"
#!/bin/bash

read -p "What name would you like to give this backup?" BACKUP_NAME
cd /var/www/public
echo "Creating your backup, please wait a moment."
rm -rf /var/www/backups/${BACKUP_NAME}/
mkdir -p /var/www/backups/${BACKUP_NAME}/
tar -czf /var/www/backups/${BACKUP_NAME}/${BACKUP_NAME}.gz .
cd /var/www
echo "Backup $BACKUP_NAME created in /var/www/backups/$BACKUP_NAME."
EOF

if ! grep -q "cd /var/www" /home/vagrant/.profile; then
    echo "cd /var/www" >> /home/vagrant/.profile
    echo -e "\nalias sitebackup='bash /home/vagrant/backup.sh'" >> /home/vagrant/.profile
fi

sudo touch /etc/update-motd.d/99-custom-header
echo "#!/bin/sh" > /etc/update-motd.d/99-custom-header
echo '
cat << "EOF"
*** Super Duper Local Web Development Environment
*** https://github.com/Kalan-Brock/Super-Duper-Local-Web-Development-Environment
-------------------------------------------------------------------------------
## To Create a Site File Backup: sitebackup
-------------------------------------------------------------------------------
Http:       | http://192.168.33.10
Https:      | https://192.168.33.10
MySQL:      | Port: 3306  User: root  Password: root (SSH not needed)
Redis:      | Port: 6379
MongoDB:    | Port: 27017
Mailhog:    | http://192.168.33.10:8025

EOF' >> /etc/update-motd.d/99-custom-header
sudo chmod +x /etc/update-motd.d/99-custom-header

echo -e "\n\n--- Super Duper!  Visit your box at http://192.168.33.10 or https://192.168.33.10 ---\n\n"

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = 'ubuntu/xenial64'
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = "superduper"
  config.vm.network "forwarded_port", guest: 3306, host: 3306
  config.vm.network "forwarded_port", guest: 6379, host: 6379
  config.vm.network "forwarded_port", guest: 27017, host: 27017
  config.vm.synced_folder ".", "/var/www", :mount_options => ["dmode=777", "fmode=666"]
  config.vm.provision 'shell', inline: @script

  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
  end
end