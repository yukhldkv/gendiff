# Проект: Вычислитель отличий (gendiff)  

**Полный план с примерами кода**

---

## Шаг 1: Вывод справки

**Задачи:**

- Инициализировать проект:
  ```bash
  uv init
  ```

- Создать CLI-скрипт `gendiff`
- Подключить `argparse` и реализовать базовую справку

**Пример справки:**
```bash
gendiff -h
usage: gendiff [-h] first_file second_file

Compares two configuration files and shows a difference.

positional arguments:
  first_file
  second_file

optional arguments:
  -h, --help  show this help message and exit
```

**Запуск во время разработки:**
```bash
uv run gendiff file1.json file2.json
uv run python -m gendiff.scripts.gendiff file1.json file2.json
```

---

## Шаг 2: Аргументы и опции

**Цель:** добавить флаг `--format`, указывающий формат вывода.

**Пример справки:**
```bash
gendiff -h
usage: gendiff [-h] [-f FORMAT] first_file second_file

optional arguments:
  -f FORMAT, --format FORMAT   set format of output
```

**Конфигурация `pyproject.toml`:**
```toml
[tool.hatch.build.targets.wheel]
packages = ["gendiff"]
```

---

## Шаг 3: Чтение и парсинг файлов

**Пример файла `file1.json`:**
```json
{
  "host": "hexlet.io",
  "timeout": 50,
  "proxy": "123.234.53.22",
  "follow": false
}
```

**Пример кода:**
```python
import json

with open('file1.json') as f:
    data = json.load(f)
```

---

## Шаг 4: Сравнение плоских JSON-файлов

**Пример вывода:**
```diff
{
  - follow: false
    host: hexlet.io
  - proxy: 123.234.53.22
  - timeout: 50
  + timeout: 20
  + verbose: true
}
```

**Интерфейс:**
```python
from gendiff import generate_diff

diff = generate_diff(file_path1, file_path2)
print(diff)
```

---

## Шаг 5: Автотесты и CI

**Что подключить:**

- `pytest`
- `make lint` с `flake8`
- GitHub Actions для линтера и тестов
- CodeClimate + бейджи

**Структура тестов:**
```
tests/
  ├── test_data/
  │   ├── file1.json
  │   ├── file2.json
  │   └── expected.txt
  └── test_gendiff.py
```

**Пример теста:**
```python
def test_flat_json():
    expected = read_file('expected.txt')
    result = generate_diff('file1.json', 'file2.json')
    assert result == expected
```

---

## Шаг 6: Поддержка YAML

**Пример файла `.yaml`:**
```yaml
host: hexlet.io
timeout: 50
proxy: 123.234.53.22
follow: false
```

**Формат вывода** — тот же, что и для JSON.

**Рекомендованная библиотека:**
```python
import yaml
```

**Определение формата по расширению:**
```python
if file_path.endswith(('.yml', '.yaml')):
    data = yaml.safe_load(open(file_path))
```

---

## Шаг 7: Рекурсивное сравнение

**Пример вложенного вывода (stylish):**
```diff
{
    common: {
      + follow: false
        setting1: Value 1
      - setting2: 200
      - setting3: true
      + setting3: null
      + setting4: blah blah
      + setting5: {
            key5: value5
        }
        setting6: {
            doge: {
              - wow:
              + wow: so much
            }
            key: value
          + ops: vops
        }
    }
    group1: {
      - baz: bas
      + baz: bars
        foo: bar
      - nest: {
            key: value
        }
      + nest: str
    }
  - group2: {
        abc: 12345
        deep: {
            id: 45
        }
    }
  + group3: {
        deep: {
            id: {
                number: 45
            }
        }
        fee: 100500
    }
}
```

**Ключевая идея:**

- Разделить построение `diff`-дерева и форматирование
- Использовать `children` в узлах дерева, если оба значения — словари

---

## Шаг 8: Формат plain

**Пример вывода:**
```
Property 'common.follow' was added with value: false
Property 'common.setting2' was removed
Property 'common.setting3' was updated. From true to null
Property 'common.setting4' was added with value: 'blah blah'
Property 'common.setting5' was added with value: [complex value]
Property 'common.setting6.doge.wow' was updated. From '' to 'so much'
Property 'common.setting6.ops' was added with value: 'vops'
Property 'group1.baz' was updated. From 'bas' to 'bars'
Property 'group1.nest' was updated. From [complex value] to 'str'
Property 'group2' was removed
Property 'group3' was added with value: [complex value]
```

**Особенности:**

- Составные значения → `[complex value]`
- Значения в кавычках
- Путь — полное имя свойства (`a.b.c`)

---

## Шаг 9: Формат json

**Пример использования:**
```bash
gendiff --format json file1.json file2.json
```

**Пример вызова:**
```python
generate_diff(file1, file2, format_name='json')
```

**Вывод (сериализованный diff в формате JSON):**
```json
[
  {
    "key": "host",
    "status": "unchanged",
    "value": "hexlet.io"
  },
  {
    "key": "timeout",
    "status": "updated",
    "old_value": 50,
    "new_value": 20
  }
  ...
]
```

