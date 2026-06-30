# Интеграция header.php и footer.php

Правила подключения компонентов в шапке и подвале. Применимо к любому шаблону в `local/templates/<шаблон>/`.

## Роли файлов

| Файл | Ответственность |
|------|-----------------|
| `header.php` | `<html>`, `<head>`, CSS/JS, открытие `<body>`, шапка |
| `footer.php` | Подвал, модалки, формы, счётчики, закрытие `</body></html>` |
| `index.php` | Контент **конкретной** страницы |

```php
require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/header.php';
// ... контент index.php ...
require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/footer.php';
```

## Что подключать в header.php

### Допустимо (глобальное, на всех страницах)

| Элемент | Компонент / подход |
|---------|-------------------|
| Меню | `bitrix:menu` |
| Логотип | `main.include` или статичная разметка + файл из настроек |
| Телефон | `main.include` или `COption` / модуль настроек |
| CSS/JS | `Asset::getInstance()` |
| Админ-панель | `$APPLICATION->ShowPanel()` |

```php
$APPLICATION->IncludeComponent('bitrix:menu', 'top', [
    'ROOT_MENU_TYPE' => 'top',
    'CHILD_MENU_TYPE' => 'sub_top',
    'MAX_LEVEL' => 2,
    'MENU_CACHE_TYPE' => 'A',
    'MENU_CACHE_TIME' => 36000000,
]);
```

### Условно допустимо

- Небольшой `news.list` в шапке (соцсети, контакты филиалов) — с кешем
- Подключение скриптов по условию URL

### Не подключать в header.php

- `bitrix:news`, `bitrix:catalog` — контент страницы
- `bitrix:breadcrumb` — зависит от раздела
- `bitrix:news.list` с контентом конкретной страницы
- Тяжёлая бизнес-логика

---

## Что подключать в footer.php

### Допустимо (глобальное)

| Элемент | Компонент |
|---------|-----------|
| Меню подвала | `bitrix:menu` |
| Телефон, адрес, юр. текст | `main.include` или настройки |
| Соцсети | `bitrix:news.list` |
| Формы (модалки) | `bitrix:form.result.new` |
| Аналитика | `<script>` перед `</body>` |

### Не подключать в footer.php

- Основной контент страницы
- `bitrix:breadcrumb`
- Комплексные разделы

---

## Что оставлять в index.php

| Элемент | Почему |
|---------|--------|
| `bitrix:breadcrumb` | Не нужен на главной; путь разный |
| `bitrix:news` / `bitrix:catalog` | Основной контент раздела |
| `bitrix:news.list` | Блоки страницы |
| `bitrix:main.include` | Блоки, специфичные для раздела |
| `SetTitle`, `SetPageProperty` | SEO страницы |

**Пример лендинга:**

```php
require '/bitrix/header.php';
$APPLICATION->SetTitle('О компании');

$APPLICATION->IncludeComponent('bitrix:breadcrumb', '', [...]);

$APPLICATION->IncludeComponent('bitrix:main.include', '', [
    'PATH' => '/include/about/banner.php',
]);

$APPLICATION->IncludeComponent('bitrix:news.list', 'team', [
    'IBLOCK_ID' => $iblockId,
    'CACHE_TYPE' => 'A',
    'CACHE_TIME' => 36000000,
]);

require '/bitrix/footer.php';
```

---

## Сводная таблица

| Компонент | header | footer | index.php |
|-----------|:------:|:------:|:---------:|
| `bitrix:menu` | ✅ | ✅ | ❌ |
| `bitrix:main.include` (logo, phone) | ✅ | ✅ | ✅ |
| `bitrix:news.list` (soc_links) | ✅ | ✅ | ✅ |
| `bitrix:breadcrumb` | ❌ | ❌ | ✅ |
| `bitrix:news` / `bitrix:catalog` | ❌ | ❌ | ✅ |
| `bitrix:form.result.new` | ❌ | ✅ | ✅ |
| Модалки общие | ❌ | ✅ | ❌ |

---

## Пример: корпоративный сайт

На одном из проектов команды в шапке: `news.list` (клиники, соцсети), меню, телефон из `askaron.settings`. В подвале: меню, `main.include` (юр. текст), несколько `form.result.new` в модалках, заказ звонка через `local/ajax/form.php`. Это **не шаблон для всех** — сверяйтесь с `header.php` / `footer.php` вашего репозитория.

## Правила для нового кода

1. Глобальный блок → `header.php` или `footer.php`
2. Блок только для раздела → `index.php` или `include/<раздел>/`
3. Не раздувать `header.php` — вынести в `include` + `main.include`
4. Счётчики — в `footer.php` перед `</body>`

## См. также

- [pages.md](pages.md)
- [standard-components.md](standard-components.md)
- [../configuration.md](../configuration.md)
