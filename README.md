# Собираем пакет и создаём репозиторий
## Задание выполнялось с использованием VirtualBox 6.1.40. ВМ собирались с помощью vagrant на базе box-образа generic/debian10 (debian 10.13)
## В результате:

##Собираем DEB-пакет 
- Устанавливаем пакеты необходимые для сборки deb-пакетов
```
sudo apt-get install build-essential autoconf automake autotools-dev dh-make debhelper devscripts fakeroot xutils lintian pbuilder
```

- Качаем исходный код **Nginx** версии `1.22.1`. В **debian 10.13** исходно была версия `1.14.2`
```
mkdir ~/test2/
cd ~/test2/
wget https://nginx.org/download/nginx-1.22.1.tar.gz
tar -xzf nginx-1.22.1.tar.gz
```

- Создаем необходимую структуру для построения deb-пакета
```
cd nginx-1.22.1/
dh_make -e behlc@e1.ru -f ../nginx-1.22.1.tar.gz
```
В процессе бдует задан вопрос, гворим s - т.е. single binary. В результате будет в текущей папке будет создана папка **debian**, некоторые файлы из нее необходимо будет подправить.
![]()

- Проверяем настройки и наличие всего необходимого для построения пакета. Попутон получаем список зависимостей
```
dpkg-depcheck -d ./configure
```
Получили спсок зависимостей:
```
Packages needed:
  libc6-x32
  libpcre3-dev:amd64
  libfakeroot:amd64
  libc6-i386
  zlib1g-dev:amd64
```
![](рис. 02)

- Внёс изменения в файл debian/control, отвечающий за сборку пакета
 ![](рис. 03) 
 
- Попробывал запустить сборку пакета (по идее пакет должен быть подписан)
```
dpkg-buildpackage -rfakeroot --sign-key=D948F4DA0635BACF
```
К сожалению процесс сборки сломался... Пришлось схитрить, с сайта `nginx` скачал src-пакета для debian (версия 1.22.0), хотел взять за пример. Но распаковав его и изучив содержимое файлов в папке **debian**, пришёл к выводу, что за разумное время мне не осилить всаимосвязь сущеностей внутри этих файлов и просто скопировал все файлы в свою папку **debian** (**control**, **rules**, **CHANGES** и т.п.). Повторно запустил сборку.
```
cp -n ~/nginx-1.22.0/debian/* ~/build/nginx-1.22.1/debian/
dpkg-buildpackage -rfakeroot --sign-key=D948F4DA0635BACF
```
![](рис. 04)

8. Экпортируем открытый и приватный ключи. Подписывать репозиторий будем теми же ключами, что и пакет.
gpg --armor --output repo.gpg.pub.key --export behlc
gpg -a --outpu repo.gpg.priv.key --export-secret-keys behlc

9. dpkg -i nginx
vi /lib/systemd/system/nginx.service - исправил /var/run/nginx.pid -> /run/nginx.pid
systemctl daemon-reload 

gpg --batch --gen-key key.gen
gpg --import foo.pub

/etc/nginx/conf.d/default.conf
location / {
        root   /var/www/repo;
        autoindex on;
        #index  index.html index.htm;
    }


gpg --digest-algo SHA256 --clearsign --output InRelease Release
gpg --digest-algo SHA256 --armor --output Release.gpg --detach-sign Release
