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
- 
<img width="657" alt="0" src="https://user-images.githubusercontent.com/92876637/152000482-329fb3be-1d35-48dd-9e4a-b2d7abb6f0d3.PNG">


Install nginx pada VM, dan setting codeigniter untuk lxc_php5_1 dan lxc_php5_2 dengan IP yang telah di atur



kemudian, buat ansible codeiginiter dengan nama deploy-app.yml yang berisi domain container yang digunakan.


```
- hosts: ci
  vars:
    git_url: 'https://github.com/aldonesia/sas-ci'
    destdir: '/var/www/html/ci'
    domain: 'lxc_php5_1.dev'
    domain: 'lxc_php5_2.dev'
  roles:
    - app
```

*picture*



kemudian, jalankan Ansible

*picture*



Install mariadb pada LXC_DB_SERVER dan CodeIgniter 3 pada LXC_CI dengan ansible

<img width="480" alt="install mariadb" src="https://user-images.githubusercontent.com/92876637/152000830-fb0c33a5-ad9a-4208-9220-ef0b533efcd8.PNG">


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

<img width="481" alt="install wp" src="https://user-images.githubusercontent.com/92876637/152000931-0d739789-12bc-4f11-b397-0524b4a82c42.PNG">


*picture 2*

*picture 3*

*picture 4*

*picture 5*



Install YII 2.0 on LXC_YII dengan ansible

*link ansible*

<img width="482" alt="install yii" src="https://user-images.githubusercontent.com/92876637/152000974-f05bcf3a-b8e1-432a-bd94-3c98a4d3541e.PNG">


*picture 2*



copy LXC sebelumnya, seperti arsitektur 




