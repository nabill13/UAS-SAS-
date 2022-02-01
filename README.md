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

kemudian install ci, laravel, wordpress, yii dan mariadb

```
nano install-ci.yml //for code igniter
nano install-laravel.yml //for laravel
nano install-wp.yml //for wordpress
nano install-yii.yml //for yii2.0
nano install-mariadb.yml //for phpmyadmin
```

*Install WordPress*
<img width="481" alt="install wp" src="https://user-images.githubusercontent.com/92932656/152006345-92221ff8-11ca-4b97-85d2-1de041e81e0f.PNG">

*Install yii*
<img width="482" alt="install yii" src="https://user-images.githubusercontent.com/92932656/152006473-784207e0-e9c0-421a-afa0-fe3580ffce98.PNG">

*Install mariadb*
<img width="480" alt="install mariadb" src="https://user-images.githubusercontent.com/92932656/152006534-4ff7d8f6-b580-4a1d-b84b-2e1eb89e1c4b.PNG">


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





