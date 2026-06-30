# Шаблоны компонентов

Работа с `template.php`, `result_modifier.php` и `component_epilog.php`.

Переопределения: `local/templates/<шаблон>/components/bitrix/<компонент>/<шаблон>/`.

## Файлы шаблона

```
templates/<шаблон>/components/bitrix/<компонент>/<имя_шаблона>/
├── template.php           # вёрстка
├── result_modifier.php    # модификация arResult перед выводом
├── component_epilog.php   # код после template (скрипты, мета)
├── style.css
├── script.js
├── .parameters.php        # параметры шаблона (опционально)
└── lang/
    └── ru/
        └── template.php
```

## template.php — только вёрстка

```php
<?php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) {
    die();
}

/** @var array $arResult */
/** @var array $arParams */
?>

<ul class="news-list">
<?php foreach ($arResult['ITEMS'] as $item): ?>
    <li>
        <a href="<?= htmlspecialcharsbx($item['DETAIL_PAGE_URL']) ?>">
            <?= htmlspecialcharsbx($item['NAME']) ?>
        </a>
    </li>
<?php endforeach; ?>
</ul>
```

В шаблоне **не должно быть**:

- SQL-запросов и `GetList`
- вызовов API с побочными эффектами (создание, удаление)
- тяжёлой бизнес-логики

## result_modifier.php — подготовка данных

Используйте для форматирования, обогащения и фильтрации `arResult`:

```php
<?php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) {
    die();
}

foreach ($arResult['ITEMS'] as &$item) {
    $item['DISPLAY_DATE'] = FormatDate('j F Y', MakeTimeStamp($item['ACTIVE_FROM']));
    $item['PREVIEW_TEXT_SHORT'] = TruncateText($item['PREVIEW_TEXT'], 200);
}
unset($item);
```

Подходит для:

- форматирования дат и цен
- добавления вычисляемых полей
- объединения данных из разных источников (через сервис)

Не подходит для тяжёлой логики — выносите в сервис и вызывайте отсюда.

## component_epilog.php — после вывода

Для подключения скриптов, установки заголовков, Open Graph:

```php
<?php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) {
    die();
}

use Bitrix\Main\Page\Asset;

Asset::getInstance()->addJs($templateFolder . '/script.js');
```

## Переопределение стандартного компонента

Чтобы изменить только вёрстку `bitrix:news.list`, не копируйте весь компонент. Достаточно шаблона:

```
local/templates/<шаблон>/components/bitrix/news.list/<имя>/
├── template.php
├── result_modifier.php
└── component_epilog.php    # опционально
```

Копировать `component.php` из ядра — только если нужно изменить логику выборки.

## Типичные приёмы

### `result_modifier.php`

Форматирование дат, группировка элементов, подготовка полей перед выводом. Без SQL и тяжёлой логики.

### `component_epilog.php`

Подключение `script.js`, установка мета-тегов после рендера.

### Глобальный флаг для условного блока

```php
// В news.php / section.php комплексного компонента:
$GLOBALS['SHOW_SEO_BLOCK'] = true;

// В index.php раздела:
if (!empty($GLOBALS['SHOW_SEO_BLOCK'])) {
    $APPLICATION->IncludeComponent('bitrix:main.include', '', [...]);
}
```

### `AddViewContent`

Вставка блока из `index.php` в нужное место шаблона каталога или новостей:

```php
// index.php
$APPLICATION->AddViewContent('page_seo', $html);

// section.php / template.php
$APPLICATION->ShowViewContent('page_seo');
```

## Модификация arParams

В `result_modifier.php` можно менять `arResult`, но не `arParams` (они уже обработаны). Параметры настраивайте в `.parameters.php` и вызове `IncludeComponent`.

## Экранирование

| Контекст | Функция |
|----------|---------|
| HTML | `htmlspecialcharsbx()` или `HtmlFilter::encode()` |
| JavaScript | `CUtil::JSEscape()` |
| URL | `urlencode()` / `CHTTP::urlAddParams()` |
| Атрибут HTML | `htmlspecialcharsbx($val, ENT_QUOTES)` |

## См. также

- [include-rules.md](include-rules.md)
- [standard-components.md](standard-components.md)
- [structure.md](../structure.md)
