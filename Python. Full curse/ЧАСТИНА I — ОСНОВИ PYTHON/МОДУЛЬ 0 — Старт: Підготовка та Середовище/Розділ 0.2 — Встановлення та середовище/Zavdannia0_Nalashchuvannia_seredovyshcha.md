# 📋 Завдання 0 — Налаштування середовища

**Охоплює:** §§ 0.2.1 – 0.2.4
**Складність:** початковий рівень
**Орієнтовний час:** 1,5 – 3 години залежно від операційної системи і наявного досвіду

---

## Навіщо це завдання

Це не звичайна вправа де ти щось написав, перевірив і пішов далі. Це налаштування твого робочого місця — середовища, в якому буде написано весь подальший код курсу. Якщо зробити його уважно і правильно зараз, воно буде непомітно і надійно служити місяцями. Якщо поспішити і пропустити якийсь крок — рано чи пізно це дасться взнаки в найнезручніший момент.

Виконуй кожен пункт послідовно. Не переходь до наступного, поки поточний не завершений і не перевірений.

---

## Частина 1: Python через pyenv

### 1.1 Встанови pyenv

**macOS / Linux:**

```bash
curl https://pyenv.run | bash
```

Після виконання додай рядки до конфігурації своєї оболонки (`~/.zshrc` для Zsh або `~/.bashrc` для Bash):

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Перезапусти термінал або виконай `source ~/.zshrc` (або відповідний файл). Перевір:

```bash
pyenv --version
```

**Windows:** використай офіційний інсталятор Python з `python.org` і постав галочку **«Add Python to PATH»** під час встановлення. Або встанови WSL і дотримуйся інструкції для Linux.

---

### 1.2 Встанови Python і зроби його глобальним

Спочатку встанови необхідні системні залежності (Ubuntu/Debian):

```bash
sudo apt update && sudo apt install -y make build-essential libssl-dev \
  zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
  libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
  libffi-dev liblzma-dev
```

На macOS залежності встановлюються через Xcode Command Line Tools:

```bash
xcode-select --install
```

Встанови Python:

```bash
pyenv install 3.12.3
pyenv global 3.12.3
```

**Перевірка — очікуваний результат:**

```bash
python --version
# Python 3.12.3

which python
# /home/ім'я/.pyenv/shims/python  ← має містити .pyenv, не /usr або /bin
```

Якщо `which python` показує системний шлях без `.pyenv` — перевір чи правильно додані рядки до конфігурації оболонки і чи перезапущений термінал.

---

## Частина 2: Git

### 2.1 Встанови і налаштуй Git

**macOS:**
```bash
brew install git   # або xcode-select --install
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt install git
```

**Windows:** завантаж з `git-scm.com`. На кроці вибору редактора обери VS Code.

Налаштуй своє ім'я та email — вони будуть у кожному коміті:

```bash
git config --global user.name "Твоє Ім'я"
git config --global user.email "твій@email.com"
git config --global init.defaultBranch main
```

Останній рядок встановлює `main` як назву гілки за замовчуванням — це сучасний стандарт.

**Перевірка:**

```bash
git --version
git config --global --list
# Має показати user.name, user.email і init.defaultBranch
```

---

### 2.2 Налаштуй глобальний .gitignore

Один раз — для всіх проєктів на цій машині:

```bash
git config --global core.excludesfile ~/.gitignore_global
```

Створи файл `~/.gitignore_global`:

```
# macOS
.DS_Store
.DS_Store?
.Spotlight-V100
.Trashes

# Windows
Thumbs.db
Desktop.ini
ehthumbs.db

# JetBrains IDE
.idea/

# Vim / Neovim
*.swp
*.swo
*~
```

Тепер ці файли ніколи не з'являтимуться у `git status` жодного з твоїх проєктів.

---

## Частина 3: VS Code

### 3.1 Встанови VS Code

Завантаж з `code.visualstudio.com` для своєї системи.

**macOS:** після встановлення відкрий VS Code, натисни `Cmd+Shift+P`, введи `shell command` і обери **«Shell Command: Install 'code' command in PATH»**.

**Windows:** під час інсталятора постав галочки **«Add to PATH»** і **«Add 'Open with Code' action»**.

**Перевірка:**

```bash
code --version
```

---

### 3.2 Встанови розширення

Відкрий панель розширень (`Ctrl+Shift+X`) і встанови кожне:

| Розширення | ID в магазині |
|------------|--------------|
| Python | `ms-python.python` |
| Pylance | `ms-python.vscode-pylance` |
| Black Formatter | `ms-python.black-formatter` |
| Ruff | `charliermarsh.ruff` |
| GitLens | `eamodio.gitlens` |
| Thunder Client | `rangav.vscode-thunder-client` |
| SQLite Viewer | `qwtel.sqlite-viewer` |
| Docker | `ms-azuretools.vscode-docker` |

Або встанови всі одразу через термінал:

```bash
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-python.black-formatter
code --install-extension charliermarsh.ruff
code --install-extension eamodio.gitlens
code --install-extension rangav.vscode-thunder-client
code --install-extension qwtel.sqlite-viewer
code --install-extension ms-azuretools.vscode-docker
```

---

### 3.3 Налаштуй settings.json

Відкрий командну палітру (`Ctrl+Shift+P` або `Cmd+Shift+P`), введи `Open User Settings JSON` і заміни вміст файлу на:

```json
{
  "editor.fontSize": 14,
  "editor.tabSize": 4,
  "editor.insertSpaces": true,
  "editor.rulers": [88],
  "editor.minimap.enabled": false,
  "editor.renderWhitespace": "boundary",
  "editor.wordWrap": "off",
  "files.autoSave": "onFocusChange",
  "python.analysis.typeCheckingMode": "basic",
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.ruff": "explicit",
      "source.organizeImports.ruff": "explicit"
    }
  }
}
```

---

## Частина 4: Робочий проєкт

### 4.1 Створи директорію і ініціалізуй репозиторій

```bash
mkdir python-course
cd python-course
git init
```

Перевір що репозиторій ініціалізований:

```bash
git status
# On branch main
# No commits yet
# nothing to commit
```

---

### 4.2 Зафіксуй версію Python для проєкту

```bash
pyenv local 3.12.3
```

Це створить файл `.python-version` у директорії. Тепер кожен хто зайде в цю папку матиме ту саму версію Python.

**Перевірка:**

```bash
cat .python-version
# 3.12.3
```

---

### 4.3 Створи і активуй віртуальне середовище

```bash
python -m venv venv
```

Активація:

```bash
source venv/bin/activate      # macOS / Linux
venv\Scripts\activate         # Windows (Command Prompt)
venv\Scripts\Activate.ps1     # Windows (PowerShell)
```

Після активації рядок запрошення зміниться:

```
(venv) user@machine:~/python-course$
```

**Перевірка — обидві команди мають показувати шлях всередині `venv`:**

```bash
which python
# /home/ім'я/python-course/venv/bin/python

which pip
# /home/ім'я/python-course/venv/bin/pip
```

---

### 4.4 Оновити pip і встанови базові інструменти

```bash
pip install --upgrade pip
pip install pre-commit
```

---

### 4.5 Створи .gitignore

У корені `python-course/` створи файл `.gitignore` з таким вмістом:

```
# Віртуальні середовища
venv/
.venv/
env/
ENV/

# Байт-код і кеш Python
__pycache__/
*.py[cod]
*$py.class

# Дистрибуція та збірка
dist/
build/
*.egg-info/
*.egg

# Тестування і покриття
.coverage
.coverage.*
htmlcov/
.pytest_cache/
.tox/

# Змінні середовища і секрети
.env
.env.*
!.env.example

# Редактори і IDE
.vscode/
.idea/

# Операційні системи
.DS_Store
Thumbs.db

# Логи і тимчасові файли
*.log
*.tmp
```

**Перевірка:**

```bash
git status
# .python-version і .gitignore мають з'явитись як нові файли
# директорія venv/ — НЕ має з'являтись
```

Якщо `venv/` все одно з'являється у `git status` — перевір що в `.gitignore` є рядок `venv/` і що файл збережений.

---

### 4.6 Налаштуй pre-commit

Створи файл `.pre-commit-config.yaml` у корені проєкту:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
      - id: black

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff
        args: [--fix]
```

Активуй хуки:

```bash
pre-commit install
```

Оновити версії хуків до актуальних (рекомендовано):

```bash
pre-commit autoupdate
```

Запусти хуки вручну на всіх файлах — перша перевірка може зайняти кілька хвилин, бо pre-commit завантажує залежності:

```bash
pre-commit run --all-files
```

Якщо все пройшло — побачиш рядки зі статусом `Passed` або `Skipped` навпроти кожного хука. Жодного `Failed`.

---

## Частина 5: Перша програма і перший коміт

### 5.1 Відкрий проєкт у VS Code

```bash
code .
```

VS Code відкриється з директорією `python-course/`. Він, мабуть, одразу запропонує обрати інтерпретатор Python — обери той що у `venv`.

Якщо не запропонував: `Ctrl+Shift+P` → `Python: Select Interpreter` → обери `./venv/bin/python`.

---

### 5.2 Напиши перший файл

Створи `hello.py` у корені проєкту:

```python
from datetime import date

name = "Твоє Ім'я"
year = date.today().year

print(f"Привіт! Мене звуть {name}.")
print(f"Зараз {year} рік, і я починаю вивчати Python.")
```

Запусти з терміналу VS Code (переконайся що середовище активоване):

```bash
python hello.py
```

**Очікуваний результат:**

```
Привіт! Мене звуть Твоє Ім'я.
Зараз 2025 рік, і я починаю вивчати Python.
```

---

### 5.3 Зроби перший коміт

```bash
git add .
git commit -m "feat: init project"
```

pre-commit запуститься автоматично. Можливий сценарій: Black переформатував `hello.py`, коміт зупинився. Це нормально — це саме та поведінка, яку ми й хотіли. Виконай:

```bash
git add .
git commit -m "feat: init project"
```

Тепер коміт пройде — файл вже відформатований.

**Перевірка:**

```bash
git log --oneline
# abc1234 feat: init project
```

Ти бачиш перший коміт в історії. Це важливий момент — можна зупинитись на секунду і оцінити.

---

### 5.4 Перевір структуру проєкту

Після всіх кроків структура директорії має виглядати так:

```
python-course/
├── .git/                     ← прихована директорія, Git тут живе
├── .gitignore                ← що не йде в репозиторій
├── .pre-commit-config.yaml   ← конфігурація хуків
├── .python-version           ← версія Python для pyenv
├── hello.py                  ← перша програма
├── requirements.txt          ← поки порожній або відсутній
└── venv/                     ← віртуальне середовище (не в git)
```

Якщо щось відрізняється — з'ясуй чому перш ніж рухатись далі.

---

## Частина 6: GitHub (необов'язково, але рекомендовано)

### 6.1 Зареєструйся і створи репозиторій

Якщо ще немає акаунту — зареєструйся на `github.com`. Це безкоштовно.

Створи новий репозиторій: натисни «New», дай назву `python-course`, постав **Private** (особистий навчальний репозиторій не має сенсу робити публічним), **не** ставляй галочки на ініціалізацію з README чи `.gitignore` — він вже є локально.

---

### 6.2 Підключи локальний репозиторій і відправ

```bash
git remote add origin https://github.com/твій-нікнейм/python-course.git
git branch -M main
git push -u origin main
```

Перевір на `github.com/твій-нікнейм/python-course` — мають з'явитись твої файли.

Тепер після кожного коміту достатньо:

```bash
git push
```

---

## Частина 7: Бонусні завдання

Ці пункти не обов'язкові для продовження курсу, але допоможуть глибше розібратись з інструментами.

---

### ⭐ Бонус 1: uv як альтернатива pip

Встанови `uv`:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# або через pip
pip install uv
```

Спробуй відтворити все що робив через `pip`, але тепер через `uv`. Виконай по черзі в тій самій директорії:

```bash
# Деактивуй старе середовище
deactivate

# Видали старе venv
rm -rf venv

# Створи нове через uv
uv venv

# Активуй
source .venv/bin/activate

# Встанови pre-commit через uv
uv pip install pre-commit

# Переглянь список встановленого
uv pip list
```

Порівняй швидкість встановлення між `pip install` і `uv pip install` — встанови будь-яку бібліотеку обома способами і відчуй різницю.

---

### ⭐ Бонус 2: зіпсуй і відновись

Це навчальний експеримент. Мета — зрозуміти як Git рятує від помилок.

Відкрий `hello.py` і внеси якісь зміни — наприклад, видали кілька рядків або зіпсуй синтаксис. Збережи файл, але не комітуй. Тепер перевір що змінилось:

```bash
git diff hello.py
```

Ти бачиш що саме змінилось: рядки з мінусом зліва — видалені, з плюсом — додані.

Тепер відкоти зміни і повернись до стану останнього коміту:

```bash
git restore hello.py
```

Відкрий файл — він знову такий як був. Запусти програму — вона знову працює. Ось вона — суперсила Git.

---

### ⭐ Бонус 3: другий коміт і перегляд історії

Доповни `hello.py` — додай ще один рядок виводу, наприклад:

```python
print("Це буде чудова подорож.")
```

Зроби другий коміт:

```bash
git add hello.py
git commit -m "feat: додати мотиваційний рядок"
```

Тепер переглянь повну історію:

```bash
git log
```

І скорочену:

```bash
git log --oneline
```

Ти бачиш два коміти: перший знизу, другий зверху. Спробуй переглянути що змінилось у другому коміті — візьми його хеш з `git log --oneline` і виконай:

```bash
git show abc1234
# де abc1234 — хеш другого коміту
```

---

### ⭐ Бонус 4: перевір .gitignore у дії

Переконайся що `.gitignore` дійсно ігнорує те, що має ігнорувати.

Створи тимчасовий файл у директорії `venv/`, наприклад `venv/test.txt`. Тепер виконай:

```bash
git status
```

Файл `venv/test.txt` не повинен з'явитись у переліку невідстежуваних файлів — Git його не бачить завдяки `.gitignore`.

Перевір чому конкретний файл ігнорується:

```bash
git check-ignore -v venv/test.txt
# .gitignore:2:venv/    venv/test.txt
```

Git покаже в якому файлі `.gitignore` і на якому рядку знайшов відповідний шаблон.

Видали тимчасовий файл:

```bash
rm venv/test.txt
```

---

## Чеклист: перш ніж рухатись далі

Пройдись по кожному пункту і переконайся що він виконаний:

```
□ pyenv встановлений і команда `pyenv --version` працює
□ Python 3.12+ встановлений через pyenv
□ `python --version` показує 3.12.x і шлях містить .pyenv
□ Git встановлений, налаштовані user.name і user.email
□ Глобальний .gitignore налаштований
□ VS Code встановлений, команда `code .` відкриває директорію
□ Всі вісім розширень встановлені
□ settings.json налаштований
□ Директорія python-course/ створена і ініціалізована через git
□ Версія Python зафіксована через `pyenv local`
□ Віртуальне середовище venv/ створено і активоване
□ .gitignore створений і venv/ не з'являється в git status
□ .pre-commit-config.yaml створений, хуки встановлені
□ hello.py написаний і виводить правильний результат
□ Перший коміт зроблений і видно в git log
□ VS Code використовує інтерпретатор з venv/
```

Якщо всі пункти відмічені — середовище налаштоване повністю і правильно. Можна рухатись до першого модуля.

---

*← § 0.2.4 — Git та pre-commit хуки*

*§ 1.1.1 — Змінні та типи даних →*
