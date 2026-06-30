# Структура проекта

Где лежит код в Bitrix-проекте и что можно менять. Применимо к **любому** проекту команды.

## Что нельзя трогать

| Путь | Правило |
|------|---------|
| `bitrix/` | Ядро CMS — не редактируем |
| `upload/` | Файлы пользователей |
| `bitrix/cache/`, `bitrix/managed_cache/` | Служебный кеш |
| `.env`, ключи и пароли окружения | Не в Git (есть `.env.example` или аналог) |

## Типовая структура корня сайта

```
www/
├── index.php                 # главная
├── <раздел>/index.php        # страницы разделов
├── include/                  # включаемые области (если принято в проекте)
├── local/                    # весь кастомный код
├── bitrix/                   # ядро (не трогаем)
└── upload/                   # загруженные файлы
```

Разделы (`/news/`, `/catalog/`, `/about/` и т.д.) — у каждого проекта свои. Точка входа раздела — всегда `index.php` в папке раздела.

Рядом может лежать **`.section.php`** — служебный файл раздела, не страница для пользователя. Bitrix подключает его при заходе в папку **до** вывода контента из `index.php`. В нём задают общие свойства всего раздела:

- **`$sSectionName`** — название для хлебных крошек и заголовков;
- **`$arDirProperties`** — SEO и свойства каталога раздела: `title`, `description`, `keywords`, `MENU` (показывать в меню) и др.

Пример `/about/.section.php`:

```php
<?php
$sSectionName = 'О компании';
$arDirProperties = [
    'title' => 'О компании',
    'description' => 'Краткое описание раздела для meta description',
    'keywords' => 'о компании, контакты',
    'MENU' => 'Y',
];
```

Свойства из `.section.php` действуют на **весь раздел**, пока конкретная страница не переопределит их в своём `index.php` через `$APPLICATION->SetTitle()` или `SetPageProperty()`.

## Структура `local/`

```
local/
├── ajax/                     # AJAX-endpoint'ы (опционально)
├── components/               # свои компоненты (vendor:component)
│   └── <vendor>/<name>/
├── modules/                  # свои модули (опционально)
├── php_interface/
│   ├── init.php              # точка входа
│   ├── constants.php         # константы (опционально)
│   ├── classes/              # классы, хелперы
│   └── include/              # urlrewrites, фильтры, подключаемые файлы
└── templates/
    └── <шаблон_сайта>/
        ├── header.php
        ├── footer.php
        └── components/       # переопределения bitrix-компонентов
            └── bitrix/
```

Не каждый проект содержит все папки. Перед задачей откройте `local/` и посмотрите, что уже есть.

## Приоритет поиска шаблона компонента

Когда Bitrix рендерит, например, `bitrix:news.list` с шаблоном `main`, ищет файлы в таком порядке:

1. `local/templates/<шаблон>/components/bitrix/news.list/main/`
2. `local/components/bitrix/news.list/templates/main/`
3. `bitrix/templates/<шаблон>/components/...`
4. `bitrix/components/bitrix/news.list/templates/main/`

**Правило:** перед созданием нового шаблона проверьте п.1 — в проектах команды почти все переопределения лежат там.

## Где размещать изменения

| Задача | Куда |
|--------|------|
| Вёрстка списка | `local/templates/<шаблон>/components/bitrix/news.list/<шаблон>/` |
| Комплексный раздел | `local/templates/<шаблон>/components/bitrix/news/<шаблон>/` |
| Каталог (если есть) | `local/templates/<шаблон>/components/bitrix/catalog/<шаблон>/` |
| Шапка / подвал | `local/templates/<шаблон>/header.php`, `footer.php` |
| Редактируемый HTML | `include/` или `local/templates/.../include/` + `main.include` |
| Свой компонент | `local/components/<vendor>/<name>/` |
| ID инфоблоков | конфиг проекта (см. [configuration.md](configuration.md)) |
| Точка входа страницы | `<раздел>/index.php` |

## См. также

- [configuration.md](configuration.md)
- [include-areas.md](include-areas.md)
- [getting-started.md](getting-started.md)
- [components/templates.md](components/templates.md)
- [components/header-footer.md](components/header-footer.md)
