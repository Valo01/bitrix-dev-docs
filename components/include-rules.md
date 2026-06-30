# Правила подключения компонентов

Синтаксис и технические правила вызова `IncludeComponent`.

> **Где подключать** — [pages.md](pages.md), [header-footer.md](header-footer.md).

## Базовый вызов

```php
<?php
$APPLICATION->IncludeComponent(
    'bitrix:news.list',
    'main',
    [
        'IBLOCK_ID' => $iblockId,
        'NEWS_COUNT' => 10,
        'SORT_BY1' => 'ACTIVE_FROM',
        'SORT_ORDER1' => 'DESC',
        'FIELD_CODE' => ['NAME', 'PREVIEW_TEXT', 'PREVIEW_PICTURE', 'DATE_ACTIVE_FROM'],
        'PROPERTY_CODE' => [],  // коды свойств, которые нужны в template.php
        'CACHE_TYPE' => 'A',
        'CACHE_TIME' => 36000000,
        'CACHE_GROUPS' => 'Y',
    ],
    false,
    ['HIDE_ICONS' => 'Y']
);
```

## Обязательные правила

### 1. Только через IncludeComponent

**Неправильно:**

```php
require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/components/bitrix/news.list/component.php';
include 'templates/news/template.php';
```

### 2. Параметры — в массиве вызова

Фильтры и ID — в `IncludeComponent`, не в `template.php`.

### 3. Кеш на публичных страницах

```php
'CACHE_TYPE' => 'A',
'CACHE_TIME' => 36000000,
'CACHE_GROUPS' => 'Y',
```

Исключение: персональный контент, корзина — `CACHE_TYPE` => `N` или `CACHE_GROUPS` => `Y`.

### 4. ID — из конфигурации проекта

```php
// Правильно
'IBLOCK_ID' => $_ENV['IBLOCK_ID_NEWS'],
'IBLOCK_ID' => IBLOCK_NEWS_ID,
'IBLOCK_ID' => \Project\IBlock::get('news'),

// Неправильно
'IBLOCK_ID' => 12,
```

Способ хранения — см. [configuration.md](../configuration.md).

## Выбор шаблона

Приоритет: `local/templates/<шаблон>/components/` → `local/components/` → ядро.

Перед созданием нового шаблона проверьте существующие переопределения в проекте.

## Вложенные компоненты

```php
$APPLICATION->IncludeComponent(
    'bitrix:system.pagenavigation',
    '',
    [],
    $component,
    ['HIDE_ICONS' => 'Y']
);
```

## SEF (ЧПУ)

```php
'SEF_MODE' => 'Y',
'SEF_FOLDER' => '/news/',
'SEF_URL_TEMPLATES' => [
    'news' => '',
    'section' => '#SECTION_CODE#/',
    'detail' => '#ELEMENT_CODE#/',
],
```

После изменения — проверьте `urlrewrite.php`.

## См. также

- [../configuration.md](../configuration.md)
- [standard-components.md](standard-components.md)
- [pages.md](pages.md)
- [templates.md](templates.md)
