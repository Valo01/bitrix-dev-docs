# Конфигурация проекта

Как хранить ID инфоблоков, форм и настройки сайта. Подход зависит от проекта — ниже общие варианты и рекомендации.

## ID инфоблоков и форм

### Правило

Числовые ID **не пишут** напрямую в `index.php` и шаблонах. Они меняются между dev/stage/prod.

### Вариант 1: `.env` + [vlucas/phpdotenv](https://github.com/vlucas/phpdotenv)

[PHP dotenv](https://github.com/vlucas/phpdotenv) загружает переменные из файла `.env` в `$_ENV` и `$_SERVER`. ID инфоблоков, форм и другие значения, которые отличаются между dev/stage/prod, хранят в `.env`, а не в коде.

#### 1. Установка через Composer

Пакет ставится **только через Composer** — в корне сайта.

**Способ A — добавить зависимость одной командой** (если `composer.json` уже есть или создастся автоматически):

```powershell
cd <путь-до-вашего-сайта>
composer require vlucas/phpdotenv
```

**Способ B — прописать вручную** в `composer.json`:

```json
{
    "require": {
        "vlucas/phpdotenv": "^5.6"
    }
}
```

Затем установить зависимости:

```powershell
cd <путь-до-вашего-сайта>
composer install
```

**Что появится после установки:**

| Файл / папка | Назначение |
|--------------|------------|
| `vendor/` | Скачанные пакеты (в т.ч. phpdotenv) |
| `vendor/autoload.php` | Autoload — подключается в `init.php` |
| `composer.lock` | Зафиксированные версии зависимостей |

**Git:** коммитят `composer.json` и `composer.lock`. Папку `vendor/` — только если она принята в вашем проекте; иначе на каждом стенде выполняют `composer install`.

**Требования:** PHP и Composer. Актуальная ветка пакета — v5.x ([релизы на GitHub](https://github.com/vlucas/phpdotenv/releases)). При обновлении мажорной версии смотрите [UPGRADING.md](https://github.com/vlucas/phpdotenv/blob/master/UPGRADING.md).

Дальше: создать `.env` / `.env.example` и подключить загрузку в `init.php` (см. ниже).

---

#### ⚠️ OSPanel: `composer` не работает в PowerShell / VScode

> **Важно при локальной разработке на OSPanel.**  
> Команды `composer install` и `composer require` в PowerShell, CMD и терминале **VScode** часто падают с ошибкой. Это **особенность OSPanel**, а не phpdotenv и не проекта.  
> Ниже — почему так происходит и **два рабочих способа** установки.

Типичная ошибка:

```text
"" не является внутренней или внешней командой...
```

**Почему так:** OSPanel кладёт в PATH обёртку `composer.bat` (например `C:\OSPanel\modules\php\PHP_8.2\composer.bat` — папка зависит от **вашей** версии PHP). Она запускает PHP и `composer.phar` через переменные `%PHP_BIN%` и `%PHP_DIR%`, которые OSPanel выставляет **только во встроенном своём терминале**. В PowerShell, VScode и обычном CMD эти переменные пустые — Windows пытается выполнить пустую команду `""` и падает.

| Команда | Что происходит |
|---------|----------------|
| `composer require vlucas/phpdotenv` | OSPanel `composer.bat` → пустой `PHP_BIN` → ошибка |
| `php.exe` + `composer.phar` напрямую | Обход обёртки OSPanel, работает в любом терминале |
| `composer install` во **встроенном терминале OSPanel** | Переменные заданы → обёртка работает |

**Способ 1: встроенный терминал OSPanel** (рекомендуется — короткие команды `composer`)

1. Запустите **OSPanel** и убедитесь, что нужная версия PHP включена (в трее или в главном окне панели).
2. Откройте **встроенный терминал** OSPanel:
   - правый клик по иконке OSPanel в трее → **Терминал** (или **Консоль**);
   - либо кнопка терминала в главном окне OSPanel (зависит от версии панели).
3. Перейдите в корень сайта:

```bat
cd /d C:\OSPanel\domains\<ваш-сайт>
```

4. Установите пакет — одна из команд:

```bat
composer require vlucas/phpdotenv
```

Если `composer.json` уже есть в репозитории:

```bat
composer install
```

5. Проверьте результат: в папке сайта должны появиться `vendor/` и `vendor/autoload.php`.

В этом терминале OSPanel сам выставляет `PHP_BIN` и `PHP_DIR`, поэтому обёртка `composer.bat` работает без ошибки `"" не является внутренней или внешней командой...`.

---

**Способ 2: полные пути к PHP и Composer** (PowerShell, CMD, терминал VScode)

Когда встроенный терминал OSPanel недоступен, вызывайте `php.exe` и `composer.phar` напрямую. В путях замените `PHP_<версия>` на папку **вашей** версии PHP в OSPanel (например `PHP_8.2`, `PHP_8.3` — смотрите в `C:\OSPanel\modules\php\` или в настройках панели):

```powershell
cd C:\OSPanel\domains\<ваш-сайт>

& "C:\OSPanel\modules\php\PHP_<версия>\php.exe" `
  "C:\OSPanel\userdata\composer\composer.phar" `
  require vlucas/phpdotenv
```

Для уже существующего `composer.json` вместо `require` — `install`:

```powershell
& "C:\OSPanel\modules\php\PHP_<версия>\php.exe" `
  "C:\OSPanel\userdata\composer\composer.phar" `
  install
```

| Часть команды | Назначение |
|---------------|------------|
| `&` | оператор вызова в PowerShell |
| `PHP_<версия>` | папка вашей версии PHP (`PHP_8.2`, `PHP_8.3`, …) |
| `"C:\OSPanel\userdata\composer\composer.phar"` | Composer |
| `require` / `install` | аргументы Composer |

Пути к `php.exe` и `composer.phar` — в `C:\OSPanel\modules\php\PHP_<версия>\` (версия та же, что включена в OSPanel) и `userdata\composer\`.

---

После успешной установки любым способом:

1. создаётся или обновляется `composer.json`;
2. пакеты скачиваются в `vendor/`;
3. пишется `composer.lock`;
4. генерируется `vendor/autoload.php` — его подключают в `init.php`.

---

#### 2. `.env.example` — шаблон для команды

Файл в корне сайта, **в Git**. Реальные ID не указывают — только имена переменных:

```dotenv
IBLOCK_ID_SERVICES=
IBLOCK_ID_FAQ=
IBLOCK_ID_CLINICS=
IBLOCK_ID_REVIEWS=
IBLOCK_ID_PROMO=
IBLOCK_ID_NEWS=
IBLOCK_ID_BLOG=
IBLOCK_ID_LICENSES=
IBLOCK_ID_LEGAL_INFO=
IBLOCK_ID_LEGAL_INFO_DOCUMENTS=

WEB_FORM_ID_APPOINTMENT=
WEB_FORM_ID_ORDER_CALL=

HLBLOCK_ID_REVIEW_SOURCES=
```

В коде проекта используются и другие ключи (`IBLOCK_ID_DOCTORS`, `IBLOCK_ID_SOCLINKS`, `WEB_FORM_ID_REVIEWS` и т.д.) — при настройке нового стенда сверяйтесь с `$_ENV[...]` в репозитории и дополняйте `.env` и `.env.example`.

#### 3. Локальный `.env` — не в Git

После клонирования:

```bash
copy .env.example .env
```

Заполните ID с **вашего** dev-стенда (админка → инфоблоки / формы → столбец ID):

```dotenv
IBLOCK_ID_SERVICES=12
IBLOCK_ID_DOCTORS=7
IBLOCK_ID_NEWS=3
WEB_FORM_ID_APPOINTMENT=1
HLBLOCK_ID_REVIEW_SOURCES=2
```

Добавьте `.env` в `.gitignore`. В некоторых репозиториях корень закрыт правилом `/*`, а `.env.example` явно разрешён через `!/.env.example`.

#### 4. Подключение в `local/php_interface/init.php`

Загрузка — **первые строки** `init.php`, до любого `$_ENV` в коде:

```php
<?php

require_once $_SERVER['DOCUMENT_ROOT'] . '/vendor/autoload.php';
Dotenv\Dotenv::createImmutable($_SERVER['DOCUMENT_ROOT'])->safeLoad();
```

| Что делает вызов | |
|------------------|--|
| `createImmutable(DOCUMENT_ROOT)` | Ищет `.env` в корне сайта; не перезаписывает уже заданные переменные окружения |
| `safeLoad()` | Если файла `.env` нет — сайт не падает (удобно, когда ID заданы на уровне сервера) |

Альтернатива — `load()`: без `.env` будет исключение.

#### 5. Использование в коде

После `safeLoad()` переменные читают через `$_ENV` в параметрах компонентов и шаблонах:

```php
// index.php раздела
$APPLICATION->IncludeComponent('bitrix:news', '', [
    'IBLOCK_ID' => $_ENV['IBLOCK_ID_NEWS'],
    'SEF_MODE' => 'Y',
    'SEF_FOLDER' => '/news/',
]);

// footer.php — веб-форма
$APPLICATION->IncludeComponent('bitrix:form.result.new', '', [
    'WEB_FORM_ID' => $_ENV['WEB_FORM_ID_FEEDBACK'],
]);

// result_modifier.php — highload-блок
$hlblock = \Bitrix\Highloadblock\HighloadBlockTable::getById(
    $_ENV['HLBLOCK_ID_SOURCES']
)->fetch();
```

То же значение доступно в `$_SERVER['IBLOCK_ID_NEWS']`.

#### 6. Добавление новой переменной

1. Добавить строку в `.env.example` (без реального ID).
2. Заполнить значение в локальном `.env` на каждом стенде.
3. Использовать в коде: `$_ENV['IBLOCK_ID_NEW']`.

#### 7. Если `$_ENV` пустой

Проверьте `variables_order` в `php.ini` — должны быть `E` и `S`. См. [php.net — variables_order](https://www.php.net/manual/en/ini.core.php#ini.variables-order).

Дополнительно: валидация `required()`, `notEmpty()`, `isInteger()` — в [документации phpdotenv](https://github.com/vlucas/phpdotenv#usage). На практике часто достаточно `safeLoad()` без проверок.

#### 8. Правила

- `.env` — только на стенде, не в Git
- `.env.example` — в Git, обновлять при каждой новой переменной
- ID инфоблоков и форм — не хардкодить в `index.php`
- Секреты (API, пароли) — только в `.env`, не в `.env.example`

### Вариант 2: константы в PHP

```php
// local/php_interface/constants.php
define('IBLOCK_NEWS_ID', 5);
define('IBLOCK_CATALOG_ID', 12);
```

```php
// index.php
'IBLOCK_ID' => IBLOCK_NEWS_ID,
```

Подходит для небольших проектов без Composer. Значения всё равно не хардкодят в страницах.

### Вариант 3: хелпер-класс

```php
// local/php_interface/classes/IBlock.php
class IBlock {
    public static function get(string $code): int {
        // маппинг код → ID из .env, БД или массива
    }
}
```

```php
'IBLOCK_ID' => \Project\IBlock::get('news'),
```

Удобно, когда в коде используются символьные коды инфоблоков.

---

## Настройки сайта (телефон, логотип, контакты)

Данные, которые правит контент-менеджер без деплоя:

```php
$phone = \COption::GetOptionString('module.id', 'UF_PHONE');
$logo = \CFile::GetPath(\COption::GetOptionString('module.id', 'UF_LOGO'));
```

Варианты хранения:

| Способ | Когда |
|--------|-------|
| Модуль настроек (`askaron.settings`, свой модуль) | Часто на корпоративных сайтах |
| `COption` модуля `main` | Простые проекты |
| Отдельный инфоблок «Настройки» | Редко, один элемент |

**Правило:** телефон и логотип в шапке — из настроек или `main.include`, не строкой в `header.php`.

> **Пример:** модуль `askaron.settings`, поля `UF_PHONE`, `UF_MAIL`, `UF_LOGO_FOOTER` — на одном из сайтов команды.

---

## `init.php` — точка входа

Типичное содержимое (не всё обязательно):

- подключение Composer / Dotenv
- автозагрузка классов из `local/php_interface/classes/`
- регистрация обработчиков событий
- вспомогательные функции

### Хелпер для хлебных крошек (опционально)

```php
function ShowNavChain($template = '')
{
    global $APPLICATION;
    $APPLICATION->IncludeComponent('bitrix:breadcrumb', $template, [
        'START_FROM' => '0',
        'PATH' => '',
        'SITE_ID' => SITE_ID,
    ]);
}
```

Можно вызывать `bitrix:breadcrumb` напрямую — хелпер лишь сокращает дублирование.

### ComponentHelper (опционально)

Класс для отложенного вывода внутри кешируемых компонентов (крошки, блоки вне кеша). Путь и API — смотрите в `local/php_interface/classes/` конкретного проекта.

---

## `local/php_interface/include/`

Часто встречающиеся файлы:

| Файл | Назначение |
|------|------------|
| `urlrewrites.php` | Дополнительные правила ЧПУ |
| `site_closed.php` | Заглушка при закрытом сайте |
| `*_filter.inc.php` | Общие фильтры для компонентов |

---

## Чеклист при добавлении инфоблока

- [ ] ID вынесен в конфиг (`.env`, константа или хелпер)
- [ ] `.env.example` обновлён (если используется Dotenv)
- [ ] В вызовах компонента заданы `CACHE_TYPE` и `CACHE_TIME`
- [ ] При SEF — запись в `urlrewrite.php`

## См. также

- [getting-started.md](getting-started.md)
- [structure.md](structure.md)
- [include-areas.md](include-areas.md)
