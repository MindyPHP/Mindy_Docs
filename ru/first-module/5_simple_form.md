# Простейшая форма

Сейчас пользователь сайта в нашем модуле может лишь просмотреть продукцию, но не может связаться с администратором сайта. Предоставим ему такую возможность, создав форму заказа отдельного продукта, в которой человек может оставить свои контактные данные и с ним свяжется администрация сайта.

## Модель

Для хранения заявок создадим модель:

```php
<?php
namespace Modules\Furniture\Models;

use Mindy\Orm\Fields\CharField;
use Mindy\Orm\Fields\ForeignField;
use Mindy\Orm\Model;

class Request extends Model
{
    public static function getFields() 
    {
        return [
            'name' => [
                'class' => CharField::className(),
                'required' => true
            ],
            'phone' => [
                'class' => CharField::className(),
                'required' => true
            ],
            'furniture' => [
                'class' => ForeignField::className(),
                'modelClass' => Furniture::className()
            ]
        ];
    }
    
    public function __toString() 
    {
        return (string) $this->name . ' ' . $this->phone;
    }
}
```

Модель очень похожа на модель отдельного продукта, но здесь появилось новое поле - **ForeignField** - внешний ключ. Оно описывает связь с моделью Furniture таким образом, что у одной заявки (Request) может быть указан один продукт (Furniture).


После определения всех полей модели, выполним в консоли команду, которая автоматически создаст необходимую таблицу в БД и добавит в нее необходимые поля:

```bash
# php www/index.php db sync
```

## Форма

Для работы с формами создадим в нашем модуле папку **Forms**. Опишем форму заявки в файле **RequestForm.php**:

```php
<?php

namespace Modules\Furniture\Forms;

use Mindy\Form\Fields\HiddenField;
use Mindy\Form\ModelForm;
use Modules\Furniture\Models\Request;

class RequestForm extends ModelForm
{
    public function getFields()
    {
        return [
            'furniture' => [
                'class' => HiddenField::className()
            ]
        ];
    }

    public function getModel()
    {
        return new Request;
    }
}
```

Формы очень похожи на модели по своему описанию - в них точно так же описываются поля. Для работы с моделями существует особый класс форм - **ModelForm** - такие формы идеально подходят для работы с моделями. Основное их преимущество их в том, что описывать снова все поля из модели не нужно - они автоматически "подхватываются" из модели. 

На нашем примере видно, что в методе **getFields()** нет описания полей **name** и **phone** - они автоматически унаследуются от модели и их менять нам не нужно, в отличие от поля **furniture**, в которое значение должно устанавливаться автоматически, и которое пользователь не должен видеть.

## Контроллер

Внесем изменения в наш контроллер для обработки формы:

```php
<?php
...
    public function actionView($slug)
    {
        $item = Furniture::objects()->filter(['slug' => $slug])->get();
        if (!$item) {
            $this->error(404);
        }
        $form = new RequestForm();
        $form->setAttributes([
            'furniture' => $item->id
        ]);
        if ($this->r->isPost && $form->populate($_POST)->isValid() && $form->save()) {
            $this->r->flash->success('Вы успешно заказали товар! В ближайшее время с вами свяжется наш менеджер.');
            $this->redirect($item);
        }
        echo $this->render('furniture/view.html', [
            'item' => $item,
            'form' => $form
        ]);
    }
...

```

Здесь мы создаем экземпляр формы:

```php
$form = new RequestForm();
```

Далее, укажем значение в скрытое поле **furniture**:

```php
$form->setAttributes([
	'furniture' => $item->id
]);
```

Далее, обрабатываем форму следующим образом:

1. Проверим, что это POST-запрос
2. Наполним форму данными из запроса
3. Проверим, что форма валидна (что все поля заполнены верно)
4. Сохраняем результат в новую модель **Request** (этот метод доступен при использовании **ModelForm**)

```php
if ($this->r->isPost && $form->populate($_POST)->isValid() && $form->save()) {
	$this->r->flash->success('Вы успешно заказали товар! В ближайшее время с вами свяжется наш менеджер.');
	$this->redirect($item);
}
``

Если все вышеперечисленные шаги выполнены успешно, то запишем во **flash** сообщение об успешной отправке заявки и перенаправим пользователя обратно на страницу с товаром при помощи **redirect**.

*Flash-сообщения - сообщения, которые основаны на передаче данных через сессию - используются для разовой передачи сообщения пользователю. При следующем обновлении страницы сообщение будет удалено.*

## Шаблон

Изменим шаблон вывода отдельного объекта:

```twig
{% extends "base.html" %}

{% block content %}
    Наименование: {{ item.name }}<br/>
    Цена: {{ item.price }} руб.

	<h2>Заказать данный товар</h2>
    <form action="" method="post">
        {{ form.asBlock()|raw }}

        <button type="submit">
            Отправить
        </button>
    </form>
{% endblock %}
```

Теперь наша форма работает. Осталось предоставить интерфейс для работы с заявками.

## Админка

Добавим новый класс админки **RequestAdmin.php**:

```php
<?php

namespace Modules\Furniture\Admin;

use Modules\Admin\Components\ModelAdmin;
use Modules\Furniture\Models\Request;
use Modules\Furniture\FurnitureModule;

class RequestAdmin extends ModelAdmin
{
    public function getColumns()
    {
        return ['name', 'phone'];
    }
    
    public function getModel()
    {
        return new Request;
    }
    
    public function getActionsList()
    {
        return ['info', 'delete'];
    }
}
```

В методе **getActionsList()** переопределим стандартный вывод действий на *Просмотр* и *Удаление*, так как нет необходимости редактировать заявки.

Так же добавим пункт в модуль:

```php
<?php

namespace Modules\Furniture;

use Mindy\Base\Module;

class FurnitureModule extends Module
{
    public function getMenu()
    {
        return [
            'name' => $this->getName(),
            'items' => [
                [
                    'name' => self::t('Furniture'),
                    'adminClass' => 'FurnitureAdmin',
                ],
                [
                    'name' => self::t('Requests'),
                    'adminClass' => 'RequestAdmin',
                ],
            ]
        ];
    }
}

```

## Итог

Мы добавили форму заявки на отдельный продукт, а так же вывели заявки в админ-панель сайта. 