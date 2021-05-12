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
```shell  
 $ mkdir vagrant_env     
 $ cd vagrant_env ディレクトリ移動    
 $ vagrant init centos/7 vagrant init box名    
```  

### Vagrantfile編集  
```shell
 $ vi Vagrantfile    
```    
   Vagrantfileから下記の記述を探してコメント外してください。  
```vagrantfile
 config.vm.network "forwarded_port", guest: 80, host: 1234  
 config.vm.network "private_network", ip: “192.168.33.19" ipアドレス指定:192.168.33.19  
```
   さらに下記の記述を探しコメントのまま変更してください。  
```vagrantfile
 config.vm.synced_folder “../data”, “/vagrant_data”  
 上記の記述を以下へ変更してください。  
 config.vm.synced_folder "./", "/vagrant", type:”virtualbox”  
```  

### VagrantゲストOS起動  
```shell 
 $ vagrant up  
 $ vagrant ssh  
```  

### パッケージインストール  
```shell
 $ sudo yum -y groupinstall "development tools”  
```  

### PHPインストール  
```shell
 $ sudo yum -y install epel-release wget  
 $ sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm  
 PHPバージョンを指定してインストール（今回はPHP7.3）  
 $ sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip  
```  

### composerインストール  
```shell
 $ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"  
 $ php composer-setup.php  
 $ php -r "unlink('composer-setup.php');"  
```  

  どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動  
```shell 
 $ sudo mv composer.phar /usr/local/bin/composer  
 $ composer -v  
 composerのバージョンが確認できたらOK  
```  

### Nginxインストール  
```shell
 $ sudo vi /etc/yum.repos.d/nginx.repo  
```

 - viエディタでnginx.repoに下記内容追加  
```shell
  [nginx]    
 name=nginx repo      
 baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/  
 gpgcheck=0  
 enabled=1  
```
 - 実際にNginxをインストールしていきます
```shell
 $ sudo yum install -y nginx インストール  
 $ nginx -v バージョン確認  
 $ sudo systemctl start nginx 起動  
```
### Nginxファイル編集  
```shell
 $ sudo vi /etc/nginx/conf.d/default.conf  

  以下を編集  
  location / {  
          #root  /usr/share/nginx/html; コメントアウト  
          #index  index.html index.htm; コメントアウト  
          try_files $uri $uri/ /index.php$is_args$args; 追加  
      }  

  location ~ \.php$ {  
      #    root           html; コメントアウトのまま  
           fastcgi_pass   127.0.0.1:9000;  
           fastcgi_index  index.php;  
           fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name; /$document_root/へ変更  
           include        fastcgi_params;  
       }  
```

### php-fpmファイル編集  
```shell
 $ sudo vi /etc/php-fpm.d/www.conf  
```  
 - 上記ファイル24行目付近のuser/groupの値を変更する  
```
   user = apache  
   以下に編集  
   user = nginx  
  
   group = apache  
   以下に編集  
   group = nginx  
```
 - 書き込み権限付与  
```shell
 $ sudo chmod -R 777 storage  
```

### laravel6インストール  
```shell
 $ composer create-project --prefer-dist laravel/laravel todo_app "6.*"  
```  

### DB（My SQL）インストール  
```shell
 $ sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  
 $ sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  
 $ sudo yum install -y mysql-community-server  
 $ mysql --version  
 バージョン確認できたらOK  
```
- パスワード設定変更  
```shell
 $ sudo vi /etc/my.cnf  
 以下を編集  
 validate-password=OFF 追加  

- mysql再起動  
 $ sudo systemctl restart mysqld  

- 現在のパスワード取得  
 $ sudo cat /var/log/mysqld.log | grep 'temporary password'  
 上記コマンド実行後`root@localhost: password`のpassword部分をコピーしておく  

- mysqlにログイン  
 $ sudo systemctl start mysqld  
 $ mysql -u root -p  
 Enter password: 取得したパスワードコピペ  

- 任意のパスワードへ変更  
 mysql > set password = "新たなpassword”;   
```

### データベース作成  
```sql
 mysql > create database todo-app;  
```

### Laravelの.envファイル編集  
```shell
 $ vi .env  

- 作成したデータベース名に変更
  DB_DATABASE=todo_app  

- 設定したパスワードへ変更  
  DB_PASSWORD=パスワード  

- マイグレーション  
 $ php artisan migrate  
```

### 画面表示確認    
  ブラウザURLに`http://192.168.33.19`入力しアクセス  
  Laravel画面が表示されてていればOK  

###  Laravelログイン機能  
```shell
 laravel/uiパッケージ
 $ composer require laravel/ui "^1.0" —dev  

 認証に必要なviewを作成  
 $ php artisan ui vue —auth  
```

### ログイン機能が実装されているか確認    
  ブラウザURLに再度`http://192.168.33.19`入力しアクセス  
  Laravel画面の右上にloginとregisterが表示されていればOK    

*****
## 環境構築の所感  
 - バージョンの違いで打つコマンドが違っている  
 - 設定ファイルをしっかり確認していないとエラーに躓く  
 - 今までのように用意されたコマンドをただ実行していた時と違い  
   打ったコマンドの意味も確認しながらできたので理解が深まった  
 - 一から自分で構築することで背景や流れが理解できた  
 - 所々エラーで躓きましたが、調べながら自己解決できたのでよかった  

#### 参考サイト  
[Laravelインストール](https://readouble.com/laravel/6.x/ja/installation.html)  
[Laravelログイン](https://readouble.com/laravel/6.x/ja/authentication.html)  
他GIZTECH参照  




