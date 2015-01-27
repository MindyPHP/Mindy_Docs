# Человеко-понятные Url

Сейчас ссылка на страницу отдельного продукта выглядит как **/furniture/view/{number}** (например, **/furniture/view/1**).

Для того, чтобы человеку было удобнее ориентироваться - транслитерируем имя и используем его как ссылку, например: **/furniture/view/stul-s-vitoi-spinkoi**.

Стало гораздо понятнее, чего ожидать от этой страницы. Теперь, выполним эту задачу всего в несколько простых шагов. 

## Модель

Для того, чтобы хранить транслитерированное имя добавим новое поле в нашу модель: 

```php
...
class Furniture extends Model
{
    public static function getFields()
    {
        return [
            'name' => [
                'class' => CharField::className(),
                'required' => true,
            ],
            'slug' => [
                'class' => SlugField::className(),
                'source' => 'name',
                'unique' => true
            ],
            'price' => [
                'class' => DecimalField::className(),
                'precision' => 10,
                'scale' => 2
            ]
        ];
    }

    ...
}
```

Поле SlugField автоматически транслитерирует данные из поля **name** (указано в параметре **source**) и сохранит их. 
Параметр **unique** указывает на то, что ссылки должны быть уникальными для каждого продукта 
(при совпадении ссылок к новой ссылке будет дописано число 1, в следующий раз 2 и так далее, то есть ссылки будут всегда разными).

## urls

Изменим файл url, для того, чтобы работала логика вывода по полю **slug**, а не по **pk**:

```php
<?php

return [
    '/list' => [
        'name' => 'list',
        'callback' => '\Modules\Furniture\Controllers\FurnitureController:list',
    ],
    '/view/{slug:c}' => [
        'name' => 'view',
        'callback' => '\Modules\Furniture\Controllers\FurnitureController:view',
    ],
];
```

Мы заменили имя параметра (**pk** на **slug**) и регулярное выражение на краткую запись регулярного выражения, 
позволяющего работать slug (**\d+** заменено на **c**, где **c** - краткая запись регулярного выражения **[a-zA-Z0-9+_\-\.]+**).

PS: Тут все страшно описано, но если почти ничего не понятно, то можно просто двигаться дальше - впоследстии с опытом вы разберетесь в тонкостях работы UrlManager.

## Controller

Теперь, нужно изменить выборку с pk на slug в Controller:

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

    public function actionView($slug)
    {
        $item = Furniture::objects()->filter(['slug' => $slug])->get();
        if (!$item) {
            $this->error(404);
        }
        echo $this->render('furniture/view.html', [
            'item' => $item
        ]);
    }
}
```

Здесь почти ничего не поменялось, только в **actionView** поле и параметр **pk** был заменен на **slug**.

## getAbsoluteUrl

Осталось изменить логику работы метода **getAbsoluteUrl** в нашей модели:

```php
<?php

namespace Modules\Furniture\Models;

use Mindy\Orm\Fields\SlugField;
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
            'slug' => [
                'class' => SlugField::className(),
                'unique' => true
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
        return $this->reverse('furniture:view', ['slug' => $this->slug]);
    }
}
```

Аналогично контроллеру, здесь мы заменили лишь **pk** на **slug**. Все.

## Итог

Вот так, в несколько простейших шагов мы добавили ЧПУ к нашей модели. 
И, что самое главное, при использовании метода **getAbsoluteUrl** нам не пришлось изменять шаблоны нашего модуля, 
что может быть достаточно критичным, при написании больших приложений, когда количество шаблонов числится десятками, 
а то и сотнями.
