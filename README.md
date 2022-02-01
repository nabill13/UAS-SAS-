# FINAL PROJECT

### SISTEM ADMINISTRASI SERVER 

##### KELOMPOK 2

- NABILA NUR AMALIA_1202190013
- EVYRA RIZKI SAFITRI_1202190015
- M. PRADATA YUDA P_1202190061

Pertama-tama buat lxc, setting manual IP dan SSH, jangan lupa di autostart.

lxc yang akan dibuat :

- 6 lxc Ubuntu 20.04 PHP 7.4
- 2 lxc debian 10 PHP 5.6
- 1 LXC debian 10 mariadb server

```
sudo lxc-create -n lxc_php7_1 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_2 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_3 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_4 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_5 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php7_6 -t download -- --dist ubuntu --release focal --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

2 lxc debian 10
```
sudo lxc-create -n lxc_php5_1 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org

sudo lxc-create -n lxc_php5_2 -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```

1 lxc untuk mariadb (debian 10)
```
sudo lxc-create -n lxc_mariadb -t download -- --dist debian --release buster --arch amd64 --force-cache --no-validate --server images.linuxcontainers.org
```


<img width="657" alt="0" src="https://user-images.githubusercontent.com/92876637/152006200-1a568439-3245-40a1-a85d-371bfd4cf75e.PNG">

<img width="954" alt="1" src="https://user-images.githubusercontent.com/92876637/151707964-2eec3c05-7e6f-4ac6-a228-85b5bce20cc0.PNG">

<img width="238" alt="4" src="https://user-images.githubusercontent.com/92876637/151708030-739ad761-34d0-4d08-a562-721e05fd0f92.PNG">   <img width="215" alt="5" src="https://user-images.githubusercontent.com/92876637/151708092-b3dbf2f1-ffad-4ecc-bd11-b4f6ab280dac.PNG">


<img width="484" alt="3" src="https://user-images.githubusercontent.com/92876637/151708022-90774ddd-a35b-4022-bbc5-4d13e0aa9c32.PNG">

Pertama kita buat folder di direktori ansible sudo mkdir -p ~/ansible/uas untuk menampung semua script konfigurasi yang akan kita gunakan dalam tugas akhir ini.

Di direktori tubes, buat direktori untuk menampung semua skrip di setiap framework yang akan diinstal dalam proyek ini.

Buat direktori laravel di direktori sudo mkdir laravel untuk mengakomodasi skrip yang memungkinkan untuk menginstal laravel framewrok.

```
sudo mkdir -p laravel/tasks
sudo mkdir -p laravel/handlers
sudo mkdir -p laravel/templates
```

Di direktori task, buat beberapa file bernama main.yml. Kemudian ketikkan script seperti di bawah ini
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: Download and install Composer
  shell: curl -sS https://getcomposer.org/installer | php
  args:
    chdir: /usr/src/
    creates: /usr/local/bin/composer
    warn: false
  become: yes

- name: Add Composer to global path
  copy:
    dest: /usr/local/bin/composer
    group: root
    mode: '0755'
    owner: root
    src: /usr/src/composer.phar
    remote_src: yes
  become: yes

- name: Ansible delete file create-project
  file:
    path: /var/www/html/laravel
    state: absent

- name: composer create-project
  shell: /usr/local/bin/composer create-project laravel/laravel /var/www/html/laravel --prefer-dist --no-interaction

- name: Copy .env.template
  template:
    src=templates/env.template
    dest=/var/www/html/laravel/.env

- name: composer
  shell: cd /var/www/html/laravel; /usr/local/bin/composer install  --no-interaction

- name: key
  shell: /usr/bin/php7.4 /var/www/html/laravel/artisan key:generate

- name: chmod
  become: yes
  become_user: root
  become_method: su
  command: chmod 777 -R /var/www/html/laravel/storage

- name: Copy lv.conf
  template:
    src=templates/lv.conf
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: copy php7.conf
  template:
    src=templates/php7.conf
    dest=/etc/php/7.4/fpm/pool.d/www.conf

- name: Symlink lv.conf
  command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
  notify:
    - restart nginx

- name: Write {{ domain }} to /etc/hosts
  lineinfile:
    dest: /etc/hosts
    regexp: '.*{{ domain }}$'
    line: "127.0.0.1 {{ domain }}"
    state: present

```


Di direktori handler, buat beberapa file bernama main.yml. Kemudian ketikkan script seperti di bawah ini.
```
---
- name: restart php
  become: yes
  become_user: root
  become_method: su
  action: service name=php7.4-fpm state=restarted

- name: restart nginx
  become: yes
  become_user: root
  become_method: su
  action: service name=nginx state=restarted

```

Pada direktori templates, kita akan membuat 3 file script seperti di bawah ini.

env.template

```
APP_NAME=Landing
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://kelompok12.fpsas

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=10.0.3.100
DB_PORT=3306
DB_DATABASE=landing
DB_USERNAME=admin
DB_PASSWORD=admin

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DRIVER=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MEMCACHED_HOST=127.0.0.1

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailhog
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```


Install mariadb pada LXC_DB_SERVER dan CodeIgniter 3 pada LXC_CI dengan ansible

*link ansible*

*picture 1*

*picture 2*

*picture 3*

*picture 4*



Install laravel 8 pada LXC_LARAVEL dengan ansible

*link ansible*

*picture 1*

*picture 2*

*picture 3*

*picture 4*



Install latest wordpress pada LXC_WORDPRESS dengan ansible

*link ansible*

<img width="481" alt="install wp" src="https://user-images.githubusercontent.com/92876637/152006638-3915e03f-f952-4f16-b196-850b706f7375.PNG">


*picture 2*

*picture 3*

*picture 4*

*picture 5*



Install YII 2.0 on LXC_YII dengan ansible

*link ansible*

<img width="482" alt="install yii" src="https://user-images.githubusercontent.com/92876637/152006699-8189d217-4392-4634-9f2f-075096d891b8.PNG">

*picture 2*



copy LXC sebelumnya, seperti arsitektur 

*picture*



setelah itu tulis perintah berikut 
```
cd /roles/ci/tasks
nano main.yml
```

*picture 1*

setelah itu tulis perintah 
 ```
 cd ../templates
nano app.conf
```

*picture*

setelah itu tulis perintah 

```
cd ../handlers
nano main.yml
```

*picture*


 setelah ansible Code Igniter, maka kita ke cd task

```
cd ../../
cd php/tasks
nano main.yml
```
 *picture*
 
kemudian masuk ke main.yml dengan perintah

```
cd ../handlers
nano main.yml
```

*picture*

setelah ansible pada php selesai, selanjutnya setting laravel.
ketikkan perintah berikut 

```
cd ../../
cd lv/tasks
nano main.yml
```
*picture*


kemudian masuk ke env.template dengan perintah berikut :
```
cd ../templates
nano env.template
```

*picture*

