# Установка системы (разработчику)

> Перед началом чтения пожалуйста ознакомтесь с системными требованиями системы

Предположим что основной путь к проекту будет `/home/user/project`.

**Обратите внимание, что система по умолчанию использует `gulp` в качестве сборщика, что требует наличие `nodejs` на сервере. Так же необходимо установить composer. Чтобы установить composer на windows — просто скачиваем инсталятор. Для unix вводим следующие строки в консоли:
curl -sS https://getcomposer.org/installer | php 
mv composer.phar /usr/local/bin/composer**

```bash
cd /home/user/project # переходим в директорию с проектом
git clone git@github.com/Studio107/Mindy.git . # Клонируем репозиторий в текущую папку
sh ./install.sh # Устанавливаем основные модули системы
cd app # Переходим в директорию app
composer install --no-dev # Устанавливаем зависимости системы

cd ../www/static_admin # Переходим в директорию static_admin
npm install # Устанавливаем зависимости npm
bower install # Устанавливаем зависимости bower
gulp # Компилируем статику панели администратора

cd ../static # Переходим в директорию static
npm install # Устанавливаем зависимости npm
bower install # Устанавливаем зависимости bower
gulp # Компилируем статику которую будем использовать на сайте
```

## Конфигурация

Перейдем в директорию `app/config` и переименуем файл `settings_local.php.dist` в `settings_local.php` для разработки.

В файле `settings_local.php` перекрываем настройки подключения к базе данных, перекрываем почтовые компоненты (к примеру, для верстки писем), и т.д.

Пример файла `settings_local.php`:

```php
<?php

return \Mindy\Utils\Settings::override(require(dirname(__FILE__) . '/settings.php'), [
    'components' => [
        'db' => [
            'class' => '\Mindy\Query\ConnectionManager',
            'databases' => [
                'default' => [
                    # Подключение к локальной базе данных
                    'class' => '\Mindy\Query\Connection',
                    # Хост и имя базы данных
                    'dsn' => 'mysql:host=localhost;dbname=mindy',
                    # Имя пользователя для подключения к бд
                    'username' => 'root',
                    # Пароль подключения
                    'password' => '123456',
                    # Кодировка по умолчанию используемая в проекте
                    'charset' => 'utf8',
                ]
            ]
        ],
        'cache' => [
            # Кеш пустышка
            'class' => '\Mindy\Cache\DummyCache'
        ],
        'mail' => [
            # Почтовый компонент с шаблонами в базе данных
            'class' => '\Modules\Mail\Components\DbMailer',
            # Сохранять отправленные письма в файлы в runtime/mail
            'useFileTransport' => true
        ],
    ],
]);
```

## Запуск

Перейдем в директорию `/home/user/project` и запустим наш сайт:

```bash
php -S 0.0.0.0:8000 -t www/
```

Далее проверяем доступность нашего сайта по адресу [localhost:8000](http://localhost:8000/)

Так же вы можете воспользоваться файлом в корне проекта `Procfile` используемый [foreman](https://github.com/ddollar/foreman) или его форк на `golang` [goreman](https://github.com/mattn/goreman) для запуска всех зависимостей проекта.

```bash
foreman start
```

или

```bash
goreman start
```
