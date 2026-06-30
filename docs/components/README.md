# Компоненты: правила и подключение

Общие правила работы с компонентами в Bitrix-проектах команды.

> Сначала: [getting-started.md](../getting-started.md) → [structure.md](../structure.md) → [configuration.md](../configuration.md)

## Документы

| Документ | Содержание |
|----------|------------|
| [types.md](types.md) | Простые, комплексные, служебные |
| [standard-components.md](standard-components.md) | Какой `bitrix:*` для какой задачи |
| [include-rules.md](include-rules.md) | `IncludeComponent`, кеш, SEF |
| [pages.md](pages.md) | Подключение в `index.php` |
| [header-footer.md](header-footer.md) | `header.php` / `footer.php` |
| [templates.md](templates.md) | `template.php`, `result_modifier` |
| [custom-components.md](custom-components.md) | Свои компоненты и модули |

## Быстрая шпаргалка

```php
// Список на странице
$APPLICATION->IncludeComponent('bitrix:news.list', 'main', [
    'IBLOCK_ID' => $iblockId,  // из конфига проекта
    'FIELD_CODE' => ['NAME', 'PREVIEW_TEXT', 'PREVIEW_PICTURE', 'DATE_ACTIVE_FROM'],
    'PROPERTY_CODE' => [],  // коды свойств, которые нужны в template.php
]);

// Комплексный раздел (новости, статьи, каталог статей)
$APPLICATION->IncludeComponent('bitrix:news', '', [
    'IBLOCK_ID' => $iblockId,
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/news/',
]);

// Каталог с разделами (услуги, товары — если есть модуль catalog)
$APPLICATION->IncludeComponent('bitrix:catalog', '', [
    'IBLOCK_ID' => $iblockId,
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/catalog/',
]);

// Редактируемая область
$APPLICATION->IncludeComponent('bitrix:main.include', '', [
    'AREA_FILE_SHOW' => 'file',
    'PATH' => '/include/about/banner.php',
]);

// Меню
$APPLICATION->IncludeComponent('bitrix:menu', 'top', [
    'ROOT_MENU_TYPE' => 'top',
    'MAX_LEVEL' => 2,
]);
```

Подробнее — [header-footer.md](header-footer.md).
