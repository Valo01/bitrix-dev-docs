# Подключение компонентов на страницах

Где вызывать компоненты в структуре сайта.

> Шапка и подвал — [header-footer.md](header-footer.md)

## Точки подключения

```
www/
├── index.php                 # главная
├── <раздел>/index.php        # страница раздела
├── include/                  # включаемые области (если есть)
└── local/templates/<шаблон>/
    ├── header.php
    └── footer.php
```

| Файл | Что подключать |
|------|----------------|
| `index.php` | Основной контент страницы |
| `header.php` | Меню, глобальные виджеты |
| `footer.php` | Меню, формы, юр. блоки |
| `include/*.php` | Фрагменты через `main.include` |

## Структура типовой страницы

```php
<?php
require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/header.php';

$APPLICATION->SetTitle('Заголовок');
$APPLICATION->SetPageProperty('description', '...');
?>

<?php
$APPLICATION->IncludeComponent('bitrix:breadcrumb', '', [
    'START_FROM' => '0',
    'SITE_ID' => SITE_ID,
]);
?>

<!-- компоненты контента -->

<?php require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/footer.php'; ?>
```

## Сценарии страниц

### Главная — несколько блоков

Допустимо несколько независимых `news.list` и `main.include`:

```php
$APPLICATION->IncludeComponent('bitrix:news.list', 'banner', [
    'IBLOCK_ID' => $iblockBanners,
    'CACHE_TYPE' => 'A',
    'CACHE_TIME' => 36000000,
]);

$APPLICATION->IncludeComponent('bitrix:news.list', 'news_main', [
    'IBLOCK_ID' => $iblockNews,
    'NEWS_COUNT' => 6,
    'CACHE_TYPE' => 'A',
    'CACHE_TIME' => 36000000,
]);
```

### Раздел с ЧПУ — один комплексный компонент

```php
$APPLICATION->IncludeComponent('bitrix:news', '<шаблон>', [
    'IBLOCK_ID' => $iblockId,
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/news/',
    'CACHE_TYPE' => 'A',
    'CACHE_TIME' => 36000000,
]);
```

### Лендинг — много блоков

`main.include` + несколько `news.list` + свой компонент при необходимости. Каждый блок — отдельный `IncludeComponent`.

### AJAX-подгрузка

Логика в `local/ajax/*.php`: `prolog_before.php`, `check_bitrix_sessid()`, JSON-ответ. Компонент на странице отдаёт первую порцию, AJAX — следующие.

---

## Комплексные разделы — одна точка входа

```
/news/index.php     → bitrix:news (один вызов)
/catalog/index.php  → bitrix:catalog (один вызов)
```

**Неправильно:**

```
/news/index.php     → news.list
/news/detail.php    → news.detail
```

Комплексный компонент сам определяет шаблон по URL.

---

## Include-области

```php
$APPLICATION->IncludeComponent('bitrix:main.include', '', [
    'AREA_FILE_SHOW' => 'file',
    'PATH' => '/include/about/banner.php',
]);
```

Подробнее — [../include-areas.md](../include-areas.md).

## Подключение в шаблоне компонента

```php
// template.php родителя
$APPLICATION->IncludeComponent(
    'bitrix:news.list',
    'related',
    ['IBLOCK_ID' => $iblockId, ...],
    $component
);
```

Четвёртый параметр `$component` обязателен для кеша и AJAX.

---

## Размещение: сводная таблица

| Компонент | index.php | header | footer | Внутри шаблона |
|-----------|:---------:|:------:|:------:|:--------------:|
| `bitrix:news` / `bitrix:catalog` | ✅ | ❌ | ❌ | ❌ |
| `bitrix:news.list` | ✅ | ✅ | ✅ | ✅ |
| `bitrix:menu` | ❌ | ✅ | ✅ | ❌ |
| `bitrix:breadcrumb` | ✅ | ❌ | ❌ | ❌ |
| `bitrix:main.include` | ✅ | ✅ | ✅ | ❌ |
| `bitrix:form.result.new` | ✅ | ✅ | ✅ | ❌ |
| `bitrix:system.pagenavigation` | ❌ | ❌ | ❌ | ✅ |

## См. также

- [header-footer.md](header-footer.md)
- [standard-components.md](standard-components.md)
- [include-rules.md](include-rules.md)
