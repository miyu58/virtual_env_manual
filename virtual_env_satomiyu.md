# Vagrant環境構築手順書  
****  
| 使用ツール | バージョン | 
| :--------- | ---------- | 
| OS         | CentOS7    | 
| PHP        | 7.3.28     | 
| MySQL      | 5.7.34     | 
| Laravel    | 6.x        | 
| Nginx      | 1.19.10    |  


## 環境構築    
### Vagrant作業ディレクトリ作成  
`$ mkdir vagrant_env`  
`$ cd vagrant_env` #ディレクトリ移動  
`$ vagrant init centos/7` #vagrant init box名  

### Vagrantfile編集  
`$ vi Vgrantfile`  
#コメント外す箇所  
config.vm.network "forwarded_port", guest: 80, host: 1234  
config.vm.network "private_network", ip: “192.168.33.19"  

#コメントのまま変更  
config.vm.synced_folder “../data”, “/vagrant_data”  
下記へ変更  
config.vm.synced_folder "./", "/vagrant", type:”virtualbox”  

### VagrantゲストOS起動  
`$ vagrant up`  
`$vagrant ssh`  

### パッケージインストール  
`$ sudo yum -y groupinstall "development tools”`  

### PHPインストール  
`$ sudo yum -y install epel-release wget`  
`$ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm`  
`$ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip`  

### composerインストール  
`$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"`  
`$ php composer-setup.php`  
`$ php -r "unlink('composer-setup.php');"`  

#どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動  
`$ sudo mv composer.phar /usr/local/bin/composer`  
`$ composer -v`  

### Nginxインストール  
`$ sudo vi /etc/yum.repos.d/nginx.repo`  

- viエディタで下記内容追加  

[nginx]    
name=nginx repo      
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/  
gpgcheck=0  
enabled=1  

`$ sudo yum install -y nginx`#インストール    
`$ nginx -v`#バージョン確認  
`$ sudo systemctl start nginx` #起動  

### Nginxファイル編集  
`$ sudo vi /etc/nginx/conf.d/default.conf`  
  
location / {  
        #root  /usr/share/nginx/html; ＃コメントアウト  
        #index  index.html index.htm; ＃コメントアウト  
        try_files $uri $uri/ /index.php$is_args$args; ＃追加  
    }  

location ~ \.php$ {  
    #    root           html; ＃コメントアウトのまま  
         fastcgi_pass   127.0.0.1:9000;  
         fastcgi_index  index.php;  
         fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name; ＃/$document_root/へ変更  
         include        fastcgi_params;  
     }  

### php-fpmファイル編集  
`$ sudo vi /etc/php-fpm.d/www.conf`  
`$ ＄ sudo chmod -R 777 storage` #書き込み権限付与  

### laravel6インストール  
`$ composer create-project --prefer-dist laravel/laravel todo_app "6.*"`  

### DB（My SQL）インストール  
`$ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm`  
`$ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm`  
`$ sudo yum install -y mysql-community-server`  
`$ mysql --version`  

- パスワード設定変更  
`$ sudo vi /etc/my.cnf`  
`validate-password=OFF  ：追加`  

- mysql再起動  
`$ sudo systemctl restart mysqld`  

- 現在のパスワード取得  
`$ sudo cat /var/log/mysqld.log | grep 'temporary password'`  

- mysqlにログイン  
`$ sudo systemctl start mysqld`  
`$ mysql -u root -p`  
`Enter password:` #取得したパスワードコピペ  

- 任意のパスワードへ変更  
`mysql > set password = "新たなpassword”;`  

### データベース作成  
`mysql > create database todo-app;`  

### laravelの.envファイル編集  
`$ ＄ vi .env`  
- 作成したデータベース名に変更  
 DB_DATABASE=todo_app  

- 設定したパスワードへ変更  
 DB_PASSWORD=パスワード  

- マイグレーション  
`$ php artisan migrate`  


###  Laravelログイン機能  
`$ composer require laravel/ui "^1.0" —dev`  
`$ php artisan ui vue —auth`  

##### 画面表示確認  


*****
## 環境構築の所感  
 - バージョンの違いで打つコマンドが違っている  
 - 設定ファイルをしっかり確認していないとエラーに躓く  
 - 今までのように用意されたコマンドをただ実行していた時と違い  
   打ったコマンドの意味も確認しながらできたので理解が深まった  
 - 一から自分で構築することで背景や流れが理解できた  
 - 所々エラーで躓きましたが、調べながら自己解決できたのでよかった  

## 参考サイト  
[Laravelインストール](https://readouble.com/laravel/6.x/ja/installation.html)  
[Laravelログイン](https://readouble.com/laravel/6.x/ja/authentication.html)  
他GIZTECH参照  




