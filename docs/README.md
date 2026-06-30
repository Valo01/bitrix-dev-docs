# Документация по разработке на 1C-Bitrix

Внутренние стандарты команды. Описывают **общие подходы** к Bitrix-разработке; конкретная реализация зависит от проекта — смотрите `local/`, `.env.example` и README репозитория.

## С чего начать

1. **[getting-started.md](getting-started.md)** — первые шаги на любом проекте
2. **[structure.md](structure.md)** — куда класть код
3. **[configuration.md](configuration.md)** — ID инфоблоков, настройки, `init.php`
4. **[include-areas.md](include-areas.md)** — включаемые области
5. **[components/README.md](components/README.md)** — компоненты

## Разделы

### Старт и структура

| Документ | Содержание |
|----------|------------|
| [getting-started.md](getting-started.md) | Окружение, первые файлы, чеклисты |
| [structure.md](structure.md) | `local/`, шаблон сайта, приоритеты |
| [configuration.md](configuration.md) | Конфигурация: `.env`, константы, модули настроек |
| [include-areas.md](include-areas.md) | `main.include`, расположение файлов |

### Компоненты

| Документ | Содержание |
|----------|------------|
| [types.md](components/types.md) | Простые, комплексные, служебные |
| [standard-components.md](components/standard-components.md) | Какой `bitrix:*` для какой задачи |
| [include-rules.md](components/include-rules.md) | `IncludeComponent`, кеш, SEF |
| [pages.md](components/pages.md) | Подключение в `index.php` |
| [header-footer.md](components/header-footer.md) | `header.php` / `footer.php` |
| [templates.md](components/templates.md) | Шаблоны, `result_modifier` |
| [custom-components.md](components/custom-components.md) | Свои компоненты и модули |

## Главные правила (для всех проектов)

1. Кастомный код — в `local/` (и при необходимости `/include/`), ядро `bitrix/` не редактируем
2. ID инфоблоков и форм — **не хардкодить** в страницах; выносить в конфиг (`.env`, `constants.php`, хелпер)
3. Контакты и глобальные настройки — из модуля настроек или `COption`, не дублировать в коде
4. Редактируемый HTML — через `bitrix:main.include` или инфоблок
5. Переопределения компонентов — в `local/templates/<шаблон>/components/`
6. Комплексный раздел — **один** вызов `bitrix:news` / `bitrix:catalog` на `index.php`
7. Публичные списки и меню — с кешем (`CACHE_TYPE` => `A`)

## Как читать примеры в документации

Блоки с пометкой **Пример** иллюстрируют реальный паттерн из корпоративных сайтов команды (каталог услуг, разделы врачей/новостей, модуль настроек). Это не обязательная схема для каждого проекта — перед задачей изучите структуру **конкретного** репозитория.
