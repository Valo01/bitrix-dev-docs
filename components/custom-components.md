# Создание своих компонентов

Когда и как добавлять компоненты в `local/components/`.

## Когда создавать свой компонент

| Ситуация | Решение |
|----------|---------|
| Другая вёрстка `news.list` | Шаблон в `local/templates/<шаблон>/components/bitrix/news.list/` |
| Статичный HTML | `bitrix:main.include` + файл в `include/` |
| UI с параметрами, переиспользуется | `local/components/<vendor>/<name>/` |
| Данные из инфоблока | `bitrix:news.list` или `bitrix:news` |
| Сложная логика + API | D7-компонент или сервис + тонкий компонент |

**Правило:** сначала шаблон стандартного компонента, потом свой — только если стандартного недостаточно.

## Структура компонента

```
local/components/<vendor>/<name>/
├── .description.php
├── .parameters.php
├── class.php              # D7 (рекомендуется)
└── templates/
    └── .default/
        ├── template.php
        └── result_modifier.php
```

Подключение: `<vendor>:<name>`.

```php
$APPLICATION->IncludeComponent('company:section.title', '', [
    'TITLE' => 'Заголовок',
    'SUBTITLE' => 'Подзаголовок',
]);
```

## D7-компонент

```php
<?php
// class.php
use Bitrix\Main\Loader;

class CompanySectionTitleComponent extends CBitrixComponent
{
    public function executeComponent(): void
    {
        if (!Loader::includeModule('iblock')) {
            return;
        }

        $this->includeComponentTemplate();
    }
}
```

## Кеш

```php
public function executeComponent(): void
{
    if ($this->startResultCache(false, $this->getCacheKey())) {
        $this->arResult['ITEMS'] = $this->getItems();
        $this->endResultCache();
    }

    $this->includeComponentTemplate();
}
```

## Компоненты из модулей

Модули в `local/modules/<module>/` могут устанавливать компоненты в `install/bitrix/components/`. Namespace — по имени модуля (`rapid:constructor.detail` и т.д.).

Перед использованием проверьте, установлен ли модуль: `Loader::includeModule('module.id')`.

## AJAX-endpoint'ы

Для действий вне рендера страницы — `local/ajax/`:

```php
<?php
require $_SERVER['DOCUMENT_ROOT'] . '/bitrix/modules/main/include/prolog_before.php';

if (!check_bitrix_sessid()) {
    die(json_encode(['error' => 'sessid']));
}

// логика

echo json_encode(['success' => true]);
```

Типичные задачи: отправка формы, подгрузка списка, фильтрация.

## ComponentHelper (опционально)

В некоторых проектах есть класс для отложенного вывода внутри кешируемого компонента (крошки, блоки вне кеша). Ищите в `local/php_interface/classes/` — API может отличаться.

---


## См. также

- [standard-components.md](standard-components.md)
- [types.md](types.md)
- [templates.md](templates.md)
- [../structure.md](../structure.md)
