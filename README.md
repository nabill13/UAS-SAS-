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

*picture*

Install nginx pada VM, dan setting codeigniter untuk lxc_php5_1 dan lxc_php5_2 dengan IP yang telah di atur

*picture*



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





