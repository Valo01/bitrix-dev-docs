# Типы компонентов

Классификация компонентов Bitrix — общая для всех проектов.

## Простые компоненты

Выполняют **одну задачу**. На странице может быть несколько вызовов.

| Компонент | Задача |
|-----------|--------|
| `bitrix:news.list` | Список элементов инфоблока |
| `bitrix:news.detail` | Одна детальная (обычно внутри комплексного) |
| `bitrix:menu` | Навигация |
| `bitrix:breadcrumb` | Хлебные крошки |
| `bitrix:main.include` | Подключение PHP/HTML-файла |
| `bitrix:form.result.new` | Веб-форма |

```php
$APPLICATION->IncludeComponent('bitrix:news.list', 'main', [
    'IBLOCK_ID' => $iblockId,
    'FIELD_CODE' => ['NAME', 'PREVIEW_TEXT', 'PREVIEW_PICTURE', 'DATE_ACTIVE_FROM'],
    'PROPERTY_CODE' => [],  // коды свойств, которые нужны в template.php
]);
```

## Комплексные компоненты

Один компонент **маршрутизирует URL** раздела.

| Компонент | Когда |
|-----------|-------|
| `bitrix:news` | Список + детальные без торговой логики |
| `bitrix:catalog` | Разделы + элементы (каталог, услуги) |

**Правило:** на `index.php` раздела — **один** вызов, не пара `news.list` + `news.detail`.

```php
$APPLICATION->IncludeComponent('bitrix:news', '<шаблон>', [
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/news/',
    'IBLOCK_ID' => $iblockId,
]);
```

Шаблоны страниц:

```
local/templates/<шаблон>/components/bitrix/news/<шаблон>/
├── news.php
├── detail.php
└── section.php   # опционально
```

## Служебные и вложенные

| Компонент | Где |
|-----------|-----|
| `bitrix:system.pagenavigation` | внутри шаблонов списков |
| вложенные `bitrix:news.list` | похожие элементы, документы |
| вложенные `bitrix:main.include` | фрагменты в детальных |


## Схема выбора типа

```
Нужен целый раздел с ЧПУ (список + детальные)?
  ├─ С разделами каталога → bitrix:catalog
  └─ Без каталога → bitrix:news

Нужен один список или блок на странице?
  └─ bitrix:news.list

Нужен статичный HTML?
  └─ bitrix:main.include + include/

Нужен переиспользуемый блок с параметрами?
  └─ свой компонент в local/components/

Меню / крошки / формы?
  └─ bitrix:menu, bitrix:breadcrumb, form.result.new
```

## См. также

- [standard-components.md](standard-components.md)
- [pages.md](pages.md)
- [custom-components.md](custom-components.md)
