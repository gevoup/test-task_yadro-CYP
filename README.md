# Тестовое задание в Common Yadro Team

## Задача

В тестовом задании требовалось разобраться со сборочной системой Yocto и poky. Собрать и запустить образ core-image-minimal из ветки Scarthgap, а затем добавить в образ пакет golang и обновить его до версии, актуальной в ветке master (1.22.12 -> 1.24.1).

## Выполнение

Для начала был скачан poky из ветки Scarthgap, и было иинициализировано окружение:
```
git clone -b scarthgap git://git.yoctoproject.org/poky
cd poky
source oe-init-build-env
```

Для задания не требовалась установка дополнительных слоёв, так как рецепт для golang содержался в предустановленном слое `openembedded-core` по пути `/meta/recipes-devtools/go`.

После чего был собран и запущен в qemu базовый образ `core-image-minimal`:
```
bitbake core-image-minimal
runqemu runqemu qemux86-64
```
Так, мы создали и запустили в qemu базовый образ Yocto.

Далее, внесём изменения в конфигурационный файл образа так, чтобы:
- go был добавлен в перечень инсталлируемых в target пакетов
- был добавлен класс `image-buildinfo`, чтобы в образе был создан файл с описанием сборки системы
    - также, в переменные, отображаемые в `buildinfo` была добавлена переменная `METADATA_BRANCH`, которая показывает текущую ветку слоя `openembedded-core`
    - файл располагается по пути `/etc/buildinfo`

Для этого, добавим в файл `local.conf` следующие строки:
```
INHERIT += "image-buildinfo"
IMAGE_BUILDINFO_VARS:append = " METADATA_BRANCH"
IMAGE_INSTALL:append = " go"
```

После чего, заменим рецепты go на версию из ветки master. Для этого, склонируем poky из ветки master, и заменим папку с рецептом go на новую:
```
git clone -b master git://git.yoctoproject.org/poky poky-master
rm -rf poky/meta/recipes-devtools/go/
cp -a poky-master/meta/recipes-devtools/go/ poky/meta/recipes-devtools/go/
```

Далее остаётся только пересобрать образ, и проверить правильность выполнения:
```
bitbake core-image-minimal
runqemu runqemu qemux86-64
```

Образ должен быть успешно собран и запущен в qemu. 

Для проверки правильности, проверим установленную версию go и посмотрим содержимое файла `buildinfo`:
```
go version
cat /etc/buildinfo
```

- Патчи к репозиториям хранятся в файле
- Скриншот команды go version из qemu хранится в файле
- Файл buildinfo из собранной системы размещён в файле

- Так же, изменённый файл конфигурации образа `local.conf` размещён в файле 
