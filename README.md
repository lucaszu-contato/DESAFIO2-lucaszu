# DESAFIO2-lucaszu
WORDPRESS COM ANSIBLE 

---
- hosts: teste
  user: centos
  become: yes
  tasks:

    - name: Atualizando pacotes
      shell: yum -y update
         
    - name:  Epel
      yum: name=epel-release state=present
         
    - name: wget
      yum: name=wget state=present
         
    - name: nginx
      yum: name=nginx state=present
                          
    - name: mariadb-server
      yum: name=mariadb-server state=present
         
    - name: php
      yum: name=php-fpm state=present          
         
    - name: php-common
      yum: name=php-common state=present
         
    - name: php-mysql
      yum: name=php-mysql state=present
         
    - name: php-gd
      yum: name=php-gd state=present
         
    - name: php-xml
      yum: name=php-xml state=present
         
    - name: php-mbstring
      yum: name=php-mbstring state=present
         
    - name: php-mcrypt
      yum: name=php-mcrypt state=present
         
    - name: Reiniciando Daeler
      service: name=daemon_reload
         
    - name: Startando Nginx
      shell: systemctl start nginx
         
    - name: Startando Mysqld
      shell: systemctl start mariadb
          
    - name: "Excluindo pasta se existir Criando pasta PHP e dando permissao"
      shell: rm -rf /var/run/php && mkdir /var/run/php && chmod 755 /var/run/php
         
    - name: Start php-fpm
      shell: systemctl start php-fpm

    - name: "Atualizar sistema"
      shell: sudo yum -y update
    
    - name: "Instalar pre requisitos [NGINX]"
      yum:
        name: epel-release
        state: latest
    
    - name: "pacotes padrao"
      yum:
        name: yum-utils
        state: latest
    
    # INSTALACAO NGINX
    - name: "NGINX"
      yum:
        name: nginx
        state: latest
    # INSTALACAO WGET
    - name: "WGET"
      yum:
        name: wget
        state: latest
    # INSTALACAO MYSQL
    - name: "MYSQL"
      yum:
        name: mariadb-server
        state: latest
 
    - name: "Instalando remi 7"
      yum:
        name: http://rpms.remirepo.net/enterprise/remi-release-7.rpm
        state: latest
    
    - name: "Instalando remi 7"
      shell: sudo yum-config-manager --enable remi-php73
    
    - name: "php7.3"
      yum:
        name: php
        state: latest

    - name: "Instalando [PHP]"
      yum:
        name: php-fpm
        state: latest
    - name: "Instalando [php-common]"
      yum:
        name: php-common
        state: latest
    - name: "Instalando [php-mysql]"
      yum:
        name: php-mysql
        state: latest
    - name: "Instalando [php-gd]"
      yum:
        name: php-gd
        state: latest
    - name: "Instalando [php-xml]"
      yum:
        name: php-xml
        state: latest
    - name: "Instalando [php-mbstring]"
      yum:
        name: php-mbstring
        state: latest
    - name: "Instalando [php-mcrypt]"
      yum:
        name: php-mcrypt
        state: latest
    - name: Reiniciando Daeler
      service:
        name: daemon_reload
    - name: "Iniciando [Nginx]"
      shell: sudo systemctl start nginx
    - name: "Iniciando [Mysqld]"
      shell: sudo systemctl start mariadb
    - name: "Iniciando [php-fpm]"
      shell: sudo systemctl start php-fpm
    - name: "Criando diretorio caso não exista"
      file:
        path: /var/www/html/blog
        state: directory
        mode: '777'
        recurse: yes
    - name: "Adicionar ao grupo [Wordpress]"
      group:
        name: wordpress
    - name: "Adicionar usuario [Wordpress]"
      user: name=wordpress group=wordpress home=/var/www/html/blog/wordpress/
    - name: "Realizando download [Wordpress]"
      get_url:
        url: https://wordpress.org/latest.tar.gz
        dest: /var/www/html/blog
    - name: "Extrair arquivo [Wordpress]"
      unarchive:
        src: /var/www/html/blog/wordpress-5.3.tar.gz
        dest: /var/www/html/blog
        remote_src: yes
    - name: "Copiar [WordPress config file]"
      template:
        src: config/wp-config.php
        dest: "/var/www/html/blog/wordpress"
    - name: "Alterar resposavel [WordPress installation]"
      file:
        path: "/var/www/html/blog/wordpress/"
        owner: wordpress
        group: wordpress
        state: directory
        recurse: yes
        setype: httpd_sys_content_t
    - name: "Iniciar [php-fpm Service]"
      service:
        name: php-fpm
        state: started
        enabled: yes
    # #####
    # MODULO MYSQL
    # https://docs.ansible.com/ansible/latest/modules/mysql_user_module.html
    # #####
    - name: "Instalando [python-pip]"
      yum:
        name: python-pip
        state: latest
    - name: "Make sure pymysql is present"
      become: true # needed if the other tasks are not played as root
      pip:
        name: pymysql
        state: present
    - name: "Criar [WordPress database]"
      mysql_db:
        name: wordpress
        state: present
    - name: "Criar [WordPress database user]"
      mysql_user:
        name: worduser
        password: PSwTQ$ek5
        priv: wordpress.*:ALL
        host: "localhost"
        state: present
       #######
    # NGINX
    #
    ######
    - name: "Copiar [NGINX]"
      template:
        src: config/wordpress.conf
        dest: "/etc/nginx/conf.d/wordpress.conf"

    - name: "Restart [NGINX]"
      shell: sudo systemctl restart nginx

    

    - name: "Restart [PHP-FPM]"
      shell: sudo systemctl restart php-fpm
  #ALTERAR SOMENTE A LINHA NECESSARIA REALIZANDO BACKUP DE VERSOES
    - lineinfile:
        path: /etc/php-fpm.d/www.conf
        regexp: 'user = apache'
        line: 'user = nginx'
        backup: yes

    - lineinfile:
        path: /etc/php-fpm.d/www.conf
        regexp: 'group = apache'
        line: 'group = nginx'
        backup: yes

    - lineinfile:
        path: /etc/php-fpm.d/www.conf
        regexp: 'listen = 127.0.0.1:9000'
        line: 'listen = /var/run/php/php-fpm.sock'
        backup: yes

    - lineinfile:
        path: /etc/php-fpm.d/www.conf
        regexp: ';listen.owner = nobody'
        line: 'listen.owner = nginx'
        backup: yes

    - lineinfile:
        path: /etc/php-fpm.d/www.conf
        regexp: ';listen.group = nobody'
        line: 'listen.group = nginx'
        backup: yes
    - name: "PERMISSÃO PASTA VAR"
      shell: sudo chown -R root:nginx /var/lib/php

    - name: "Restart [PHP-FPM]"
      shell: sudo systemctl restart php-fpm  
