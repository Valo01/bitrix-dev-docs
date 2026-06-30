# Включаемые области

Где хранить файлы для `bitrix:main.include` и как их подключать.

## Назначение

`bitrix:main.include` — для фрагментов HTML/PHP, которые:

- редко меняются или правятся контент-менеджером;
- не требуют выборки из инфоблока;
- не нуждаются в кеше и параметрах компонента.

Для списков, каталогов, форм с логикой — используйте `news.list`, `catalog`, `form.result.new` или свой компонент.

## Где лежат файлы

Два распространённых соглашения в проектах команды:

### Вариант A: корень сайта

```
/include/
├── header/          # logo.php, phone.php
├── footer/          # about_company.php, warning_text.php
├── about/           # блоки страницы «О компании»
└── <раздел>/        # по одной папке на раздел сайта
```

### Вариант B: внутри шаблона

```
local/templates/<шаблон>/include/
├── header/
├── footer/
└── <раздел>/
```

**Перед правкой** проверьте, какой вариант принят в **вашем** проекте. Путь в `PATH` должен соответствовать реальному расположению файла.

## Подключение

```php
$APPLICATION->IncludeComponent('bitrix:main.include', '', [
    'AREA_FILE_SHOW' => 'file',
    'PATH' => '/include/about/banner.php',
]);
```

Путь — **от корня сайта** (начинается с `/`).

Для файлов в шаблоне:

```php
'PATH' => '/local/templates/<шаблон>/include/header/logo.php',
```

## Содержимое файла области

Обычный PHP/HTML **без** `bitrix/header.php` и `bitrix/footer.php`:

```php
<p>Текст блока</p>
```

## Группировка по разделам

Рекомендуемая структура папок — **по разделу сайта**, а не по типу контента:

```
include/
├── about/       # всё для /about/
├── contacts/    # всё для /contacts/
├── footer/      # глобальные блоки подвала
└── main/        # блоки главной
```

## Условный вывод (SEO и доп. блоки)

Если блок нужен только на определённом «экране» комплексного компонента, шаблон компонента устанавливает флаг, `index.php` проверяет его **после** вызова компонента:

```php
// В шаблоне комплексного компонента (news.php, section.php):
$GLOBALS['SECTION_SHOW_SEO'] = true;

// В index.php раздела:
if (!empty($GLOBALS['SECTION_SHOW_SEO'])) {
    $APPLICATION->IncludeComponent('bitrix:main.include', '', [
        'PATH' => '/include/<раздел>/seo-title.inc.php',
    ]);
}
```

Альтернатива — `AddViewContent` / `ShowViewContent` для вставки блока внутрь шаблона каталога.

## Что НЕ класть в include

| Данные | Где хранить |
|--------|-------------|
| Телефон, email, логотип | Модуль настроек / `COption` |
| Списки элементов | Инфоблок + `news.list` / `catalog` |
| Формы с обработкой | `form.result.new` или `local/ajax/` |

## См. также

- [configuration.md](configuration.md)
- [components/header-footer.md](components/header-footer.md)
- [components/standard-components.md](components/standard-components.md)
