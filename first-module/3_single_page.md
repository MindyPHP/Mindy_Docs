# Страница отдельного объекта

На текущий момент у нас есть лишь список объектов и посмотреть продукт на отдельной странице невозможно. Исправим это!

## urls.php

Зарегистрируем новый "роут" для страницы отдельного объекта, расширив файл **urls.php** нашего модуля:

```php
<?php

return [
    '/list' => [
        'name' => 'list',
        'callback' => '\Modules\Furniture\Controllers\FurnitureController:list',
    ],
    '/view/{pk:\d+}' => [
        'name' => 'view',
        'callback' => '\Modules\Furniture\Controllers\FurnitureController:view',
    ],
];

```
Второй роут (который мы и добавили) практически идентичен первому, за исключением вот этой конструкции:

```
{pk:\d+}
```

Здеcь pk - это входящий параметр в роут, который будет передан в контроллер (см. раздел Контроллер). Далее, после знака '**:**' указано регулярное выражение, которому должен соответствовать входящий параметр. В нашем случае это **\d+** , то есть одна или более цифр.

## Контроллер

В контроллер **FurnitureController** необходимо добавить еще один **action**, который и будет отвечать за вывод страницы отдельного объекта. Код получившегося контроллера:

```php
<?php

namespace Modules\Furniture\Controllers;

use Mindy\Pagination\Pagination;
use Modules\Core\Controllers\CoreController;
use Modules\Furniture\Models\Furniture;

class FurnitureController extends CoreController
{
    public function actionList()
    {
        $pager = new Pagination(Furniture::objects());
        $furniture = $pager->paginate();
        echo $this->render('furniture/list.html', [
            'furniture' => $furniture,
            'pager' => $pager
        ]);
    }

    public function actionView($pk)
    {
        $item = Furniture::objects()->filter(['pk' => $pk])->get();
        if (!$item) {
            $this->error(404);
        }
        echo $this->render('furniture/view.html', [
            'item' => $item
        ]);
    }
}
```

Входящим параметром **$pk** в **actionView** является pk (первичный ключ) продукта, который мы хотим открыть на отдельной странице. Первой строкой в **actionView** мы выбираем объект по первичному ключу (метод filter), указывая, что выбирать нужно не список объектов, а один объект (метод get). Далее, если объект не найден - выводим 404 ошибку, иначе - рендерим шаблон **furniture/view.html**, передавая в него параметр **item**, который содержит выбранный объект.

## Шаблон

Осталось добавить шаблон нашей странички (**furnitue/view.html**). Добавим его в ранее созданную папку **templates/furniture**:

```
Furniture
- Admin
	- FurnitureAdmin.php
- Controllers
	- FurnitureController.php
- Models
	- Furniture.php
- templates
	- furniture
		- custom_pager.html
		- list.html
		- view.html
- FurnitureModule.php
- urls.php
```

Допустим, он может быть таким:

```twig
{% extends "base.html" %}

{% block content %}
	Наименование: {{ item.name }}<br/>
	Цена: {{ item.price }} руб.
{% endblock %}
```

Теперь, если перейти по url, например, **furniture/view/2** мы увидим наш объект с pk 2.

Это отлично, но давайте сделаем, чтобы в списке продукции выводилась ссылка на отдельный объект:

```twig
{% extends "base.html" %}

{% block content %}
    {% for item in furniture %}
        Наименование: {{ item.name }}, Цена: {{ item.price }} руб.
        <a href="{% url 'furniture.view' item.pk %}">Подробнее</a>
        <br/>
    {% endfor %}

    {{ pager.render('furniture/custom_pager.html')|raw }}
{% endblock %}
```

Встроенный в шаблонизатор тэг **url** автоматически построит url до объекта. Первым входящим параметром является имя роута (в нашем случае - **furniture.view**). Оно состоит из двух частей, разделенных точкой - неймспейса, определенного в общем файле **urls.php** (в нашем случае - **furniture**) и имени роута, указанного в **urls.php** нашего модуля (в нашем случае - **view**).

Откроем наш список продукции и убедимся - всё действительно работает.

## getAbsoluteUrl()

Вывести ссылку в нашем случае можно и иным способом - воспользовавшись методом модели **getAbsoluteUrl()** - этот метод возвращает ссылку (url) на объект модели. Это как-раз то что нам нужно. Добавим этот метод в нашу модель:

```php
<?php

namespace Modules\Furniture\Models;

use Mindy\Orm\Model;
use Mindy\Orm\Fields\CharField;
use Mindy\Orm\Fields\DecimalField;

class Furniture extends Model
{
    public static function getFields()
    {
        return [
            'name' => [
                'class' => CharField::className(),
                'required' => true,
            ],
            'price' => [
                'class' => DecimalField::className(),
                'precision' => 10,
                'scale' => 2
            ]
        ];
    }

    public function __toString()
    {
        return $this->name;
    }

    public function getAbsoluteUrl()
    {
        return $this->reverse('furniture.view', ['pk' => $this->pk]);
    }
}
```

Формирование url происходит практически так же как и с помощью тэга **url** в шаблоне, но описание метода **getAbsoluteUrl()** дает нам несколько дополнительных приятных моментов. В первую очередь - это возможность изменять роут и его структуру (например, выбирать данные не по **pk**, а по другому полю/полям) и не менять при этом шаблон (в следущей статье нам это пригодится). Во-вторых, мы сможем обращаться к этому методу не только из шаблонов, но и, например, при отправке писем или формировании иных уведомлений.

Осталось добавить обращение к методу в наш шаблон:

```twig
{% extends "base.html" %}

{% block content %}
    {% for item in furniture %}
        Наименование: {{ item.name }}, Цена: {{ item.price }} руб.
        <a href="{{ item.getAbsoluteUrl() }}">Подробнее</a>
        <br/>
    {% endfor %}

    {{ pager.render('furniture/custom_pager.html')|raw }}
{% endblock %}
```

## Результат

У нас появилась страничка отдельного объекта, мы чуть больше узнали о роутинге, познакомились с методом **getAbsoluteUrl()**. В следующих статьях мы разберем как сделать url отдельного объекта [человекопонятным](https://ru.wikipedia.org/wiki/%D0%A7%D0%9F%D0%A3_(%D0%98%D0%BD%D1%82%D0%B5%D1%80%D0%BD%D0%B5%D1%82) и добавим мета-информацию к нашим страницам.