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

<img width="516" alt="00" src="https://user-images.githubusercontent.com/92932656/152089760-af95f4dd-a6b5-4644-9531-31673d78c5cc.PNG">

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

lv.conf
```
server {
     listen 80;
     listen [::]:80;

     # Log files for Debugging
     access_log /var/log/nginx/laravel-access.log;
     error_log /var/log/nginx/laravel-error.log;

     # Webroot Directory for Laravel project
     root /var/www/html/laravel/public;
     index index.php index.html index.htm;

     # Your Domain Name
     server_name {{ domain }};

     location / {
             try_files $uri $uri/ /index.php?$query_string;
     }

     # PHP-FPM Configuration Nginx
     location ~ \.php$ {
             try_files $uri =404;
             fastcgi_split_path_info ^(.+\.php)(/.+)$;
             fastcgi_pass 127.0.0.1:9001;
             fastcgi_index index.php;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
     }
}
```

php7.conf
```
; Start a new pool named 'www'.
; the variable $pool can we used in any directive and will be replaced by the
; pool name ('www' here)
[www]

; Per pool prefix
; It only applies on the following directives:
; - 'slowlog'
; - 'listen' (unixsocket)
; - 'chroot'
; - 'chdir'
; - 'php_values'
; - 'php_admin_values'
; When not set, the global prefix (or /usr) applies instead.
; Note: This directive can also be relative to the global prefix.
; Default Value: none
;prefix = /path/to/pools/$pool

; Unix user/group of processes
; Note: The user is mandatory. If the group is not set, the default user's group
;       will be used.
user = www-data
group = www-data

; The address on which to accept FastCGI requests.
; Valid syntaxes are:
;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
;                            a specific port;
;   'port'                 - to listen on a TCP socket to all addresses on a
;                            specific port;
;   '/path/to/unix/socket' - to listen on a unix socket.
; Note: This value is mandatory.
listen = 127.0.0.1:9001

; Set listen(2) backlog. A value of '-1' means unlimited.
; Default Value: 128 (-1 on FreeBSD and OpenBSD)
;listen.backlog = -1

; Set permissions for unix socket, if one is used. In Linux, read/write
; permissions must be set in order to allow connections from a web server. Many
; BSD-derived systems allow connections regardless of permissions.
; Default Values: user and group are set as the running user
;                 mode is set to 0666
listen.owner = www-data
listen.group = www-data
listen.mode = 0666

; List of ipv4 addresses of FastCGI clients which are allowed to connect.
; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
; must be separated by a comma. If this value is left blank, connections will be
; accepted from any ip address.
; Default Value: any
;listen.allowed_clients = 127.0.0.1

; Choose how the process manager will control the number of child processes.
; Possible Values:
;   static  - a fixed number (pm.max_children) of child processes;
;   dynamic - the number of child processes are set dynamically based on the
;             following directives. With this process management, there will be
;             always at least 1 children.
;             pm.max_children      - the maximum number of children that can
;                                    be alive at the same time.
;             pm.start_servers     - the number of children created on startup.
;             pm.min_spare_servers - the minimum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is less than this
;                                    number then some children will be created.
;             pm.max_spare_servers - the maximum number of children in 'idle'
;                                    state (waiting to process). If the number
;                                    of 'idle' processes is greater than this
;                                    number then some children will be killed.
;  ondemand - no children are created at startup. Children will be forked when
;             new requests will connect. The following parameter are used:
;             pm.max_children           - the maximum number of children that
;                                         can be alive at the same time.
;             pm.process_idle_timeout   - The number of seconds after which
;                                         an idle process will be killed.
; Note: This value is mandatory.
pm = dynamic

; The number of child processes to be created when pm is set to 'static' and the
; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
; This value sets the limit on the number of simultaneous requests that will be
; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
; CGI. The below defaults are based on a server without much resources. Don't
; forget to tweak pm.* to fit your needs.
; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
; Note: This value is mandatory.
pm.max_children = 5

; The number of child processes created on startup.
; Note: Used only when pm is set to 'dynamic'
; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
pm.start_servers = 2

; The desired minimum number of idle server processes.
; Note: Used only when pm is set to 'dynamic'
; Note: Mandatory when pm is set to 'dynamic'
pm.min_spare_servers = 1

; The desired maximum number of idle server processes.
; Note: Used only when pm is set to 'dynamic'
; Note: Mandatory when pm is set to 'dynamic'
pm.max_spare_servers = 3

; The number of seconds after which an idle process will be killed.
; Note: Used only when pm is set to 'ondemand'
; Default Value: 10s
;pm.process_idle_timeout = 10s;

; The number of requests each child process should execute before respawning.
; This can be useful to work around memory leaks in 3rd party libraries. For
; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
; Default Value: 0
pm.max_requests = 100

; The URI to view the FPM status page. If this value is not set, no URI will be
; recognized as a status page. It shows the following informations:
;   pool                 - the name of the pool;
;   process manager      - static, dynamic or ondemand;
;   start time           - the date and time FPM has started;
;   start since          - number of seconds since FPM has started;
;   accepted conn        - the number of request accepted by the pool;
;   listen queue         - the number of request in the queue of pending
;                          connections (see backlog in listen(2));
;   max listen queue     - the maximum number of requests in the queue
;                          of pending connections since FPM has started;
;   listen queue len     - the size of the socket queue of pending connections;
;   idle processes       - the number of idle processes;
;   active processes     - the number of active processes;
;   total processes      - the number of idle + active processes;
;   max active processes - the maximum number of active processes since FPM
;                          has started;
;   max children reached - number of times, the process limit has been reached,
;                          when pm tries to start more children (works only for
;                          pm 'dynamic' and 'ondemand');
; Value are updated in real time.
; Example output:
;   pool:                 www
;   process manager:      static
;   start time:           01/Jul/2011:17:53:49 +0200
;   start since:          62636
;   accepted conn:        190460
;   listen queue:         0
;   max listen queue:     1
;   listen queue len:     42
;   idle processes:       4
;   active processes:     11
;   total processes:      15
;   max active processes: 12
;   max children reached: 0
;
; By default the status page output is formatted as text/plain. Passing either
; 'html', 'xml' or 'json' in the query string will return the corresponding
; output syntax. Example:
;   http://www.foo.bar/status
;   http://www.foo.bar/status?json
;   http://www.foo.bar/status?html
;   http://www.foo.bar/status?xml
;
; By default the status page only outputs short status. Passing 'full' in the
; query string will also return status for each pool process.
; Example:
;   http://www.foo.bar/status?full
;   http://www.foo.bar/status?json&full
;   http://www.foo.bar/status?html&full
;   http://www.foo.bar/status?xml&full
; The Full status returns for each process:
;   pid                  - the PID of the process;
;   state                - the state of the process (Idle, Running, ...);
;   start time           - the date and time the process has started;
;   start since          - the number of seconds since the process has started;
;   requests             - the number of requests the process has served;
;   request duration     - the duration in µs of the requests;
;   request method       - the request method (GET, POST, ...);
;   request URI          - the request URI with the query string;
;   content length       - the content length of the request (only with POST);
;   user                 - the user (PHP_AUTH_USER) (or '-' if not set);
;   script               - the main script called (or '-' if not set);
;   last request cpu     - the %cpu the last request consumed
;                          it's always 0 if the process is not in Idle state
;                          because CPU calculation is done when the request
;                          processing has terminated;
;   last request memory  - the max amount of memory the last request consumed
;                          it's always 0 if the process is not in Idle state
;                          because memory calculation is done when the request
;                          processing has terminated;
; If the process is in Idle state, then informations are related to the
; last request the process has served. Otherwise informations are related to
; the current request being served.
; Example output:
;   ************************
;   pid:                  31330
;   state:                Running
;   start time:           01/Jul/2011:17:53:49 +0200
;   start since:          63087
;   requests:             12808
;   request duration:     1250261
;   request method:       GET
;   request URI:          /test_mem.php?N=10000
;   content length:       0
;   user:                 -
;   script:               /home/fat/web/docs/php/test_mem.php
;   last request cpu:     0.00
;   last request memory:  0
;
; Note: There is a real-time FPM status monitoring sample web page available
;       It's available in: ${prefix}/share/fpm/status.html
;
; Note: The value must start with a leading slash (/). The value can be
;       anything, but it may not be a good idea to use the .php extension or it
;       may conflict with a real PHP file.
; Default Value: not set
pm.status_path = /php-status

; The ping URI to call the monitoring page of FPM. If this value is not set, no
; URI will be recognized as a ping page. This could be used to test from outside
; that FPM is alive and responding, or to
; - create a graph of FPM availability (rrd or such);
; - remove a server from a group if it is not responding (load balancing);
; - trigger alerts for the operating team (24/7).
; Note: The value must start with a leading slash (/). The value can be
;       anything, but it may not be a good idea to use the .php extension or it
;       may conflict with a real PHP file.
; Default Value: not set
;ping.path = /ping

; This directive may be used to customize the response of a ping request. The
; response is formatted as text/plain with a 200 response code.
; Default Value: pong
;ping.response = pong

; The access log file
; Default: not set
;access.log = log/$pool.access.log

; The access log format.
; The following syntax is allowed
;  %%: the '%' character
;  %C: %CPU used by the request
;      it can accept the following format:
;      - %{user}C for user CPU only
;      - %{system}C for system CPU only
;      - %{total}C  for user + system CPU (default)
;  %d: time taken to serve the request
;      it can accept the following format:
;      - %{seconds}d (default)
;      - %{miliseconds}d
;      - %{mili}d
;      - %{microseconds}d
;      - %{micro}d
;  %e: an environment variable (same as $_ENV or $_SERVER)
;      it must be associated with embraces to specify the name of the env
;      variable. Some exemples:
;      - server specifics like: %{REQUEST_METHOD}e or %{SERVER_PROTOCOL}e
;      - HTTP headers like: %{HTTP_HOST}e or %{HTTP_USER_AGENT}e
;  %f: script filename
;  %l: content-length of the request (for POST request only)
;  %m: request method
;  %M: peak of memory allocated by PHP
;      it can accept the following format:
;      - %{bytes}M (default)
;      - %{kilobytes}M
;      - %{kilo}M
;      - %{megabytes}M
;      - %{mega}M
;  %n: pool name
;  %o: ouput header
;      it must be associated with embraces to specify the name of the header:
;      - %{Content-Type}o
;      - %{X-Powered-By}o
;      - %{Transfert-Encoding}o
;      - ....
;  %p: PID of the child that serviced the request
;  %P: PID of the parent of the child that serviced the request
;  %q: the query string
;  %Q: the '?' character if query string exists
;  %r: the request URI (without the query string, see %q and %Q)
;  %R: remote IP address
;  %s: status (response code)
;  %t: server time the request was received
;      it can accept a strftime(3) format:
;      %d/%b/%Y:%H:%M:%S %z (default)
;  %T: time the log has been written (the request has finished)
;      it can accept a strftime(3) format:
;      %d/%b/%Y:%H:%M:%S %z (default)
;  %u: remote user
;
; Default: "%R - %u %t \"%m %r\" %s"
;access.format = %R - %u %t "%m %r%Q%q" %s %f %{mili}d %{kilo}M %C%%

; The log file for slow requests
; Default Value: not set
; Note: slowlog is mandatory if request_slowlog_timeout is set
;slowlog = log/$pool.log.slow

; The timeout for serving a single request after which a PHP backtrace will be
; dumped to the 'slowlog' file. A value of '0s' means 'off'.
; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
; Default Value: 0
;request_slowlog_timeout = 0

; The timeout for serving a single request after which the worker process will
; be killed. This option should be used when the 'max_execution_time' ini option
; does not stop script execution for some reason. A value of '0' means 'off'.
; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
; Default Value: 0
;request_terminate_timeout = 0

; Set open file descriptor rlimit.
; Default Value: system defined value
;rlimit_files = 1024

; Set max core size rlimit.
; Possible Values: 'unlimited' or an integer greater or equal to 0
; Default Value: system defined value
;rlimit_core = 0

; Chroot to this directory at the start. This value must be defined as an
; absolute path. When this value is not set, chroot is not used.
; Note: you can prefix with '$prefix' to chroot to the pool prefix or one
; of its subdirectories. If the pool prefix is not set, the global prefix
; will be used instead.
; Note: chrooting is a great security feature and should be used whenever
;       possible. However, all PHP paths will be relative to the chroot
;       (error_log, sessions.save_path, ...).
; Default Value: not set
;chroot =

; Chdir to this directory at the start.
; Note: relative path can be used.
; Default Value: current directory or / when chroot
chdir = /

; Redirect worker stdout and stderr into main error log. If not set, stdout and
; stderr will be redirected to /dev/null according to FastCGI specs.
; Note: on highloaded environement, this can cause some delay in the page
; process time (several ms).
; Default Value: no
catch_workers_output = yes

; Limits the extensions of the main script FPM will allow to parse. This can
; prevent configuration mistakes on the web server side. You should only limit
; FPM to .php extensions to prevent malicious users to use other extensions to
; exectute php code.
; Note: set an empty value to allow all extensions.
; Default Value: .php
;security.limit_extensions = .php .php3 .php4 .php5 .php7

; Pass environment variables like LD_LIBRARY_PATH. All $VARIABLEs are taken from
; the current environment.
; Default Value: clean env
;env[HOSTNAME] = $HOSTNAME
env[PATH] = /srv/www/phpcs/scripts/:/usr/local/bin:/usr/bin:/bin
;env[TMP] = /tmp
;env[TMPDIR] = /tmp
;env[TEMP] = /tmp

; Additional php.ini defines, specific to this pool of workers. These settings
; overwrite the values previously defined in the php.ini. The directives are the
; same as the PHP SAPI:
;   php_value/php_flag             - you can set classic ini defines which can
;                                    be overwritten from PHP call 'ini_set'.
;   php_admin_value/php_admin_flag - these directives won't be overwritten by
;                                     PHP call 'ini_set'
; For php_*flag, valid values are on, off, 1, 0, true, false, yes or no.

; Defining 'extension' will load the corresponding shared extension from
; extension_dir. Defining 'disable_functions' or 'disable_classes' will not
; overwrite previously defined php.ini values, but will append the new value
; instead.

; Note: path INI options can be relative and will be expanded with the prefix
; (pool, global or /usr)

; Default Value: nothing is defined by default except the values in php.ini and
;                specified at startup with the -d argument
;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f www@my.domain.com
;php_flag[display_errors] = off
;php_admin_value[error_log] = /var/log/fpm-php.www.log
;php_admin_flag[log_errors] = on
;php_admin_value[memory_limit] = 32M
```
Kemudian, kita perlu membuat folder bernama php untuk mengakomodasi instalasi php di peran laravel. Dalam peran ini, kita hanya perlu membuat 2 direktori bernama penangan dan tugas.

Di direktori task, buat beberapa file bernama main.yml. Kemudian ketikkan script seperti di bawah ini.
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install php
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - gtkhash
    - crack-md5
    - git
    - curl
    - nginx
    - nginx-extras
    - php7.4
    - php7.4-fpm
    - php7.4-curl
    - php7.4-xml
    - php7.4-gd
    - php7.4-opcache
    - php7.4-mbstring
    - php7.4-zip
    - php7.4-json
    - php7.4-cli

- name: enable module php mbstring
  command: phpenmod mbstring
  notify:
    - restart php
```










buat directory codeigniter dengan roles directory ```sudo mkdir codeigniter``` untuk mengakomidasikan sebuah ansible script untuk menginstall codeigniter framework.

- Pada direktori codeigniter, kita membutuhkan 3 folder lagi untuk menampung tasks, handler dan script template yang akan digunakan dalam menginstall codeigniter.

```
sudo mkdir -p codeigniter/tasks
sudo mkdir -p codeigniter/handlers
sudo mkdir -p codeigniter/templates
```
- Pada Taks directory, buat beberapa file bernama main.yml. dengan script seperti di bawah ini :
```
---
- name: delete apt chache
  become: yes
  become_user: root
  become_method: su
  command: rm -vf /var/lib/apt/lists/*

- name: install requirement dpkg to install php5
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - ca-certificates
    - apt-transport-https
    - wget
    - curl
    - python-apt
    - software-properties-common
    - git

- name: Add key
  apt_key:
    url: https://packages.sury.org/php/apt.gpg
    state: present

- name: Add Php Repository
  apt_repository:
      repo: "deb https://packages.sury.org/php/ stretch main"
      state: present
      filename: php.list
      update_cache: true

- name: wget repository
  shell: wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg

- name: add repository
  shell: echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list

- name: apt update
  shell: apt update

- name: install nginx php5
  become: yes
  become_user: root
  become_method: su
  apt: name={{ item }} state=latest update_cache=true
  with_items:
    - nginx
    - nginx-extras
    - php5.6
    - php5.6-fpm
    - php5.6-common
    - php5.6-cli
    - php5.6-curl
    - php5.6-mbstring
    - php5.6-mysqlnd
    - php5.6-xml

- name: Git clone repo sas-ci
  become: yes
  git:
    repo: '{{ git_url }}'
    dest: "{{ destdir }}"

- name: Copy app.conf
  template:
    src=templates/app.conf
    dest=/etc/nginx/sites-available/{{ domain }}
  vars:
    servername: '{{ domain }}'

- name: Delete another nginx config
  become: yes
  become_user: root
  become_method: su
  command: rm -f /etc/nginx/sites-enabled/*

- name: Symlink app.conf
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
    
 - Pada directory handlers, buat beberapa file dengan nama main.yml. dengan script di bawah ini
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

 - Pada templates directory, buat beberapa file bernama app.conf . Kemudian ketikkan script seperti di bawah ini :
      ```
      server {
        listen 80;
        server_name {{ domain }};
        root {{ destdir }};
        index index.php;
        location / {
           try_files $uri $uri/ /index.php?$query_string;
        }
        location ~ \.php$ {
           fastcgi_pass unix:/run/php/php5.6-fpm.sock;  #Sesuaikan dengan versi PHP
           fastcgi_index index.php;
           fastcgi_param SCRIPT_FILENAME {{ destdir }}$fastcgi_script_name;
           include fastcgi_params;
        }
      }
      ```

  - buat sebuah db directory pada roles directory `sudo mkdir db` untuk akomodaikan ansible script untuk menginstall db framework.
    - pada direktori db, kita membutuhkan 3 folder lagi untuk menampung skrip tasks, direktori db, kita membutuhkan 3 folder lagi untuk menampung skrip tasks,
    handler dan template yang akan digunakan dalam instalasi db.

      ```
      sudo mkdir -p db/tasks
      sudo mkdir -p db/handlers
      sudo mkdir -p db/templates
      ```

    - pada tasks direcory, buat beberapa file dengan nama main.yml. kemudian ketikkan perintah berikut :

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


    - pada handlers directory, buat beberapa file dengan nama main.yml. dan tulis perintah seperti di bawah ini
    
      ```
      ---
      - name: restart mysql
        become: yes
        become_user: root
        become_method: su
        action: service name=mysql state=restarted
      ```

    - pada template directory, buat file bernama my.cnf. kemudian tulis perintah seperti berikut :
    - 
      ```
      #
      # These groups are read by MariaDB server.
      # Use it for options that only the server (but not clients) should see
      #
      # See the examples of server my.cnf files in /usr/share/mysql/
      #
      
      # this is read by the standalone daemon and embedded servers
      [server]
      
      # this is only for the mysqld standalone daemon
      [mysqld]
      
      #
      # * Basic Settings
      #
      user            = mysql
      pid-file        = /var/run/mysqld/mysqld.pid
      socket          = /var/run/mysqld/mysqld.sock
      port            = 3306
      basedir         = /usr
      datadir         = /var/lib/mysql
      tmpdir          = /tmp
      lc-messages-dir = /usr/share/mysql
      skip-external-locking
      
      # Instead of skip-networking the default is now to listen only on
      # localhost which is more compatible and is not less secure.
      bind-address            = 0.0.0.0
      
      #
      # * Fine Tuning
      #
      key_buffer_size         = 16M
      max_allowed_packet      = 16M
      thread_stack            = 192K
      thread_cache_size       = 8
      # This replaces the startup script and checks MyISAM tables if needed
      # the first time they are touched
      myisam_recover_options  = BACKUP
      #max_connections        = 100
      #table_cache            = 64
      #thread_concurrency     = 10
      
      #
      # * Query Cache Configuration
      #
      query_cache_limit       = 1M
      query_cache_size        = 16M
      
      #
      # * Logging and Replication
      #
      # Both location gets rotated by the cronjob.
      # Be aware that this log type is a performance killer.
      # As of 5.1 you can enable the log at runtime!
      #general_log_file        = /var/log/mysql/mysql.log
      #general_log             = 1
      #
      # Error log - should be very few entries.
      #
      log_error = /var/log/mysql/error.log
      #
      # Enable the slow query log to see queries with especially long duration
      #slow_query_log_file    = /var/log/mysql/mariadb-slow.log
      #long_query_time = 10
      #log_slow_rate_limit    = 1000
      #log_slow_verbosity     = query_plan
      #log-queries-not-using-indexes
      #
      # The following can be used as easy to replay backup logs or for replication.
      # note: if you are setting up a replication slave, see README.Debian about
      #       other settings you may need to change.
      #server-id              = 1
      #log_bin                        = /var/log/mysql/mysql-bin.log
      expire_logs_days        = 10
      max_binlog_size   = 100M
      #binlog_do_db           = include_database_name
      #binlog_ignore_db       = exclude_database_name
      
      #
      # * InnoDB
      #
      # InnoDB is enabled by default with a 10MB datafile in /var/lib/mysql/.
      # Read the manual for more InnoDB related options. There are many!
      
      #
      # * Security Features
      #
      # Read the manual, too, if you want chroot!
      # chroot = /var/lib/mysql/
      #
      # For generating SSL certificates you can use for example the GUI tool "tinyca".
      #
      # ssl-ca=/etc/mysql/cacert.pem
      # ssl-cert=/etc/mysql/server-cert.pem
      # ssl-key=/etc/mysql/server-key.pem
      #
      # Accept only connections using the latest and most secure TLS protocol version.
      # ..when MariaDB is compiled with OpenSSL:
      # ssl-cipher=TLSv1.2
      # ..when MariaDB is compiled with YaSSL (default in Debian):
      # ssl=on
      
      #
      # * Character sets
      #
      # MySQL/MariaDB default is Latin1, but in Debian we rather default to the full
      # utf8 4-byte character set. See also client.cnf
      #
      character-set-server  = utf8mb4
      collation-server      = utf8mb4_general_ci
      
      #
      # * Unix socket authentication plugin is built-in since 10.0.22-6
      #
      # Needed so the root database user can authenticate without a password but
      # only when running as the unix root user.
      #
      # Also available for other users if required.
      # See https://mariadb.com/kb/en/unix_socket-authentication-plugin/
      
      # this is only for embedded server
      [embedded]
      
      # This group is only read by MariaDB servers, not by MySQL.
      # If you use the same .cnf file for MySQL and MariaDB,
      # you can put MariaDB-only options here
      [mariadb]
      ```

    - lalu, kita membuat sebuah folder dengan nama pma untuk menginstall pma dengan roles pma. pada roles, kita harus membuat 3 directory dengan nama tasks, handlers dan      templates.

      - pada tasks directory, buat file bernama main.yml, lalu tulis perintah berikut :
     
        ```
        ---
        - name: delete apt chache
          become: yes
          become_user: root
          become_method: su
          command: rm -vf /var/lib/apt/lists/*
        
        - name: install requirement dpkg to install php5
          become: yes
          become_user: root
          become_method: su
          apt: name={{ item }} state=latest update_cache=true
          with_items:
            - ca-certificates
            - apt-transport-https
            - wget
            - curl
            - python-apt
            - software-properties-common
            - git
        
        - name: Add key
          apt_key:
            url: https://packages.sury.org/php/apt.gpg
            state: present
        
        - name: Add Php Repository
          apt_repository:
              repo: "deb https://packages.sury.org/php/ stretch main"
              state: present
              filename: php.list
              update_cache: true
        - name: wget repository php
          shell: wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
        
        - name: add repository php
          shell: echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list
        
        - name: wget repository
          shell: wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.tar.gz
        
        - name: tar phpmyadmin
          shell: tar -zxvf phpMyAdmin-5.1.1-all-languages.tar.gz
        
        - name: move phpmyadmin
          shell: mv phpMyAdmin-5.1.1-all-languages /usr/share/phpMyAdmin
        
        - name: apt update
          shell: apt update
        
        - name: install nginx phpmyadmin
          become: yes
          become_user: root
          become_method: su
          apt: name={{ item }} state=latest update_cache=true
          with_items:
            - curl
            - nginx
            - nginx-extras
            - php7.2-fpm
            - php7.2-mysqli
            - php7.2-xml
            - php-mbstring
            - php-zip
            - php-gd
            - php-json
            - php-curl
        - name: enable module php mbstring
          command: phpenmod mbstring
          notify:
            - restart php
        
        - name: Copy pma.local
          template:
            src=templates/pma.local
            dest=/etc/nginx/sites-available/{{ domain }}
          vars:
            servername: '{{ domain }}'
        
        - name: Symlink pma.local
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

      - pada  handlers directory, buat file bernama main.yml, lalu tulis perintah berikut :
       
        ```
        ---
        - name: stop apache2
          become: yes
          become_user: root
          become_method: su
          action: service name=apache2 state=stopped
        
        - name: restart nginx
          become: yes
          become_user: root
          become_method: su
          action: service name=nginx state=restarted
        
        - name: restart php
          become: yes
          become_user: root
          become_method: su
          action: service name=php7.2-fpm state=restarted
        ```

      - pada template directory, buat file bernama main.yml, lalu tulis perintah berikut :
        
        ```
        server {
            listen 80;
        
              server_name {{ domain }};
        
              root /usr/share/phpMyAdmin;
        
              index index.php;
        
              location / {
        
                   try_files $uri $uri/ @phpmyadmin;
        
              }
              location @phpmyadmin {
                      fastcgi_pass unix:/run/php/php7.2-fpm.sock;   #Sesuaikan dengan versi PHP
        
                      fastcgi_param SCRIPT_FILENAME /usr/share/phpMyAdmin/index.php;
        
                      include /etc/nginx/fastcgi_params;
        
                      fastcgi_param SCRIPT_NAME /index.php;
              }
              location ~ \.php$ {
        
                      fastcgi_pass unix:/run/php/php7.2-fpm.sock;  #Sesuaikan dengan versi PHP
        
                      fastcgi_index index.php;
        
                      fastcgi_param SCRIPT_FILENAME /usr/share/phpMyAdmin$fastcgi_script_name;
        
                      include fastcgi_params;
        
              }
          }
        ```

  - buat directory wordpress dengan roles directory `sudo mkdir wordpress` untuk ansible script untuk menginstall wordpress framework

    - Pada wordpress dircetory, kita buat 3 folder lagi untuk mengakomidasikan script tasks, handlers and templates yang akan digunakan dalam instalasi wordpress.

      ```
      sudo mkdir -p wordpress /tasks
      sudo mkdir -p wordpress /handlers
      sudo mkdir -p wordpress /templates
      ```

    - pada tasks directory, buat file bernama main.yml, lalu tulis perintah berikut :

      ```
      ---
      - name: delete apt chache
        become: yes
        become_user: root
        become_method: su
        command: rm -vf /var/lib/apt/lists/*
      
      - name: install requirement
        become: yes
        become_user: root
        become_method: su
        apt: name={{ item }} state=latest update_cache=true
        with_items:
          - nginx
          - nginx-extras
          - curl
          - wget
          - php7.4
          - php7.4-fpm
          - php7.4-curl
          - php7.4-xml
          - php7.4-gd
          - php7.4-opcache
          - php7.4-mbstring
          - php7.4-zip
          - php7.4-json
          - php7.4-cli
          - php7.4-mysqlnd
          - php7.4-xmlrpc
          - php7.4-curl
      
      - name: wget wordpress
        shell: wget -c http://wordpress.org/latest.tar.gz
      
      - name: tar latest.tar.gz
        shell: tar -xvzf latest.tar.gz
      
      - name: copy folder wordpress
        shell: cp -R wordpress /var/www/html/blog
      
      - name: chmod
        become: yes
        become_user: root
        become_method: su
        command: chmod 775 -R /var/www/html/blog/
      
      - name: copy .wp-config.conf
        template:
          src=templates/wp.conf
          dest=/var/www/html/blog/wp-config.php
      
      - name: copy php7.conf
        template:
          src=templates/php7.conf
          dest=/etc/php/7.4/fpm/pool.d/www.conf
      
      - name: copy wordpress.conf
        template:
          src=templates/wordpress.conf
          dest=/etc/nginx/sites-available/{{ domain }}
        vars:
          servername: '{{ domain }}'
      
      - name: Symlink wordpress.conf
        command: ln -sfn /etc/nginx/sites-available/{{ domain }} /etc/nginx/sites-enabled/{{ domain }}
        notify:
          - restart nginx
      
      - name: Write {{ domain }} to /etc/hosts
        lineinfile:
          dest: /etc/hosts
          regexp: '.*{{ domain }}$'
          line: "127.0.0.1 {{ domain }}"
          state: present
      
      - name: enable module php mbstring
        command: phpenmod mbstring
        notify:
          - restart php
      ```

    - Pada handler directory, buat file bernama main.yml, lalu tulis perintah berikut :

      ```
      ---
      - name: restart nginx
        become: yes
        become_user: root
        become_method: su
        action: service name=nginx state=restarted
      
      - name: restart php
        become: yes
        become_user: root
        become_method: su
        action: service name=php7.4-fpm state=restarted
      ```

    - Pada templates directory, kita akan membuat 3 file script seperti dibawah ini :

      - php7.conf

        ```
        ; Start a new pool named 'www'.
        ; the variable $pool can we used in any directive and will be replaced by the
        ; pool name ('www' here)
        [www]
        
        ; Per pool prefix
        ; It only applies on the following directives:
        ; - 'slowlog'
        ; - 'listen' (unixsocket)
        ; - 'chroot'
        ; - 'chdir'
        ; - 'php_values'
        ; - 'php_admin_values'
        ; When not set, the global prefix (or /usr) applies instead.
        ; Note: This directive can also be relative to the global prefix.
        ; Default Value: none
        ;prefix = /path/to/pools/$pool
        
        ; Unix user/group of processes
        ; Note: The user is mandatory. If the group is not set, the default user's group
        ;       will be used.
        user = www-data
        group = www-data
        
        ; The address on which to accept FastCGI requests.
        ; Valid syntaxes are:
        ;   'ip.add.re.ss:port'    - to listen on a TCP socket to a specific address on
        ;                            a specific port;
        ;   'port'                 - to listen on a TCP socket to all addresses on a
        ;                            specific port;
        ;   '/path/to/unix/socket' - to listen on a unix socket.
        ; Note: This value is mandatory.
        listen = 127.0.0.1:9001
        
        ; Set listen(2) backlog. A value of '-1' means unlimited.
        ; Default Value: 128 (-1 on FreeBSD and OpenBSD)
        ;listen.backlog = -1
        
        ; Set permissions for unix socket, if one is used. In Linux, read/write
        ; permissions must be set in order to allow connections from a web server. Many
        ; BSD-derived systems allow connections regardless of permissions.
        ; Default Values: user and group are set as the running user
        ;                 mode is set to 0666
        listen.owner = www-data
        listen.group = www-data
        listen.mode = 0666
        
        ; List of ipv4 addresses of FastCGI clients which are allowed to connect.
        ; Equivalent to the FCGI_WEB_SERVER_ADDRS environment variable in the original
        ; PHP FCGI (5.2.2+). Makes sense only with a tcp listening socket. Each address
        ; must be separated by a comma. If this value is left blank, connections will be
        ; accepted from any ip address.
        ; Default Value: any
        ;listen.allowed_clients = 127.0.0.1
        
        ; Choose how the process manager will control the number of child processes.
        ; Possible Values:
        ;   static  - a fixed number (pm.max_children) of child processes;
        ;   dynamic - the number of child processes are set dynamically based on the
        ;             following directives. With this process management, there will be
        ;             always at least 1 children.
        ;             pm.max_children      - the maximum number of children that can
        ;                                    be alive at the same time.
        ;             pm.start_servers     - the number of children created on startup.
        ;             pm.min_spare_servers - the minimum number of children in 'idle'
        ;                                    state (waiting to process). If the number
        ;                                    of 'idle' processes is less than this
        ;                                    number then some children will be created.
        ;             pm.max_spare_servers - the maximum number of children in 'idle'
        ;                                    state (waiting to process). If the number
        ;                                    of 'idle' processes is greater than this
        ;                                    number then some children will be killed.
        ;  ondemand - no children are created at startup. Children will be forked when
        ;             new requests will connect. The following parameter are used:
        ;             pm.max_children           - the maximum number of children that
        ;                                         can be alive at the same time.
        ;             pm.process_idle_timeout   - The number of seconds after which
        ;                                         an idle process will be killed.
        ; Note: This value is mandatory.
        pm = dynamic
        
        ; The number of child processes to be created when pm is set to 'static' and the
        ; maximum number of child processes when pm is set to 'dynamic' or 'ondemand'.
        ; This value sets the limit on the number of simultaneous requests that will be
        ; served. Equivalent to the ApacheMaxClients directive with mpm_prefork.
        ; Equivalent to the PHP_FCGI_CHILDREN environment variable in the original PHP
        ; CGI. The below defaults are based on a server without much resources. Don't
        ; forget to tweak pm.* to fit your needs.
        ; Note: Used when pm is set to 'static', 'dynamic' or 'ondemand'
        ; Note: This value is mandatory.
        pm.max_children = 5
        
        ; The number of child processes created on startup.
        ; Note: Used only when pm is set to 'dynamic'
        ; Default Value: min_spare_servers + (max_spare_servers - min_spare_servers) / 2
        pm.start_servers = 2
        
        ; The desired minimum number of idle server processes.
        ; Note: Used only when pm is set to 'dynamic'
        ; Note: Mandatory when pm is set to 'dynamic'
        pm.min_spare_servers = 1
        
        ; The desired maximum number of idle server processes.
        ; Note: Used only when pm is set to 'dynamic'
        ; Note: Mandatory when pm is set to 'dynamic'
        pm.max_spare_servers = 3
        
        ; The number of seconds after which an idle process will be killed.
        ; Note: Used only when pm is set to 'ondemand'
        ; Default Value: 10s
        ;pm.process_idle_timeout = 10s;
        
        ; The number of requests each child process should execute before respawning.
        ; This can be useful to work around memory leaks in 3rd party libraries. For
        ; endless request processing specify '0'. Equivalent to PHP_FCGI_MAX_REQUESTS.
        ; Default Value: 0
        pm.max_requests = 100
        
        ; The URI to view the FPM status page. If this value is not set, no URI will be
        ; recognized as a status page. It shows the following informations:
        ;   pool                 - the name of the pool;
        ;   process manager      - static, dynamic or ondemand;
        ;   start time           - the date and time FPM has started;
        ;   start since          - number of seconds since FPM has started;
        ;   accepted conn        - the number of request accepted by the pool;
        ;   listen queue         - the number of request in the queue of pending
        ;                          connections (see backlog in listen(2));
        ;   max listen queue     - the maximum number of requests in the queue
        ;                          of pending connections since FPM has started;
        ;   listen queue len     - the size of the socket queue of pending connections;
        ;   idle processes       - the number of idle processes;
        ;   active processes     - the number of active processes;
        ;   total processes      - the number of idle + active processes;
        ;   max active processes - the maximum number of active processes since FPM
        ;                          has started;
        ;   max children reached - number of times, the process limit has been reached,
        ;                          when pm tries to start more children (works only for
        ;                          pm 'dynamic' and 'ondemand');
        ; Value are updated in real time.
        ; Example output:
        ;   pool:                 www
        ;   process manager:      static
        ;   start time:           01/Jul/2011:17:53:49 +0200
        ;   start since:          62636
        ;   accepted conn:        190460
        ;   listen queue:         0
        ;   max listen queue:     1
        ;   listen queue len:     42
        ;   idle processes:       4
        ;   active processes:     11
        ;   total processes:      15
        ;   max active processes: 12
        ;   max children reached: 0
        ;
        ; By default the status page output is formatted as text/plain. Passing either
        ; 'html', 'xml' or 'json' in the query string will return the corresponding
        ; output syntax. Example:
        ;   http://www.foo.bar/status
        ;   http://www.foo.bar/status?json
        ;   http://www.foo.bar/status?html
        ;   http://www.foo.bar/status?xml
        ;
        ; By default the status page only outputs short status. Passing 'full' in the
        ; query string will also return status for each pool process.
        ; Example:
        ;   http://www.foo.bar/status?full
        ;   http://www.foo.bar/status?json&full
        ;   http://www.foo.bar/status?html&full
        ;   http://www.foo.bar/status?xml&full
        ; The Full status returns for each process:
        ;   pid                  - the PID of the process;
        ;   state                - the state of the process (Idle, Running, ...);
        ;   start time           - the date and time the process has started;
        ;   start since          - the number of seconds since the process has started;
        ;   requests             - the number of requests the process has served;
        ;   request duration     - the duration in µs of the requests;
        ;   request method       - the request method (GET, POST, ...);
        ;   request URI          - the request URI with the query string;
        ;   content length       - the content length of the request (only with POST);
        ;   user                 - the user (PHP_AUTH_USER) (or '-' if not set);
        ;   script               - the main script called (or '-' if not set);
        ;   last request cpu     - the %cpu the last request consumed
        ;                          it's always 0 if the process is not in Idle state
        ;                          because CPU calculation is done when the request
        ;                          processing has terminated;
        ;   last request memory  - the max amount of memory the last request consumed
        ;                          it's always 0 if the process is not in Idle state
        ;                          because memory calculation is done when the request
        ;                          processing has terminated;
        ; If the process is in Idle state, then informations are related to the
        ; last request the process has served. Otherwise informations are related to
        ; the current request being served.
        ; Example output:
        ;   ************************
        ;   pid:                  31330
        ;   state:                Running
        ;   start time:           01/Jul/2011:17:53:49 +0200
        ;   start since:          63087
        ;   requests:             12808
        ;   request duration:     1250261
        ;   request method:       GET
        ;   request URI:          /test_mem.php?N=10000
        ;   content length:       0
        ;   user:                 -
        ;   script:               /home/fat/web/docs/php/test_mem.php
        ;   last request cpu:     0.00
        ;   last request memory:  0
        ;
        ; Note: There is a real-time FPM status monitoring sample web page available
        ;       It's available in: ${prefix}/share/fpm/status.html
        ;
        ; Note: The value must start with a leading slash (/). The value can be
        ;       anything, but it may not be a good idea to use the .php extension or it
        ;       may conflict with a real PHP file.
        ; Default Value: not set
        pm.status_path = /php-status
        
        ; The ping URI to call the monitoring page of FPM. If this value is not set, no
        ; URI will be recognized as a ping page. This could be used to test from outside
        ; that FPM is alive and responding, or to
        ; - create a graph of FPM availability (rrd or such);
        ; - remove a server from a group if it is not responding (load balancing);
        ; - trigger alerts for the operating team (24/7).
        ; Note: The value must start with a leading slash (/). The value can be
        ;       anything, but it may not be a good idea to use the .php extension or it
        ;       may conflict with a real PHP file.
        ; Default Value: not set
        ;ping.path = /ping
        
        ; This directive may be used to customize the response of a ping request. The
        ; response is formatted as text/plain with a 200 response code.
        ; Default Value: pong
        ;ping.response = pong
        
        ; The access log file
        ; Default: not set
        ;access.log = log/$pool.access.log
        
        ; The access log format.
        ; The following syntax is allowed
        ;  %%: the '%' character
        ;  %C: %CPU used by the request
        ;      it can accept the following format:
        ;      - %{user}C for user CPU only
        ;      - %{system}C for system CPU only
        ;      - %{total}C  for user + system CPU (default)
        ;  %d: time taken to serve the request
        ;      it can accept the following format:
        ;      - %{seconds}d (default)
        ;      - %{miliseconds}d
        ;      - %{mili}d
        ;      - %{microseconds}d
        ;      - %{micro}d
        ;  %e: an environment variable (same as $_ENV or $_SERVER)
        ;      it must be associated with embraces to specify the name of the env
        ;      variable. Some exemples:
        ;      - server specifics like: %{REQUEST_METHOD}e or %{SERVER_PROTOCOL}e
        ;      - HTTP headers like: %{HTTP_HOST}e or %{HTTP_USER_AGENT}e
        ;  %f: script filename
        ;  %l: content-length of the request (for POST request only)
        ;  %m: request method
        ;  %M: peak of memory allocated by PHP
        ;      it can accept the following format:
        ;      - %{bytes}M (default)
        ;      - %{kilobytes}M
        ;      - %{kilo}M
        ;      - %{megabytes}M
        ;      - %{mega}M
        ;  %n: pool name
        ;  %o: ouput header
        ;      it must be associated with embraces to specify the name of the header:
        ;      - %{Content-Type}o
        ;      - %{X-Powered-By}o
        ;      - %{Transfert-Encoding}o
        ;      - ....
        ;  %p: PID of the child that serviced the request
        ;  %P: PID of the parent of the child that serviced the request
        ;  %q: the query string
        ;  %Q: the '?' character if query string exists
        ;  %r: the request URI (without the query string, see %q and %Q)
        ;  %R: remote IP address
        ;  %s: status (response code)
        ;  %t: server time the request was received
        ;      it can accept a strftime(3) format:
        ;      %d/%b/%Y:%H:%M:%S %z (default)
        ;  %T: time the log has been written (the request has finished)
        ;      it can accept a strftime(3) format:
        ;      %d/%b/%Y:%H:%M:%S %z (default)
        ;  %u: remote user
        ;
        ; Default: "%R - %u %t \"%m %r\" %s"
        ;access.format = %R - %u %t "%m %r%Q%q" %s %f %{mili}d %{kilo}M %C%%
        
        ; The log file for slow requests
        ; Default Value: not set
        ; Note: slowlog is mandatory if request_slowlog_timeout is set
        ;slowlog = log/$pool.log.slow
        
        ; The timeout for serving a single request after which a PHP backtrace will be
        ; dumped to the 'slowlog' file. A value of '0s' means 'off'.
        ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
        ; Default Value: 0
        ;request_slowlog_timeout = 0
        
        ; The timeout for serving a single request after which the worker process will
        ; be killed. This option should be used when the 'max_execution_time' ini option
        ; does not stop script execution for some reason. A value of '0' means 'off'.
        ; Available units: s(econds)(default), m(inutes), h(ours), or d(ays)
        ; Default Value: 0
        ;request_terminate_timeout = 0
        
        ; Set open file descriptor rlimit.
        ; Default Value: system defined value
        ;rlimit_files = 1024
        
        ; Set max core size rlimit.
        ; Possible Values: 'unlimited' or an integer greater or equal to 0
        ; Default Value: system defined value
        ;rlimit_core = 0
        
        ; Chroot to this directory at the start. This value must be defined as an
        ; absolute path. When this value is not set, chroot is not used.
        ; Note: you can prefix with '$prefix' to chroot to the pool prefix or one
        ; of its subdirectories. If the pool prefix is not set, the global prefix
        ; will be used instead.
        ; Note: chrooting is a great security feature and should be used whenever
        ;       possible. However, all PHP paths will be relative to the chroot
        ;       (error_log, sessions.save_path, ...).
        ; Default Value: not set
        ;chroot =
        
        ; Chdir to this directory at the start.
        ; Note: relative path can be used.
        ; Default Value: current directory or / when chroot
        chdir = /
        
        ; Redirect worker stdout and stderr into main error log. If not set, stdout and
        ; stderr will be redirected to /dev/null according to FastCGI specs.
        ; Note: on highloaded environement, this can cause some delay in the page
        ; process time (several ms).
        ; Default Value: no
        catch_workers_output = yes
        
        ; Limits the extensions of the main script FPM will allow to parse. This can
        ; prevent configuration mistakes on the web server side. You should only limit
        ; FPM to .php extensions to prevent malicious users to use other extensions to
        ; exectute php code.
        ; Note: set an empty value to allow all extensions.
        ; Default Value: .php
        ;security.limit_extensions = .php .php3 .php4 .php5 .php7
        
        ; Pass environment variables like LD_LIBRARY_PATH. All $VARIABLEs are taken from
        ; the current environment.
        ; Default Value: clean env
        ;env[HOSTNAME] = $HOSTNAME
        env[PATH] = /srv/www/phpcs/scripts/:/usr/local/bin:/usr/bin:/bin
        ;env[TMP] = /tmp
        ;env[TMPDIR] = /tmp
        ;env[TEMP] = /tmp
        
        ; Additional php.ini defines, specific to this pool of workers. These settings
        ; overwrite the values previously defined in the php.ini. The directives are the
        ; same as the PHP SAPI:
        ;   php_value/php_flag             - you can set classic ini defines which can
        ;                                    be overwritten from PHP call 'ini_set'.
        ;   php_admin_value/php_admin_flag - these directives won't be overwritten by
        ;                                     PHP call 'ini_set'
        ; For php_*flag, valid values are on, off, 1, 0, true, false, yes or no.
        
        ; Defining 'extension' will load the corresponding shared extension from
        ; extension_dir. Defining 'disable_functions' or 'disable_classes' will not
        ; overwrite previously defined php.ini values, but will append the new value
        ; instead.
        
        ; Note: path INI options can be relative and will be expanded with the prefix
        ; (pool, global or /usr)
        
        ; Default Value: nothing is defined by default except the values in php.ini and
        ;                specified at startup with the -d argument
        ;php_admin_value[sendmail_path] = /usr/sbin/sendmail -t -i -f www@my.domain.com
        ;php_flag[display_errors] = off
        ;php_admin_value[error_log] = /var/log/fpm-php.www.log
        ;php_admin_flag[log_errors] = on
        ;php_admin_value[memory_limit] = 32M
        ```

      - wordpress.conf

        ```
        server {
             listen 80;
             listen [::]:80;
        
             # Log files for Debugging
             access_log /var/log/nginx/wordpress-access.log;
             error_log /var/log/nginx/wordpress-error.log;
        
             # Webroot Directory for Laravel project
             root /var/www/html/wordpress;
             index index.php index.html index.htm;
        
             # Your Domain Name
             server_name {{ domain }};
        
             location / {
                     try_files $uri $uri/ /index.php?$query_string;
             }
        
             # PHP-FPM Configuration Nginx
             location ~ \.php$ {
                     try_files $uri =404;
                     fastcgi_split_path_info ^(.+\.php)(/.+)$;
                     fastcgi_pass 127.0.0.1:9001;
                     fastcgi_index index.php;
                     fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                     include fastcgi_params;
             }
        }
        ```

      - wp.conf 

        ```
        <?php
        /**
         * The base configuration for WordPress
         *
         * The wp-config.php creation script uses this file during the installation.
         * You don't have to use the web site, you can copy this file to "wp-config.php"
         * and fill in the values.
         *
         * This file contains the following configurations:
         *
         * * MySQL settings
         * * Secret keys
         * * Database table prefix
         * * ABSPATH
         *
         * @link https://wordpress.org/support/article/editing-wp-config-php/
         *
         * @package WordPress
         */
        
        define( 'WP_HOME', 'http://news.kelompok12.fpsas' );
        define( 'WP_SITEURL', 'http://news.kelompok12.fpsas' );
        
        // ** MySQL settings - You can get this info from your web host ** //
        /** The name of the database for WordPress */
        define( 'DB_NAME', 'blog' );
        
        /** MySQL database username */
        define( 'DB_USER', 'admin' );
        
        /** MySQL database password */
        define( 'DB_PASSWORD', 'admin' );
        
        /** MySQL hostname */
        define( 'DB_HOST', '10.0.3.100:3306' );
        /** Database charset to use in creating database tables. */
        define( 'DB_CHARSET', 'utf8' );
        
        /** The database collate type. Don't change this if in doubt. */
        define( 'DB_COLLATE', '' );
        
        /**#@+
         * Authentication unique keys and salts.
         *
         * Change these to different unique phrases! You can generate these using
         * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
         *
         * You can change these at any point in time to invalidate all existing cookies.
         * This will force all users to have to log in again.
         *
         * @since 2.6.0
         */
        define( 'AUTH_KEY',         'put your unique phrase here' );
        define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
        define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
        define( 'NONCE_KEY',        'put your unique phrase here' );
        define( 'AUTH_SALT',        'put your unique phrase here' );
        define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
        define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
        define( 'NONCE_SALT',       'put your unique phrase here' );
        
        /**#@-*/
        
        /**
         * WordPress database table prefix.
         *
         * You can have multiple installations in one database if you give each
         * a unique prefix. Only numbers, letters, and underscores please!
         */
        $table_prefix = 'wp_';
        /**
         * For developers: WordPress debugging mode.
         *
         * Change this to true to enable the display of notices during development.
         * It is strongly recommended that plugin and theme developers use WP_DEBUG
         * in their development environments.
         *
         * For information on other constants that can be used for debugging,
         * visit the documentation.
         *
         * @link https://wordpress.org/support/article/debugging-in-wordpress/
         */
        define( 'WP_DEBUG', false );
        
        /* Add any custom values between this line and the "stop editing" line. */
        
        
        
        /* That's all, stop editing! Happy publishing. */
        
        /** Absolute path to the WordPress directory. */
        if ( ! defined( 'ABSPATH' ) ) {
                define( 'ABSPATH', __DIR__ . '/' );
        }
        
        /** Sets up WordPress vars and included files. */
        require_once ABSPATH . 'wp-settings.php';
        ```

  - create a yii directory in the roles directory `sudo mkdir yii` to accommodate an ansible script to install yii framewrok.  

    - In the yii directory, we need 3 folders again to accommodate tasks, handlers and templates scripts that will be use in the yii installation.

      ```
      sudo mkdir -p yii /tasks
      sudo mkdir -p yii /handlers
      sudo mkdir -p yii /templates
      ```

    - Pada tasks directory, buat file bernama main.yml, lalu tulis perintah berikut :

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
          path: /var/www/html/basic
          state: absent
      
      - name: composer create-project
        shell: /usr/local/bin/composer create-project yiisoft/yii2-app-basic  /var/www/html/basic --prefer-dist --no-interaction
      
      - name: chmod
        become: yes
        become_user: root
        become_method: su
        command: chmod +x /usr/local/bin/composer
      
      - name: composer
        shell: cd /var/www/html/basic; /usr/local/bin/composer install  --no-interaction
      
      
      - name: Copy yii.conf
        template:
          src=templates/yii.conf
          dest=/etc/nginx/sites-available/{{ domain }}
        vars:
          servername: '{{ domain }}'
      
      - name: Symlink yii.conf
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

    - In the handlers directory, buat file bernama main.yml, lalu tulis perintah berikut :

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

    -  Pada templates directory, buat file bernama yii.conf, lalu tulis perintah berikut :

      ```
      server {
          set $host_path "/var/www/html/basic";
          #access_log  /www/testproject/log/access.log  main;
      
          server_name  {{ domain }};
          root   $host_path/web;
          set $yii_bootstrap "index.php";
      
          charset utf-8;
      
          location / {
              index  index.html $yii_bootstrap;
              try_files $uri $uri/ /$yii_bootstrap?$args;
          }
      
          location ~ ^/(protected|framework|themes/\w+/views) {
              deny  all;
          }
      
          #avoid processing of calls to unexisting static files by yii
          location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
              try_files $uri =404;
          }
      
          # pass the PHP scripts to FastCGI server listening on UNIX socket
          location ~ \.php {
              fastcgi_split_path_info  ^(.+\.php)(.*)$;
      
              #let yii catch the calls to unexising PHP files
              set $fsn /$yii_bootstrap;
              if (-f $document_root$fastcgi_script_name){
                  set $fsn $fastcgi_script_name;
              }
             fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
              include fastcgi_params;
              fastcgi_param  SCRIPT_FILENAME  $document_root$fsn;
      
             #PATH_INFO and PATH_TRANSLATED can be omitted, but RFC 3875 specifies them for CGI
              fastcgi_param  PATH_INFO        $fastcgi_path_info;
              fastcgi_param  PATH_TRANSLATED  $document_root$fsn;
          }
      
          # prevent nginx from serving dotfiles (.htaccess, .svn, .git, etc.)
          location ~ /\. {
              deny all;
              access_log off;
              log_not_found off;
          }
      }
      ```

- Setelah itu, kita perlu membuat beberapa file untuk menjalankan konfigurasi yang telah kita buat di direktori roles.
  
  - Buat file dengan nama hosts `nano hosts` and tulis perintah di bawah ini:

    ```
    [laravel]
    laravel ansible_host=lxc_laravel.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_1L ansible_host=lxc_php7_1L.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_2L ansible_host=lxc_php7_2L.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_4L ansible_host=lxc_php7_4L.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_6L ansible_host=lxc_php7_6L.dev ansible_ssh_user=root ansible_become_pass=akbar
    
    [codeigniter]
    codeigniter ansible_host=lxc_codeigniter.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php5_1 ansible_host=lxc_php5_1.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php5_2 ansible_host=lxc_php5_2.dev ansible_ssh_user=root ansible_become_pass=akbar
    
    [wordpress]
    wordpress ansible_host=lxc_wordpress.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_2W ansible_host=lxc_php7_2W.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_3W ansible_host=lxc_php7_3W.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_4W ansible_host=lxc_php7_4W.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_5W ansible_host=lxc_php7_5W.dev ansible_ssh_user=root ansible_become_pass=akbar
    
    [yii]
    yii ansible_host=lxc_yii.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_1Y ansible_host=lxc_php7_1Y.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_2Y ansible_host=lxc_php7_2Y.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_4Y ansible_host=lxc_php7_4Y.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_5Y ansible_host=lxc_php7_5Y.dev ansible_ssh_user=root ansible_become_pass=akbar
    lxc_php7_6Y ansible_host=lxc_php7_6Y.dev ansible_ssh_user=root ansible_become_pass=akbar
    
    [database]
    lxc_mariadb ansible_host=lxc_mariadb.dev ansible_ssh_user=root ansible_become_pass=akbar
    ```

  - Buat file dengan nama install-codeigniter.yml `nano install-codeigniter.yml` dan tulis perintah seperti di bawah ini:

    ```
    ---
    - hosts: codeigniter
      vars:
        git_url: 'https://github.com/aldonesia/sas-ci'
        destdir: '/var/www/html/ci'
        domain: 'lxc_codeigniter.dev'
      roles:
        - app
    
    - hosts: lxc_php5_1
      vars:
        git_url: 'https://github.com/aldonesia/sas-ci'
        destdir: '/var/www/html/ci'
        domain: 'lxc_php5_1.dev'
      roles:
        - app
    
    - hosts: lxc_php5_2
      vars:
        git_url: 'https://github.com/aldonesia/sas-ci'
        destdir: '/var/www/html/ci'
        domain: 'lxc_php5_2.dev'
      roles:
        - app
    ```

  - buat file dengan nama install-laravel.yml `nano install-laravel.yml` dan tulis perintah seperti di bawah ini:

    ```
    ---
    - hosts: laravel
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_laravel.dev'
      roles:
        - php
        - laravel
    
    - hosts: lxc_php7_1L
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_1L.dev'
      roles:
        - php
        - laravel
    
    - hosts: lxc_php7_2L
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_2L.dev'
      roles:
        - php
        - laravel
    
    - hosts: lxc_php7_4L
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_4L.dev'
      roles:
        - php
        - laravel
    
    - hosts: lxc_php7_6L
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_6L.dev'
      roles:
        - php
        - laravel
    ```

  - Create file named install-mariadb.yml `nano install-mariadb.yml` dan tulis perintah seperti di bawah ini:

    ```
    - hosts: database
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_mariadb.dev'
      roles:
        - db
        - pma
    ```

  - Buat file nama install-wordpress.yml `nano install-wordpress.yml` dan tulis perintah seperti di bawah ini:

    ```
    ---
    - hosts: wordpress
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_wordpress.dev'
      roles:
        - wordpress
    
    - hosts: lxc_php7_2W
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_2W.dev'
      roles:
        - wordpress
    
    - hosts: lxc_php7_3W
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_3W.dev'
      roles:
        - wordpress
    
    - hosts: lxc_php7_4W
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_4W.dev'
      roles:
        - wordpress
    
    - hosts: lxc_php7_5W
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_5W.dev'
      roles:
        - wordpress
    ```

  - Buat file dengan nama install-yii.yml `nano install-yii.yml` dan tulis perintah seperti di bawah ini:

    ```
    ---
    - hosts: yii
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_yii.dev'
      roles:
        - php
        - yii
    
    - hosts: lxc_php7_1Y
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_1Y.dev'
      roles:
        - php
        - yii
    
    - hosts: lxc_php7_2Y
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_2Y.dev'
      roles:
        - php
        - yii
    
    - hosts: lxc_php7_4Y
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_4Y.dev'
      roles:
        - php
        - yii
    
    - hosts: lxc_php7_5Y
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_5Y.dev'
      roles:
        - php
        - yii
    
    - hosts: lxc_php7_6Y
      vars:
        username: 'admin'
        password: 'admin'
        domain: 'lxc_php7_6Y.dev'
      roles:
        - php
        - yii
    ```



- Kemudian untuk yang terakhir, kita perlu mengkonfigurasi pengaturan nginx. Masuk ke situs-tersedia `cd /etc/nginx/sites-available` dan buat file bernama kelompok02.fpsas. Masukkan file menggunakan `sudo nano kelompok12.fpsas` dan ketik script seperti di bawah ini untuk pengaturan konfigurasi nginx include load balancer.

  ```
  upstream laravel {
          least_conn;
          server lxc_php7_1L.dev;
          server lxc_php7_2L.dev;
          server lxc_php7_4L.dev;
          server lxc_php7_6L.dev;
  }
  
  upstream wordpress {
          ip_hash;
          server lxc_php7_2W.dev;
          server lxc_php7_3W.dev;
          server lxc_php7_4W.dev;
          server lxc_php7_5W.dev;
  }
  
  upstream yii {
          server lxc_php7_1Y.dev weight=3;
          server lxc_php7_2Y.dev weight=2;
          server lxc_php7_4Y.dev weight=4;
          server lxc_php7_5Y.dev weight=1;
          server lxc_php7_6Y.dev weight=6;
  }
  
  upstream codeigniter {
          server lxc_php5_1.dev;
          server lxc_php5_2.dev;
  }
  
  server {
          listen 80;
          listen [::]:80;
  
          server_name kelompok12.fpsas;
  
          root /var/www/html;
          index index.html;
  
          location /product {
                  rewrite /product/?(.*)$ /$1 break;
                  proxy_pass http://yii;
          }
  
          location /app {
                  rewrite /app/?(.*)$ /$1 break;
                  proxy_pass http://codeigniter;
          }
  
          location /phpmyadmin{
                  rewrite /phpmyadmin/?(.*)$ /$1 break;
                  proxy_pass http://lxc_mariadb.dev;
          }
  
          location / {
                  #rewrite /?(.*)$ /$1 break;
                  proxy_pass http://laravel;
          }
  }
  
  server {
          listen 80;
          listen [::]:80;
  
          server_name news.kelompok12.fpsas;
  
          root /var/www/html;
          index index.html;
  
          location / {
                  #rewrite /?(.*)$ /$1 break;
                  proxy_pass http://wordpress;
          }
  }
  ```
  
- Cek script dan restart the nginx.

  ```
  sudo nginx -t
  sudo nginx -s reload
  sudo service nginx restart
  ```

- Sekarang, kita dapat mengecek konfigurasi kita di browser dengan ketikkan nama dari domain kita (http://kelompok02.fpsas).

- laravel (kelompok02.fpsas/)
