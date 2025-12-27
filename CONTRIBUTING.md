# Руководство по разработке

## Требования

- **ОС**: Ubuntu 20.04 LTS
- **Shell**: Bash 4.0+
- **Инструменты**: `shellcheck`, `git`

```bash
# Установка зависимостей
sudo apt-get update
sudo apt-get install -y shellcheck
```

## Структура проекта

```
DO4/
├── .github/
│   └── workflows/          # GitHub Actions конфиги
├── src/
│   ├── 01/                 # Part 1: Генератор файлов
│   ├── 02/                 # Part 2: Засорение ФС
│   ├── ... (03-09)
│   └── tests/              # Тестовые скрипты
├── docs/                   # Документация
└── README.md
```

## Стиль кода

### Bash скрипты

1. **Shebang и основные параметры:**
   ```bash
   #!/bin/bash
   
   set -euo pipefail  # Строгий режим
   ```

2. **Переменные:**
   ```bash
   # Глобальные константы - UPPERCASE
   readonly CONSTANT_VALUE=1024
   
   # Локальные переменные - lowercase
   local temp_var="value"
   
   # Функции - snake_case
   my_function_name()
   ```

3. **Функции:**
   ```bash
   # Документация функции
   # $1 - первый параметр
   # $2 - второй параметр
   # Returns: 0 if success, 1 if error
   my_function() {
       local param1=$1
       local param2=$2
       
       # Тело функции
   }
   ```

4. **Обработка ошибок:**
   ```bash
   # Используйте функции для ошибок
   if [[ ! -d "$path" ]]; then
       echo "❌ Ошибка: директория не найдена" >&2
       return 1
   fi
   ```

5. **Комментарии:**
   ```bash
   # Однострочный комментарий
   
   # Многострочный комментарий
   # описывает сложную логику
   ```

## Процесс разработки

### 1. Создание feature branch

```bash
git checkout -b feature/description-of-change
```

### 2. Разработка и тестирование

```bash
# Проверить синтаксис
shellcheck src/XX/main.sh

# Запустить скрипт с тестовыми параметрами
cd src/XX
bash main.sh [параметры]
```

### 3. Фиксирование проблем ShellCheck

```bash
# Типичные проблемы и решения:
# SC2086: Double quote to prevent globbing and word splitting
# ❌ echo $var
# ✅ echo "$var"

# SC2181: Check exit code directly
# ❌ command; if [ $? -eq 0 ]; then
# ✅ if command; then

# SC2046: Quote expansions to avoid word splitting
# ❌ local files=$(find . -name "*.sh")
# ✅ local files=( $(find . -name "*.sh") )
```

### 4. Коммиты

```bash
# Используйте информативные сообщения
git commit -m "feat(part1): add improved error handling"
git commit -m "fix(part2): resolve slow find command"
git commit -m "docs: update README with examples"
```

**Префиксы:**
- `feat:` - новая функция
- `fix:` - исправление ошибки
- `docs:` - документация
- `refactor:` - переработка кода
- `test:` - добавление тестов
- `perf:` - оптимизация производительности

### 5. Pull Request

```bash
git push origin feature/description-of-change
```

Отправить PR с описанием:
- Что было изменено
- Почему это улучшение
- Как тестировать

## Тестирование

### Локальное тестирование

```bash
# Part 1
cd src/01
mkdir -p /tmp/test_part1
bash main.sh /tmp/test_part1 2 ab 3 file.txt 1kb
cat filesystem_*.log
rm -rf /tmp/test_part1 filesystem_*.log

# Part 2 (с таймаутом)
cd src/02
timeout 30 bash main.sh xy xy.log 2Mb || true
cat /tmp/filesystem_log.log 2>/dev/null || echo "Логи не созданы"
rm -rf /tmp/filesystem_* 2>/dev/null || true
```

### CI/CD проверка

Когда вы отправляете PR, автоматически запускаются:
1. **ShellCheck** - проверка синтаксиса
2. **Тесты** - базовые функциональные тесты

## Документация

Каждая часть проекта должна иметь `README.md` с:

```markdown
# Part X - [Название]

## Описание
Что делает эта часть проекта

## Требования
- Bash 4.0+
- [Другие требования]

## Использование
\`\`\`bash
./main.sh [параметры]
\`\`\`

## Параметры
- **Параметр 1**: описание
- **Параметр 2**: описание

## Примеры
\`\`\`bash
bash main.sh param1 param2
\`\`\`

## Особенности
- Одна особенность
- Вторая особенность

## Логирование
Описание логирования и лог-файлов
```

## Отладка

### Включить debug режим

```bash
# В начало скрипта добавить
set -x  # Выводить команды перед выполнением

# Или запустить с
bash -x main.sh [параметры]
```

### Общие проблемы

**Проблема**: "source: не найден"
```bash
# ❌ Не правильно
source config.sh

# ✅ Правильно - указать путь
source "$(dirname "$0")/config.sh"
```

**Проблема**: "Permission denied"
```bash
# Сделать скрипт исполняемым
chmod +x src/XX/main.sh
```

**Проблема**: Медленное выполнение
```bash
# Профилировать скрипт
time bash main.sh [параметры]

# Найти медленные команды
grep -n "find /" main.sh  # find / очень медленный
```

## Вопросы и обсуждение

Если у вас есть вопросы:
1. Проверьте существующие Issues
2. Создайте новый Issue с описанием
3. Обсудите в Pull Request comments

## Лицензия

См. LICENSE файл
