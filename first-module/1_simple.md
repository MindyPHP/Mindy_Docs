# Создание своего модуля на Mindy


Что такое модуль, из чего он состоит и с чем его едят?  Попробуем написать собственный модуль с нуля и в процессе поймем, что это такое. Определим цель создания модуля: на нашем сайте он будет заниматься отображением продукции, которую продает наша компания. Пусть это будет мебель.

Перейдем в папку с модулями Mindy (app/Modules) и создадим там папку своего модуля. Назовем ее Furniture (Мебель). 

Далее, в этой папке создадим файл модуля **FurnitureModule.php** следующего содержания:

	<?php

	namespace Modules\Furniture;

	use Mindy\Base\Module;

	class FurnitureModule extends Module
	{
	}

Структура нашего модуля такова:

	Furniture
	- FurnitureModule.php

Для того, чтобы сказать Mindy о том, что наш модуль включен, необходимо добавить название модуля в блок **modules** файла настроек **settings.php**:

	<?php
	return [
		...
		'modules' => [
			...
			'Furniture'
		]		
		...		
	]; 

## Модель

Для хранения данных о нашей мебели, необходима модель. Модель занимается хранением, обработкой данных и хранит в себе методы работы с этими данными. 

*Модель в Mindy является типичным примером модели из концепции [MVC (Model-View-Controller)](https://ru.wikipedia.org/wiki/Model-View-Controller)*

Модели хранятся внутри модуля, в отдельной папке **Models**. Создадим ее:

	Furniture
	- Models
	- FurnitureModule.php
	
Внутри папки с моделями создадим необходимую нам модель **Furniture.php**:

	<?php
	namespace Modules\Furniture\Models;

	use Mindy\Orm\Model;

	class Furniture extends Model
	{
    	public static function getFields() 
    	{
        	return [
        
        	];
    	}
    }

Метод **getFields()**, описанный в файле модели хранит в себе описание набора полей, соответствующих данной модели. 

Допустим, на данном этапе, нам необходимо хранить лишь наименование продукта (предмета мебели) и цена данной модели. То есть поля будет 2: наименование и цена. Наименование будет обязательным полем, цена - нет. 

Модель с описанными полями:

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
                	'required' => true
            	],
            	'price' => [
                	'class' => DecimalField::className(),
                	'precision' => 10,
                	'scale' => 2
           		]
        	];
    	}
    }
    
Добавим еще один метод, опеределяющий приведение модели к строке:

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
                	'required' => true
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
    }
    
Теперь, если попробовать привести экземпляр модели к строковому виду:

	(string)$model
 
 
либо просто вывести в шаблоне:

	{{ model }}
	
мы получим наименование данной модели.

Структура нашего модуля сейчас такова:

	Furniture
	- Models
		- Furniture.php
	- FurnitureModule.php
	
После определения всех полей модели, выполним в консоли команду, которая автоматически создаст необходимую таблицу в БД и добавит в нее необходимые поля:

	# php www/index.php db sync
	
## Администрирование

Мы описали модель. Теперь было бы неплохо добавить нашу продукцию в БД. Можно, конечно, сделать это руками, либо написать консольную команду, которая возьмет эти данные, допустим, из файла CSV или XML. Но если у нас небольшой сайт - проще это делать в административной части сайта. 

Mindy предоставляет удобный интерфейс управления данными, воспользуемся им.
Для нашей модели необходимо описать особый класс, который укажет Mindy, как отображать данные в панели администрирования. Классы администрирования находятся в модуле в папке Admin. Создадим ее:

	Furniture
	- Admin
	- Models
		- Furniture.php
	- FurnitureModule.php

Внутри папки с классами администрирования создадим необходимый нам класс **FurnitureAdmin.php**:

	<?php
	namespace Modules\Furniture\Admin;

	use Modules\Admin\Components\ModelAdmin;
	use Modules\Furniture\Models\Furniture;

	class FurnitureAdmin extends ModelAdmin
	{
    	public function getModel()
    	{
        	return new Furniture;
    	}
	}
	
В методе **getModel()** указана модель, управление которой и будет осуществлятся из администраторской панели.

Структура модуля такова:

	Furniture
	- Admin
		- FurnitureAdmin.php
	- Models
		- Furniture.php
	- FurnitureModule.php

После того, как класс администрирования создан можно добавить его в меню администраторской панели. Производится это путем добавления метода **getMenu()** в класс нашего модуля. Класс модуля будет выглядеть следующим образом:

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
            	]
        	];
    	}
	}
Структура массива, возвращаемого методом **getMenu()** такова: 'name' - имя модуля, которое будет отображено в меню администраторской панели, 'items' - массив с элементами меню данного модуля, каждый из которых отвечает за определенный класс администрирования. В элементе массива 'items' должно быть 
определено имя 'name' и класс администрирования 'adminClass'.

Мы добавили наш класс администрирования в меню, указав в качестве имени следующую конструкцию:

	self::t('Furniture')

Это - применение компонента перевода Mindy. Подробнее об этом компоненте можно прочитать вот тут: [Перевод в Mindy](#TODO).

Ура! В меню администраторской панели добавился наш пункт:

![Новый пункт меню](https://www.dropbox.com/s/ubzmonywxgdyriq/Screenshot%202014-10-24%2009.52.38.png?dl=0&raw=1)

Перейдя по ссылке в меню мы увидим стандартный вывод списка объектов:

![Стандартный вывод списка объектов](https://www.dropbox.com/s/bvm7wxprqv3rc3b/Screenshot%202014-10-24%2010.08.21.png?dl=0&raw=1)

Добавим пару-тройку продуктов. Нажимаем на кнопку создать, перед нами появляется форма добавления объекта:

![Стандартный вывод списка объектов](https://www.dropbox.com/s/6zq5nl1zkquh743/Screenshot%202014-10-24%2010.46.09.png?dl=0&raw=1)

Заполним ее и нажмем сохранить. Добавим еще объектов. В итоге получим:

![Список с объектами](https://www.dropbox.com/s/wzazybw1lfjcswi/Screenshot%202014-10-24%2010.51.24.png?dl=0&raw=1)

Подробнее про систему администрирования Mindy можно прочитать вот тут: [Администрирование в Mindy](#TODO)

## Контроллер

Для того, чтобы вывести необходимую продукцию на наш сайт необходимо описать несколько вещей: контроллер, файл urls, шаблон.

Контроллер обеспечивает связь между пользователем и системой: контролирует ввод данных пользователем и использует модель и представление для реализации необходимой реакции. [Wikipedia]

Наш контроллер будет использовать модель для получения данных о продукции и передавать данную информацию в шаблон для вывода.

Контроллеры в модуле Mindy находятся в отдельной папке **Controllers**. Добавим папку в наш модуль:

	Furniture
	- Admin
		- FurnitureAdmin.php
	- Controllers
	- Models
		- Furniture.php
	- FurnitureModule.php
	
Добавим класс контроллера **FurnitureController.php** в папку **Controllers**:

	<?php
	namespace Modules\Furniture\Controllers;

	use Modules\Core\Controllers\CoreController;

	class FurnitureController extends CoreController
	{
    	public function actionList()
    	{
        
    	}
	}

Action list будет отвечать за обработку данных по url списка продукции. Грубо говоря, что будет делать система при переходе по url списка продукции. 

Структура модуля:

	Furniture
	- Admin
		- FurnitureAdmin.php
	- Controllers
		- FurnitureController.php
	- Models
		- Furniture.php
	- FurnitureModule.php

По нашей логке, нам нужно получить из БД продукцию и передать ее в шаблон для вывода. Добавим код в необходимый для этого action, :

	<?php
	namespace Modules\Furniture\Controllers;

	use Modules\Core\Controllers\CoreController;
	use Modules\Furniture\Models\Furniture;
	
	class FurnitureController extends CoreController
	{
    	public function actionList()
    	{
        	$furniture = Furniture::objects()->all();
        	echo $this->render('furniture/list.html', [
            	'furniture' => $furniture
        	]);
    	}
	}
	
В первой строке action мы выбрали все продукты из БД. Далее вызывается функция render контроллера, которая принимает 2 входящих параметра: шаблон для вывода и данные, которые необходимо передать в этот шаблон.

## urls

Для того, чтобы указать при переходе по какому url какой action какого контроллера вызывать в Mindy существуют файлы роутинга **urls.php**. Общий файл роутинга находится в конфигурационной папке. Файл роутинга отдельного модуля находится в папке с модулем. Добавим в наш модуль файл **urls.php**:

	Furniture
	- Admin
		- FurnitureAdmin.php
	- Controllers
		- FurnitureController.php
	- Models
		- Furniture.php
	- FurnitureModule.php
	- urls.php
	
Содержание нашего файла **urls.php**:

	<?php

	return [
    	'/list' => [
        	'name' => 'list',
        	'callback' => '\Modules\Furniture\Controllers\FurnitureController:list',
    	],
	];

Файл urls содержит массив "роутов": ключом массива является часть (почему часть - будет указано ниже) url, при переходе по которой выполнится callback. В качестве callback модет быть передано указание на Controller (в нашем случае - "\Modules\Furniture\Controllers\FurnitureController") и action (в нашем случае - "list"), либо анонимная функция.

Укажем Mindy на файл роутинга нашего модуля, просто добавив следующую запись в общий **urls.php** файл:

	<?php

	use Mindy\Router\Patterns;

	return [
		...
		'/furniture' => new Patterns('Modules.Furniture.urls', 'furniture')
		...
	]
	
Теперь, при обращени по url */furniture/list* будет отрабатывать **actionList()** из **FurnitureController**. 

Подробнее про систему роутинга в Mindy можно прочитать тут: [Роутинг в Mindy](#TODO).

## Шаблоны

Дело за малым - осталось написать шаблон вывода списка нашей продукции. В модуле шаблоны находятся в своей папке - **templates**, добавим ее:

	Furniture
	- Admin
		- FurnitureAdmin.php
	- Controllers
		- FurnitureController.php
	- Models
		- Furniture.php
	- templates
	- FurnitureModule.php
	- urls.php
	
Поместим в папку с шаблонами папку с именем нашего модуля в lowercase (в нижнем регистре) - можно сделать и иначе, но будем следовать правилам хорошего тона: 

	Furniture
	- Admin
		- FurnitureAdmin.php
	- Controllers
		- FurnitureController.php
	- Models
		- Furniture.php
	- templates
		- furniture
	- FurnitureModule.php
	- urls.php
	
 Добавим в эту папку шаблон **list.html**, следующего содержания:
 
	{% extends "base.html" %}

	{% block content %}
    	{% for item in furniture %}
        	Наименование: {{ item.name }}, Цена: {{ item.price }} руб. <br/>
    	{% endfor %}
	{% endblock %}
	
Первой строкой в шаблоне мы указали наследование от основного шаблона "base.html". Далее, перекрыли блок "content" основного шаблона и вывели в нем информацию о нашей продукции.

Подробнее о системе шаблонов в Mindy можно прочитать вот тут: [Шаблонизатор Mindy](#TODO)

## Результат

На текущий момент у нас есть страничка, доступная по url */furniture/list*, на которой выводится список нашей продукции и раздел в администраторской панели сайта для добавления/редактирования продукции. Далее, мы будем расширять наш модуль, представляя, что над нами стоит злой-злой менеджер и просит всё сделать еще вчера.