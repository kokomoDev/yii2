Html-помощник
=============

Каждое веб-приложение формирует большое количество HTML-разметки. Если разметка статическая, её можно эффективно
сформировать [смешиванием PHP и HTML в одном файле](https://www.php.net/manual/ru/language.basic-syntax.phpmode.php), но
когда разметка динамическая, становится сложно формировать её без дополнительной помощи. Yii предоставляет такую помощь
в виде Html-помощника, который обеспечивает набор статических методов для обработки часто-используемых HTML тэгов, их
атрибутов и содержимого.

> Note: Если ваша разметка близка к статической, лучше использовать непосредственно HTML. Нет никакой
  необходимости в том, чтобы всё подряд оборачивать вызовами Html-помощника.


## Основы <span id="basics"></span>

Так как формирование динамической HTML-разметки при помощи конкатенации строк очень быстро вносит хаос в проект, Yii 
предоставляет набор методов для управления атрибутами тэгов и формирования тэгов на основе этих атрибутов.


### Формирование тэгов <span id="generating-tags"></span>

Код формирования тэга выглядит следующим образом:

```php
<?= Html::tag('p', Html::encode($user->name), ['class' => 'username']) ?>
```

Здесь первый аргумент - это название тэга. Второй - содержимое, которое будет заключено между открывающим и закрывающим
тэгами. Заметьте, что мы используем `Html::encode`. Это связано с тем, что содержимое не экранируется автоматически,
чтобы можно было по-необходимости использовать чистый HTML. Третий аргумент - это массив настроек для HTML-кода, а
другими словами - массив атрибутов для тэга. В этом массиве ключи являются названиями атрибутов, например `class`,
`href` или `target`, а значения в массиве являются значениями этих атрибутов.

Вышеприведённый код сформирует следующую HTML-разметку:

```html
<p class="username">samdark</p>
```

В тех случаях, когда вам необходимо только закрыть или открыть тэг, можно использовать методы `Html::beginTag()` и
`Html::endTag()`.

Дополнительные атрибуты используются во многих методах Html-помощника и в различных виджетах. Во всех этих случаях в
дело вступают механизмы дополнительной обработки данных, о которых следует знать:

- Если значение равно `null`, соответствующий атрибут не будет выведен.
- Атрибуты, значения которых имеют тип boolean, будут интерпретированы как
  [логические атрибуты](http://www.w3.org/TR/html5/infrastructure.html#boolean-attributes).
- Значения атрибутов будут экранированы с использованием метода [[yii\helpers\Html::encode()|Html::encode()]].
- Если в качестве значения атрибута передан массив, он будет обработан следующим образом:

   * Если атрибут является одним из атрибутов данных, указанных в [[yii\helpers\Html::$dataAttributes]], например `data`
     или `ng`, то будет сформирован список атрибутов по одному для каждого элемента массива. Например, код
     `'data' => ['id' => 1, 'name' => 'yii']` сформирует `data-id="1" data-name="yii"`; а код 
     `'data' => ['params' => ['id' => 1, 'name' => 'yii'], 'status' => 'ok']` сформирует
     `data-params='{"id":1,"name":"yii"}' data-status="ok"`. Заметьте, что в последнем примере используется формат JSON
     для формирования вывода вложенного массива.
     
   * Если атрибут НЕ является атрибутом данных, значение будет сконвертировано в формат JSON. Например, код
     `['params' => ['id' => 1, 'name' => 'yii']` сформирует `params='{"id":1,"name":"yii"}'`.


### Формирование CSS классов и стилей <span id="forming-css"></span>

При формировании атрибутов для HTML-тэгов часто приходится начинать с некоторых атрибутов по умолчанию, которые затем
необходимо изменять. Для того, чтобы добавить или удалить CSS-класс, можно использовать следующий подход:

```php
$options = ['class' => 'btn btn-default'];

if ($type === 'success') {
    Html::removeCssClass($options, 'btn-default');
    Html::addCssClass($options, 'btn-success');
}

echo Html::tag('div', 'Всё получилось!', $options);

// в случае, если $type содержит строку 'success', будет сформирован вывод
// <div class="btn btn-success">Всё получилось!</div>
```

Можно указать несколько CSS-классов, используя синтаксис массива:

```php
$options = ['class' => ['btn', 'btn-default']];

echo Html::tag('div', 'Сохранить', $options);
// выведет '<div class="btn btn-default">Сохранить</div>'
```

При добавлении или удалении классов тоже можно использовать массивы:

```php
$options = ['class' => 'btn'];

if ($type === 'success') {
    Html::addCssClass($options, ['btn-success', 'btn-lg']);
}

echo Html::tag('div', 'Сохранить', $options);
// выведет '<div class="btn btn-success btn-lg">Сохранить</div>'
```

`Html::addCssClass()` предотвращает дублирование классов, поэтому можно не беспокоиться, что какой-либо класс
будет добавлен дважды:

```php
$options = ['class' => 'btn btn-default'];

Html::addCssClass($options, 'btn-default'); // класс 'btn-default' уже добавлен

echo Html::tag('div', 'Сохранить', $options);
// выведет '<div class="btn btn-default">Сохранить</div>'
```

Если CSS-класс задаётся с помощью массива, можно использовать именованные ключи массива для обозначения логического
предназначения класса. В этом случае класс с таким же ключом будет проигнорирован во время использования
`Html::addCssClass()`:

```php
$options = [
    'class' => [
        'btn',
        'theme' => 'btn-default',
    ]
];

Html::addCssClass($options, ['theme' => 'btn-success']); // ключ 'theme' уже использован

echo Html::tag('div', 'Сохранить', $options);
// отобразит '<div class="btn btn-default">Сохранить</div>'
```

CSS-стили могут быть установлены схожим образом с помощью атрибута `style`:

```php
$options = ['style' => ['width' => '100px', 'height' => '100px']];

// в результате будет style="width: 100px; height: 200px; position: absolute;"
Html::addCssStyle($options, 'height: 200px; position: absolute;');

// в результате будет style="position: absolute;"
Html::removeCssStyle($options, ['width', 'height']);
```

При использовании метода [[yii\helpers\Html::addCssStyle()|addCssStyle()]] можно указать массив, пары ключ-значение
которого соответствуют названиям и значениям CSS-свойств, или строку, например `width: 100px; height: 200px;`. Эти два
формата могут быть преобразованы друг в друга с помощью методов
[[yii\helpers\Html::cssStyleFromArray()|cssStyleFromArray()]] и
[[yii\helpers\Html::cssStyleToArray()|cssStyleToArray()]]. Метод
[[yii\helpers\Html::removeCssStyle()|removeCssStyle()]] принимает на вход массив атрибутов, которые следует удалить.
Если удаляется всего один атрибут, его можно передать строкой.


### Экранирование контента <span id="encoding-and-decoding-content"></span>

Для корректного и безопасного отображения контента специальные символы в HTML-коде должны быть экранированы. В чистом
PHP это осуществляется с помощью функций [htmlspecialchars](https://www.php.net/manual/ru/function.htmlspecialchars.php)
и [htmlspecialchars_decode](https://www.php.net/manual/ru/function.htmlspecialchars-decode.php). Проблема использования
этих функций заключается в том, что приходится указывать кодировку и дополнительные флаги во время каждого вызова.
Поскольку флаги всё время одинаковы, а кодировка остаётся одной и той же в пределах приложения, Yii в целях
безопасности предоставляет два компактных и простых в использовании метода:

```php
$userName = Html::encode($user->name);
echo $userName;

$decodedUserName = Html::decode($userName);
```


## Формы <span id="forms"></span>

Разметка форм состоит из повторяющихся действий и часто приводит к ошибкам, поэтому есть целый набор методов, которые
помогают справиться с этой задачей.

> Note: Рассмотрите возможность использования [[yii\widgets\ActiveForm|ActiveForm]], если работаете с моделями и
  нуждаетесь в валидации данных.


### Создание форм <span id="creating-forms"></span>

Открывающий тэг формы может быть выведен с помощью метода [[yii\helpers\Html::beginForm()|beginForm()]] как показано
ниже:

```php
<?= Html::beginForm(['order/update', 'id' => $id], 'post', ['enctype' => 'multipart/form-data']) ?>
```

Первый аргумент - это URL-адрес, на который будет отправлена форма. Он может быть задан в виде Yii-маршрута и
параметров, подходящих для передачи в метод [[yii\helpers\Url::to()|Url::to()]]. Второй аргумент - способ отправки
данных: по умолчанию это `post`. Третий аргумент - массив атрибутов формы. В данном примере мы изменяем способ
кодирования данных в POST-запросе на `multipart/form-data`. Это необходимо для загрузки файлов.

Закрыть тэг формы можно простым кодом:

```php
<?= Html::endForm() ?>
```


### Кнопки <span id="buttons"></span>

Для формирования кнопок можно использовать следующий код:

```php
<?= Html::button('Нажми меня!', ['class' => 'teaser']) ?>
<?= Html::submitButton('Отправить', ['class' => 'submit']) ?>
<?= Html::resetButton('Сбросить', ['class' => 'reset']) ?>
```

Первый аргумент во всех трёх методах - это название кнопки, а второй - её атрибуты. Название кнопки не экранируется,
поэтому при получении данных от конечных пользователей экранируйте их с помощью метода
[[yii\helpers\Html::encode()|Html::encode()]].


### Поля ввода <span id="input-fields"></span>

Методы формирования полей ввода делятся на две группы. Первые начинаются со слова `active` и называются "active inputs",
а вторые не содержат в своём названии слова `active`. "Active inputs" формируют поля ввода, которые получают данные из
указанного атрибута модели, в то время как обычные методы формирования полей ввода непосредственно принимают данные.

Наиболее общие методы для формирования полей ввода:

```php
// тип, название поля ввода, установленное в поле значение, атрибуты поля ввода
<?= Html::input('text', 'username', $user->name, ['class' => $username]) ?>

// тип, модель, атрибут модели, атрибуты поля ввода
<?= Html::activeInput('text', $user, 'name', ['class' => $username]) ?>
```

Если вам заранее известен тип поля ввода, удобнее будет пользоваться этими вспомогательными методами:

- [[yii\helpers\Html::buttonInput()]]
- [[yii\helpers\Html::submitInput()]]
- [[yii\helpers\Html::resetInput()]]
- [[yii\helpers\Html::textInput()]], [[yii\helpers\Html::activeTextInput()]]
- [[yii\helpers\Html::hiddenInput()]], [[yii\helpers\Html::activeHiddenInput()]]
- [[yii\helpers\Html::passwordInput()]] / [[yii\helpers\Html::activePasswordInput()]]
- [[yii\helpers\Html::fileInput()]], [[yii\helpers\Html::activeFileInput()]]
- [[yii\helpers\Html::textarea()]], [[yii\helpers\Html::activeTextarea()]]

Сигнатура методов для формирования радио-переключателей и чекбоксов немного отличается: 

```php
<?= Html::radio('agree', true, ['label' => 'Я согласен']) ?>
<?= Html::activeRadio($model, 'agree', ['class' => 'agreement']) ?>

<?= Html::checkbox('agree', true, ['label' => 'Я согласен']) ?>
<?= Html::activeCheckbox($model, 'agree', ['class' => 'agreement']) ?>
```

Выпадающие и обычные списки могут быть сформированы следующим образом:

```php
<?= Html::dropDownList('list', $currentUserId, ArrayHelper::map($userModels, 'id', 'name')) ?>
<?= Html::activeDropDownList($users, 'id', ArrayHelper::map($userModels, 'id', 'name')) ?>

<?= Html::listBox('list', $currentUserId, ArrayHelper::map($userModels, 'id', 'name')) ?>
<?= Html::activeListBox($users, 'id', ArrayHelper::map($userModels, 'id', 'name')) ?>
```

Первый аргумент - это значение атрибута `name` для поля ввода, второй - выбранное в нём значение и, наконец, 
третий аргумент - это массив, ключами которого является список значений для формирования списка, а значениями массива
являются названия пунктов в нём.

Если вы хотите сделать в списке доступными для выбора несколько вариантов, хорошим выбором будет список чекбоксов:

```php
<?= Html::checkboxList('roles', [16, 42], ArrayHelper::map($roleModels, 'id', 'name')) ?>
<?= Html::activeCheckboxList($user, 'role', ArrayHelper::map($roleModels, 'id', 'name')) ?>
```

Если нет, используйте радио-переключатель:

```php
<?= Html::radioList('roles', [16, 42], ArrayHelper::map($roleModels, 'id', 'name')) ?>
<?= Html::activeRadioList($user, 'role', ArrayHelper::map($roleModels, 'id', 'name')) ?>
```


### Тэги label и отображение ошибок <span id="labels-and-errors"></span>

Так же как и для полей ввода, есть два метода формирования тэгов label для форм. Есть "active label", считывающий
данные из модели и обычный тэг "label", принимающий на вход непосредственно сами данные:

```php
<?= Html::label('Имя пользователя', 'username', ['class' => 'label username']) ?>
<?= Html::activeLabel($user, 'username', ['class' => 'label username'])
```

Для отображения общего списка ошибок формы на основе модели или моделей можно использовать следующий подход:

```php
<?= Html::errorSummary($posts, ['class' => 'errors']) ?>
```

Для отображения одной отдельной ошибки:

```php
<?= Html::error($post, 'title', ['class' => 'error']) ?>
```


### Атрибуты name и value для полей ввода <span id="input-names-and-values"></span>

Также имеются методы для получения значений атрибутов id, name и value для полей ввода, сформированных на основе
моделей. Эти методы используются в основном внутренними механизмами, но иногда могут оказаться подходящими и для прямого
использования:

```php
// Post[title]
echo Html::getInputName($post, 'title');

// post-title
echo Html::getInputId($post, 'title');

// моё первое сообщение
echo Html::getAttributeValue($post, 'title');

// $post->authors[0]
echo Html::getAttributeValue($post, '[0]authors[0]');
```

В вышеприведённом коде первый аргумент - это модель, а второй аргумент - выражение для поиска атрибута модели. В самом
простом случае оно представляет собой название атрибута модели, а вообще это может быть название, которому предшествует
(и/или после которого следует) индекс массива. Такой формат используется в основном для табличного ввода данных:

- `[0]content` используется для табличного ввода данных, чтобы указать на атрибут "content" первой модели табличного
  ввода;
- `dates[0]` указывает на первый элемент массива, с помощью которого задан атрибут модели "dates";
- `[0]dates[0]` указывает на первый элемент массива, с помощью которого задан атрибут "dates" первой модели табличного
  ввода.

Для того, чтобы получить название атрибута модели в чистом виде (без суффиксов и префиксов), можно использовать
следующий подход:

```php
// dates
echo Html::getAttributeName('dates[0]');
```


## Подключение встроенных стилей и скриптов <span id="styles-and-scripts"></span>

Для формирования встроенных скриптов и стилей есть два метода:

```php
<?= Html::style('.danger { color: #f00; }', ['media' => 'print']) ?>

Результатом будет:

<style media="print">.danger { color: #f00; }</style>


<?= Html::script('alert("Привет!");') ?>

Результатом будет:

<script>alert("Привет!");</script>
```

Если вы хотите подключить внешний CSS-файл:

```php
<?= Html::cssFile('@web/css/ie5.css', ['condition' => 'IE 5']) ?>

В результате получится:

<!--[if IE 5]>
    <link href="http://example.com/css/ie5.css" />
<![endif]-->
```

Первый аргумент - URL-адрес. Второй - массив настроек. Помимо обычных настроек можно указать следующие:

- `condition` для оборачивания тэга `<link` с помощью условных комментариев с определённым условием. Надеемся, что вам
  никогда не понадобятся условные комментарии ;)
- `noscript` может быть установлен в значение `true`, чтобы обернуть тэг `<link` с помощью тэга `<noscript>`, таким
  образом скрипт будет подключён только в том случае, если у пользователя в браузере нет поддержки JavaScript или же 
  пользователь сам отключил его.

Для подключения JavaScript-файла используйте код:

```php
<?= Html::jsFile('@web/js/main.js') ?>
```

Как и в случае с CSS, первый аргумент указывает ссылку на файл, который должен быть подключен. Настройки задаются во
втором аргументе. Можно указать настройку `condition` таким же образом, каким она указывается для метода `cssFile`.


## Ссылки <span id="hyperlinks"></span>

Существует удобный метод формирования ссылок:

```php
<?= Html::a('Профиль', ['user/view', 'id' => $id], ['class' => 'profile-link']) ?>
```

Первый аргумент - это текст ссылки. Он не экранируется, поэтому при использовании данных, полученных от конечных 
пользователей, необходимо экранировать его с помощью `Html::encode()`. Второй аргумент - это содержимое атрибута `href`
тэга `<a`. Смотрите [Url::to()](helper-url.md) для получения подробностей относительно значений, которые могут быть
переданы в качестве второго аргумента. Третий аргумент - массив атрибутов для тэга.

Если нужно сформировать ссылку `mailto`, можно использовать следующий подход:

```php
<?= Html::mailto('Обратная связь', 'admin@example.com') ?>
```


## Изображения <span id="images"></span>

Для формирования тэга изображения используйте следующий код:

```php
<?= Html::img('@web/images/logo.png', ['alt' => 'Наш логотип']) ?>

в результате получится:

<img src="http://example.com/images/logo.png" alt="Наш логотип" />
```

Помимо [псевдонимов](concept-aliases.md) первый аргумент может принимать маршруты, параметры и URL-адреса таким же образом
как и метод [Url::to()](helper-url.md).


## Списки <span id="lists"></span>

Неупорядоченные списки могут быть сформированы следующим образом:

```php
<?= Html::ul($posts, ['item' => function($item, $index) {
    return Html::tag(
        'li',
        $this->render('post', ['item' => $item]),
        ['class' => 'post']
    );
}]) ?>
```

Для формирования упорядоченных списков используйте `Html::ol()`.
