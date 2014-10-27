# Пагинация объектов

Важным элементом вывода объектов на сайте является их пагинация (постраничный вывод со списком страниц)

Предположим, что на нашем сайте образовалось больше сотни объектов. Выводить их на одной странице зачастую, не оправдано - страница получается очень длинная, нагруженная информацией, поэтому применяют постраничный вывод объектов. Mindy позволяет автоматизировать этоту процедуру. Операция затрагивает контроллер и шаблон вывода.


## Контроллер

Рассмотрим изменения в ранее созданном файле контроллере FurnitureController.php:

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
}
```

У нас появился новый объект **Pagination**, который и отвечает за постраничный вывод объектов. Для постраничного вывода объектов необходимо создать экземпляр класса Pagination, в конструктор которого и передаются объекты. В качестве первого входящего параметра (то есть списка объектов) могут быть использованы: массив (array), QuerySet ([Подробнее о QuerySet](#TODO)) и Manager ([Подробнее о Manager](#TODO)).

Вторым входящим параметром (не обязательным) можно передать массив конфигурации пагинатора. В этом случае создание объекта может выглядеть следующим образом:

```php
...
	$pager = new Pagination(Furniture::objects(), [
		'pageSize' => 25
	]);
...
```

Указанная конфигурация указывает пагинатору на то, что на одну страницу следует выводить 25 элементов *(по-умолчанию это число составляет 10)*.

Список объектов на текущей странице мы получаем уже от объекта класса **$pager** с помощью метода **paginate()**, который применяет LIMIT и OFFSET в случае передачи списка объектов QuerySet и Manager, и применяет array_slice() в случае передачи списка объектов массивом. Грубо говоря, Pagination "вырезает" всего списка объектов нужную страничку с объектами.

А не проще ли использовать только массив - выбрать всё и вывести только то, что нужно на данной странице? С точки зрения производительности выбирать лишние данные из БД (те, которые не будут отображены на данной странице) - ненужные расходы ресурсов.

## Шаблон

В созданном ранее нами шаблоне необходимо добавить одну строку. Сделаем это:

```twig
{% extends "base.html" %}

{% block content %}
    {% for item in furniture %}
        Наименование: {{ item.name }}, Цена: {{ item.price }} руб. <br/>
    {% endfor %}

    {{ pager.render()|raw }}
{% endblock %}
```

В данном случае выведется шаблон пагинатора по-умолчанию. В функцию **render()** можно передать свой собственный шаблон пагинации. Применим свой шаблон вывода. Нам нужен лишь список страниц:

```twig
...
{{ pager.render('furniture/custom_pager.html')|raw }}
...
```

В папке с шаблонами создадим файл **custom_pager.html**:

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
		- list.html
		- custom_pager.html
- FurnitureModule.php
- urls.php
```

Содержимое файла **custom_pager.html**:

```twig
{% if this.getPagesCount() > 1 %}
    <section class="pager-container">
        <ul class="pager">
            {% if this.hasPrevPage() %}
                <li class="prev">
                    <a href="{{ this.getUrl(this.currentPage - 1) }}">&larr;</a>
                </li>
            {% endif %}

            {% if this.hasPrevPage() %}
                {% for page in this.iterPrevPage() %}
                    <li>
                        <a href="{{ this.getUrl(page) }}">{{ page }}</a>
                    </li>
                {% endfor %}
            {% endif %}

            <li>
                <span>{{ this.currentPage }}</span>
            </li>

            {% if this.hasNextPage() %}
                {% for page in this.iterNextPage() %}
                    <li>
                        <a href="{{ this.getUrl(page) }}">{{ page }}</a>
                    </li>
                {% endfor %}
            {% endif %}

            {% if this.hasNextPage %}
                <li class="next">
                    <a href="{{ this.getUrl(this.currentPage+1) }}">&rarr;</a>
                </li>
            {% endif %}
        </ul>
    </section>
{% endif %}
```

Первой строкой мы определяем: если количество страниц больше одной (то есть имеется смысл выводить постраничную навигацию) - выводим пагинатор. Далее, если имеется предыдущая страница - выводим ее ссылку на нее, если имеются предыдущие страницы - выводим их, затем выводим номер текущей страницы. По аналогии поступаем и со страницами, больше текущией. Мы создали свой шаблон постраничной навигации. Пример расширенного шаблона пагинатора можно посмотреть в пакете **Modules/Core/templates/pager.html**.