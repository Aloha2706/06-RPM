# 06-NFS

Полезные ссылки
https://rpm-software-management.github.io/mock/
http://wiki.rosalab.ru/ru/index.php/%D0%A1%D0%B1%D0%BE%D1%80%D0%BA%D0%B0_RPM_-_%D0%B1%D1%8B%D1%81%D1%82%D1%80%D1%8B%D0%B9_%D1%81%D1%82%D0%B0%D1%80%D1%82
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/rpm_packaging_guide/index
https://blog.packagecloud.io/working-with-source-rpms/
https://rpm-packaging-guide.github.io/



Развертываем Vagrant

Устанавливаем необходимые пакеты

        yum install -y \
        redhat-lsb-core \
        wget \
        rpmdevtools \
        rpm-build \
        createrepo \
        yum-utils \
        gcc

Загрузим и распакуем SRPM пакет NGINX желательно в домашнюю папку обчного пользователя

        wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm

        rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm

Загрузим и распакуем исходники для openssl

        wget https://github.com/openssl/openssl/archive/refs/heads/OpenSSL_1_1_1-stable.zip
        unzip  OpenSSL_1_1_1-stable.zip

        --with-openssl=/home/vagrant/openssl-OpenSSL_1_1_1-stable


Заранее поставим все зависимости чтобы в процессе сборки не было ошибок

        sudo yum-builddep rpmbuild/SPECS/nginx.spec

В файле rpmbuild/SPECS/nginx.spec правим строчку чтобы указать на исходники Openssl 

        --with-openssl=/home/vagrant/openssl-OpenSSL_1_1_1-stable

собираем:

        rpmbuild -bb rpmbuild/SPECS/nginx.spec

смотрим результат здесь  и устанавливаем :

        [vagrant@rpm ~]$ ll rpmbuild/RPMS/x86_64/
        total 4396
        -rw-rw-r--. 1 vagrant vagrant 2007680 Dec 20 15:50 nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
        -rw-rw-r--. 1 vagrant vagrant 2489484 Dec 20 15:50 nginx-debuginfo-1.14.1-1.el7_4.ngx.x86_64.rpm
        [vagrant@rpm ~]$
        
        sudo yum localinstall -y  rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm

запускаем, проверяем. 

        sudo systemctl start nginx
        sudo systemctl status nginx

## Используем джинсы для создания своего рэпо 

mkdir /usr/share/nginx/html/repo
sudo cp  rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/

        sudo wget https://downloads.percona.com/downloads/percona-distribution-mysql-ps/percona-distribution-mysql-ps-8.0.28/binary/redhat/8/x86_64/percona-orchestrator-3.2.6-2.el8.x86_64.rpm -O /usr/share/nginx/html/repo/percona-orchestrator-3.2.6-2.el8.x86_64.rpm

Инициализируем репозиторий предварительно добавив права на запись:

        sudo chmod 777 /usr/share/nginx/html/repo
        createrepo /usr/share/nginx/html/repo/

В location / в файле /etc/nginx/conf.d/default.conf добавим директиву autoindex on.
        location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        autoindex on; Добавили эту директиву
        }

Проверяем синтаксис и перезапускаем nginx

        nginx -t
        nginx -s reload

смотрим что там опубликовалось и получаем два файла. 

        curl -a http://localhost/repo/

добавляем свой репозиторий и проверяем что он добавился

        cat >> /etc/yum.repos.d/otus.repo << EOF
        [otus]
        name=otus-linux
        baseurl=http://localhost/repo
        gpgcheck=0
        enabled=1
        EOF
        
        yum repolist enabled
        yum repolist enabled | grep otus
        yum list | grep otus

Пробуем установить percona-orchestrator

        yum install percona-orchestrator.x86_64 -y

Получаем ошибку со странными зависимостями.

        Processing Dependency: jq >= 1.5 for package: 2:percona-orchestrator-3.2.6-2.el8.x86_64
        --> Processing Dependency: oniguruma for package: 2:percona-orchestrator-3.2.6-2.el8.x86_64
        --> Processing Dependency: libc.so.6(GLIBC_2.28)(64bit) for package: 2:percona-orchestrator-3.2.6-2.el8.x86_64

Совственно работу собственного репозитария подтвердили. 



