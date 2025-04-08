# Вычислитель отличий (gendiff)

**gendiff** — CLI-утилита и Python-библиотека для сравнения конфигурационных файлов в форматах JSON и YAML. Поддерживает различные форматы вывода: `stylish`, `plain`, `json`.

---

## 🔧 Установка и запуск

```bash
# Инициализация проекта
uv init

# Установка зависимостей
uv pip install -r requirements.txt

# Запуск во время разработки
uv run gendiff file1.json file2.json
uv run python -m gendiff.scripts.gendiff file1.json file2.json

# Установка пакета и использование
uv pip install .
gendiff file1.json file2.json
```

---

## 🛠 Пример использования

### Справка
```bash
gendiff -h
usage: gendiff [-h] [-f FORMAT] first_file second_file

Compares two configuration files and shows a difference.

optional arguments:
  -h, --help            show this help message and exit
  -f FORMAT, --format FORMAT   set format of output
```

### Stylish (по умолчанию)
```diff
gendiff file1.json file2.json
{
  - follow: false
    host: hexlet.io
  - proxy: 123.234.53.22
  - timeout: 50
  + timeout: 20
  + verbose: true
}
```

### Plain
```text
gendiff --format plain file1.json file2.json
Property 'common.follow' was added with value: false
Property 'common.setting2' was removed
Property 'common.setting3' was updated. From true to null
...
```

### JSON
```bash
gendiff --format json file1.json file2.json
```
```json
[
  {
    "key": "timeout",
    "status": "updated",
    "old_value": 50,
    "new_value": 20
  },
  ...
]
```

---

## 📦 Использование как библиотеки

```python
from gendiff import generate_diff

result = generate_diff("file1.json", "file2.json", format_name="plain")
print(result)
```

---

## 🧪 Тестирование и CI

- Тесты: `pytest`
- Линтер: `make lint`
- GitHub Actions: тесты + линт
- Покрытие: отправка в CodeClimate
- Все тестовые данные: `tests/test_data/`

```bash
make test
make lint
```

---

## 📂 Поддерживаемые форматы

| Формат входных данных | Поддержка |
|------------------------|-----------|
| JSON                   | ✅        |
| YAML (.yml/.yaml)      | ✅        |

| Формат вывода  | Описание                            |
|----------------|-------------------------------------|
| `stylish`      | Иерархичный с отступами и символами |
| `plain`        | Человеческое описание изменений     |
| `json`         | Машиночитаемый JSON-формат          |

---

## 🧠 Архитектура

- `parser.py` — парсинг файлов и определение формата
- `generate_diff()` — построение дерева различий
- `formatters/` — модуль с тремя форматерами: `stylish`, `plain`, `json`
- `scripts/gendiff` — CLI интерфейс

---

## 📌 TODO

- [ ] Поддержка TOML и INI (в будущем)
- [ ] Генерация патчей
- [ ] Веб-интерфейс для визуального сравнения
