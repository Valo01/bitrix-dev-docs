# Стандартные компоненты: что для чего

Справочник по стандартным `bitrix:*` компонентам. Шаблоны и URL — **свойство проекта**; ниже — общие правила выбора.

---

## Контент из инфоблоков

### `bitrix:news` — комплексный раздел

**Когда:** раздел со списком элементов и детальными страницами по ЧПУ (новости, статьи, врачи, вакансии, проекты).

Один вызов на `index.php` раздела:

```php
$APPLICATION->IncludeComponent('bitrix:news', '<шаблон>', [
    'IBLOCK_ID' => $iblockId,
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/news/',
    'SEF_URL_TEMPLATES' => [
        'news' => '',
        'section' => '#SECTION_CODE#/',
        'detail' => '#ELEMENT_CODE#/',
    ],
    'CACHE_TYPE' => 'A',
    'CACHE_TIME' => 36000000,
]);
```

Переопределения: `local/templates/<шаблон>/components/bitrix/news/<шаблон>/` (`news.php`, `detail.php`, `section.php`).

---

### `bitrix:catalog` — каталог с разделами

**Когда:** иерархия разделов + элементы с ЧПУ (услуги, товары — при установленном модуле `catalog`).

```php
$APPLICATION->IncludeComponent('bitrix:catalog', '<шаблон>', [
    'IBLOCK_ID' => $iblockId,
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/catalog/',
    'SEF_URL_TEMPLATES' => [
        'sections' => '',
        'section' => '#SECTION_CODE_PATH#/',
        'element' => '#SECTION_CODE_PATH#/#ELEMENT_CODE#/',
    ],
]);
```

Не путать с `bitrix:news`: каталог — когда нужны разделы каталога и типовая логика `catalog.section` / `catalog.element`.

---

### `bitrix:news.list` — список или блок

**Когда:** один блок на странице — слайдер, подборка, соцсети, FAQ, виджет в шапке.

```php
$APPLICATION->IncludeComponent('bitrix:news.list', '<шаблон>', [
    'IBLOCK_ID' => $iblockId,
    'NEWS_COUNT' => 10,
    'SORT_BY1' => 'SORT',
    'SORT_ORDER1' => 'ASC',
    'SET_TITLE' => 'N',
    'CACHE_TYPE' => 'A',
    'CACHE_TIME' => 36000000,
]);
```

---

### `bitrix:news.detail` — детальная страница

Обычно **не вызывается** с `index.php` — подключается комплексным `bitrix:news` или `bitrix:catalog` через `detail.php` / `element.php`.

Переопределение: `local/templates/<шаблон>/components/bitrix/news.detail/<шаблон>/`.

---

### `bitrix:form.result.new` — веб-форма

**Когда:** форма обратной связи, заявка, запись.

```php
$APPLICATION->IncludeComponent('bitrix:form.result.new', '<шаблон>', [
    'WEB_FORM_ID' => $formId,  // из конфига
    'CACHE_TYPE' => 'N',
]);
```

Глобальные модальные формы часто подключают в `footer.php`. Альтернатива — кастомная форма + `local/ajax/form.php`.

---

## Навигация и оболочка

### `bitrix:menu`

```php
$APPLICATION->IncludeComponent('bitrix:menu', '<шаблон>', [
    'ROOT_MENU_TYPE' => 'top',
    'CHILD_MENU_TYPE' => 'sub_top',
    'MAX_LEVEL' => 2,
    'MENU_CACHE_TYPE' => 'A',
    'MENU_CACHE_TIME' => 36000000,
]);
```

Типы меню настраиваются в админке: **Контент → Типы меню**.

---

### `bitrix:breadcrumb`

На внутренних страницах — в `index.php`, не в `header.php`:

```php
$APPLICATION->IncludeComponent('bitrix:breadcrumb', '', [
    'START_FROM' => '0',
    'PATH' => '',
    'SITE_ID' => SITE_ID,
]);
```

---

### `bitrix:main.include`

```php
$APPLICATION->IncludeComponent('bitrix:main.include', '', [
    'AREA_FILE_SHOW' => 'file',
    'PATH' => '/include/footer/about.php',
]);
```

Подробнее — [../include-areas.md](../include-areas.md).

---

## Быстрый выбор

| Задача | Компонент |
|--------|-----------|
| Раздел с ЧПУ (список + детальные) | `bitrix:news` |
| Каталог с разделами | `bitrix:catalog` |
| Блок / список на странице | `bitrix:news.list` |
| Меню | `bitrix:menu` |
| Хлебные крошки | `bitrix:breadcrumb` |
| Текстовый фрагмент | `bitrix:main.include` |
| Форма | `bitrix:form.result.new` |
| Уникальный UI с логикой | свой компонент (см. [custom-components.md](custom-components.md)) |

## См. также

- [types.md](types.md)
- [pages.md](pages.md)
- [header-footer.md](header-footer.md)
- [custom-components.md](custom-components.md)
