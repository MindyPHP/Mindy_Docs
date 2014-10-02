Установка Mindy в Unix-like системах (Внимание! Это не завершенная статья, продолжение будет в течении 1 дня)

В продолжение первой статьи с описанием структуры Mindy Framework, в этой статье я расскажу о том как происходит установка системы в Unix-like системах.

Установка будет рассмотрена на основе Ubuntu 12.04 lts. Для владельцев Ubuntu 13 и выше данный абзац можно пропустить.

Первым делом добавляем репозиторий с php 5.5 (минимальные системные требования php 5.4)
```bash
sudo add-apt-repository ppa:ondrej/php5
sudo apt-get update
sudo apt-get install -yq php-pear php5-cli php5-common php5-curl php5-gd php5-mcrypt php5-mysql php5-pgsql
```

После успешной установки проверяем версию php командой `php -v` и должны увидеть нечто подобное:
```
[user:container224:~]$ php -v
PHP 5.5.17-2+deb.sury.org~precise+1 (cli) (built: Sep 25 2014 09:08:12)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
    with Zend OPcache v7.0.4-dev, Copyright (c) 1999-2014, by Zend Technologies
```

Если у вас стоял до этого настроенный apache2 версии 2.2, пожалуйста прочитайте документацию по переходу на apache2 версии 2.4 на официальном сайте.

Перезапускаем `apache`
```
sudo service apache2 restart
```

**Установка системы**

Предположим что основной путь к проекту будет `/home/user/project`.

Переходим в данную папку и выполняем `git clone git@github.com/Studio107/Mindy .`. Точка в конце означает клонирование репозитория в текущую папку.

После клонирования нам необходимо выполнить `sh ./install.sh` для установки списка доступных на текущий момент бесплатных модулей.

Обратите внимание, что система по умолчанию использует `gulp` в качестве сборщика, что требует наличие `node.js` на сервере.

**Конфигурация**