Пишем свой модуль для Юпи!
==========================

В данной статье мы расмотрим как написать, установить и начать пользоваться своим собственным модулем для Юпи!
Прежде всего Вам необходимо ознакомиться с документацией по Yii, особенно той ее частью, которая касается разработки модулей. Модули Юпи! практически ничем не отличаются от стандартных модулей Yii, но свои небольшие особенности всетаки есть. 

### Порядок разработки ###

* Планируем функционал модуля
* Выбираем название и ID для модуля
* Проектируем таблицы базы данных
* Генерируем каркас модуля (gii)
* Модифицируем класс модуля для интеграции с Юпи!
* Генерируем модели для таблиц (gii)
* Генерируем CRUD интерфейсы для моделей (gii)
* Готовим миграции и файл конфигурации
* Доработка напильником

### Планируем функционал модуля ###
Мы с вами разработаем модуль для хранения и управления клиентами, их анкетными данными и историей контактов с ними. Если хотите - считайте это простейшей CRM. Модуль мы планируем использовать в практической работе, будем вести в нем клиентов нашей "корпорации" +)

### Выбор названия и ID для модуля ###
Конечно же название модуля должно быть осмысленным, простым и говорящем о назначении модуля.
Наш модуль для учета клиентов мы назовем просто "Клиенты". Для выбора ID модуля у нас есть несколько советов и рекомендаций:

* Должен содержать буквы только в нижнем регистре: **"blog"**, **"zendsearch"** - правильно, "SphinxSearch", "SomeCoolModule" - не очень правильно 
* Используйте единственное число: **"image"**, **"user"** - правильно, "cars", "quotes" - не очень правильно
* По возможности делайте ID  модуля коротким и говорящим

Для нашего модуля мы выбрем самый простой и говорящий ID - **"client"**.

### Проектируем базу данных ###

В Юпи! мы выработали несколько правил именования таблиц, столбцов, индексов и т.д.
Они очень простые, рекомендуем их придерживаться:

* Название таблицы формируется вот по такой маске **dbPrefix_moduleId_tableName**, например **yupe_blog_blog** или **yupe_blog_post**. ID модуля в этом примере служит для эмуляции неймспейсвов или для группировки таблиц и избежания конфликтов с именами таблиц из других модулей
* В названиях таблиц используется единственное число: **yupe_blog_post** - правильно, **yupe_blog_posts** - не правильно
* К названию индекса добавляется префикс **ix_**, например **ix_publish_date**
* К названию уникального индекса добавляется префикс **ux_**, например **ux_user_id**
* К названию внешних ключей добавляется префикс **fk_**, например **fk_user_id**

### Генерируем каркас модуля ###
У Юпи! пока нет собственного генератора модуля, в связи с этим мы будем использовать тот, ктороый идет из коробки в gii.
<div class='alert alert-error'>Прежде чем двигаться дальше, нам необходимо активировать gii в настройках нашего проекта! По умолчанию gii отключен, для его активации раскомментируйте следующие строчки в файле /protected/config/main.php:</div>

~~~
[php]

'gii'   => array(
    'class'          => 'system.gii.GiiModule',
    'password'       => 'giiYupe',
    'generatorPaths' => array(
        'application.modules.yupe.extensions.yupe.gii',
    ),
    'ipFilters'=>array(),
)

~~~

После этого авторизуйтесь в gii (обычно он доступен по адресу http://localhost/yupe/gii/default/login) и выберите пункт "Module Generator"

<img src='/yupe/web/images/docs/genmodule.png'>

На открывшейся странице в поле "Module ID" введите название нашего модуя - "client"

<img src='/yupe/web/images/docs/idmodule.png'>

Нажмите кнопку "Preview", на открывшейся страничке подтвердите генерацию каркаса модуля, нажав "Generate". Если после этого Вы увидели вот такой вот текст "The module has been generated successfully" - поздравляем, каркас модуля готов!

<div class='alert alert-error'>Для продолжения работы обязательно закомментируйте строки подключения gii в конфиге приложения, иначе при обращении к сайту Вы получите ошибку</div>

### Модифицируем класс модуля для интеграции с Юпи! ###

Итак, на предыдущем шаге мы сгенерировали заготовку модуля, теперь мы хотим сделать этот модуль полноценным модулем Юпи!, хотим видеть его в панели управления, хотим включать и выключать его через интерфейс админки. Звучит круто! Все очень просто! Приступим!

Если сейчас мы задйдем в панель управления и перейдем на страницу управления модулями ( Юпи! ---> Модули ) мы убедимся в том, что Юпи! ничего не знает про наш новенький модуль =( Давайте это исправим! Основные требоыания:

* **модуль обязательно должен содержать каталог "install"**

Сейчас у нас его нет, поэтому просто берем и создаем каталог /protected/modules/client/**install**

* **модуль обязательно должен содержать конфигурационный файл, название файла формируется как moduleId+'.php'**

Даже если у нашего модуля пока нет никаких настроек - файл должен обязательно быть. В нашем случае нам достаточно создать файл /protected/modules/client/install/**client.php**

Дальше нам придется немного попрограммировать на Ruby...на PHP конечно же =) Давайте посмотрим на класс сгенерированного нами модуля (/protected/modules/client/ClientModule.php) Выглядеть он должен вот так:

~~~
[php]


<?php

class ClientModule extends CWebModule
{
	public function init()
	{
		// this method is called when the module is being created
		// you may place code here to customize the module or the application

		// import the module-level models and components
		$this->setImport(array(
			'client.models.*',
			'client.components.*',
		));
	}

	public function beforeControllerAction($controller, $action)
	{
		if(parent::beforeControllerAction($controller, $action))
		{
			// this method is called before any module controller action is performed
			// you may place customized code here
			return true;
		}
		else
			return false;
	}
}

~~~








**РАБОТА НА РУКОВОДСТВОМ НЕ ЗАВЕРШЕНА, СЛЕДИТЕ ЗА ОБНОВЛЕНИЯМИ!**