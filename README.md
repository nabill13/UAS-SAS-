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

<img width="954" alt="1" src="https://user-images.githubusercontent.com/92876637/151707964-2eec3c05-7e6f-4ac6-a228-85b5bce20cc0.PNG">

<img width="448" alt="2" src="https://user-images.githubusercontent.com/92876637/151707996-56ae0462-1d8a-4098-bbd8-5e9f725ff08a.PNG">

<img width="484" alt="3" src="https://user-images.githubusercontent.com/92876637/151708022-90774ddd-a35b-4022-bbc5-4d13e0aa9c32.PNG">

Install nginx pada VM, dan setting codeigniter untuk lxc_php5_1 dan lxc_php5_2 dengan IP yang telah di atur

<img width="238" alt="4" src="https://user-images.githubusercontent.com/92876637/151708030-739ad761-34d0-4d08-a562-721e05fd0f92.PNG">   <img width="215" alt="5" src="https://user-images.githubusercontent.com/92876637/151708092-b3dbf2f1-ffad-4ecc-bd11-b4f6ab280dac.PNG">


kemudian, buat ansible codeiginiter dengan nama deploy-app.yml yang berisi domain container yang digunakan.

<img width="481" alt="6" src="https://user-images.githubusercontent.com/92876637/151708105-684fbfdf-df88-441e-9aa0-7acd476eb0a3.PNG">


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

*picture 1*

*picture 2*

*picture 3*

*picture 4*

*picture 5*



Install YII 2.0 on LXC_YII with ansible

*link ansible*

*picture 1*

*picture 2*



Copy LXC earlier on steps 1, like architecture like





