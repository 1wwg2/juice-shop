# Звіт з циклу лабораторних робіт: Впровадження елементів DevSecOps у процес розробки ПЗ за допомогою GitHub Actions

**Мета роботи:** Розробити та інтегрувати автоматизовані процеси перевірки безпеки (DevSecOps) у CI/CD конвеєр публічного репозиторію на базі GitHub Actions. Інтеграція охоплює три ключові етапи:
1. Статичний аналіз коду (SAST).
2. Аналіз безпеки залежностей (Software Composition Analysis - SCA).
3. Динамічне тестування безпеки (DAST).

**Тестовий репозиторій:** [juice-shop-github-actions](https://github.com/dimdimuzun/juice-shop-github-actions)

---

## Підготовчий етап: Розгортання OWASP Juice Shop
Для тестування інструментів безпеки було обрано навмисно вразливий вебзастосунок OWASP Juice Shop. 
**Виконані дії:**
1. Клонування та налаштування локального середовища Juice Shop.
2. Підключення проєкту до репозиторію GitHub через SSH.
3. Локальний запуск застосунку (за допомогою команд `npm install` та `npm start`) для перевірки його працездатності перед інтеграцією DevSecOps-практик.

---

## Лабораторна робота №1: Налаштування SAST за допомогою SonarCloud

**Опис:** Інтеграція статичного аналізу коду (Static Application Security Testing) для автоматичного виявлення вразливостей, багів та "code smells" на найбільш ранніх етапах розробки (при кожному push або pull request).

### Хід виконання:
1. **Ініціалізація проєкту в SonarCloud:** Було створено новий проєкт у SonarCloud та пов'язано його з цільовим GitHub-репозиторієм.
   
   ![Налаштування SonarCloud](./images/Picture11.png)

2. **Налаштування секретів:** Для безпечної авторизації до GitHub Secrets було додано токен доступу `SONAR_TOKEN`.

   ![Додавання токена SONAR_TOKEN](./images/Picture12.png)

3. **Створення CI/CD Workflow:** Було написано конфігураційний файл `.github/workflows/sast.yml` для автоматичного запуску сканування під час подій `push` та `pull_request` у гілку `main`.

   ![Налаштування Workflow SAST](./images/Picture13.png)
   ![Виконання Workflow SAST](./images/Picture14.png)

**Лістинг файлу автоматизації SAST:**
```yml
name: SAST

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

env:
  NODE_DEFAULT_VERSION: 20
  ANGULAR_CLI_VERSION: 17

jobs:
  sonarcloud:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ env.NODE_DEFAULT_VERSION }}
        cache: npm

    - name: Install dependencies
      run: npm install --legacy-peer-deps --no-audit --no-fund --ignore-scripts

    - name: SonarCloud Scan
      uses: sonarsource/sonarcloud-github-action@v3
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

4. **Аналіз результатів:** Після успішного відпрацювання pipeline, результати сканування були перевірені у вкладці Security GitHub та на дашборді SonarCloud.

   ![Результати перевірки Security 1](./images/Picture15.png)
   ![Результати перевірки Security 2](./images/Picture16.png)

---

## Лабораторна робота №2: Аналіз залежностей (SCA) за допомогою Snyk

**Опис:** Впровадження Software Composition Analysis для перевірки всіх зовнішніх бібліотек та залежностей (npm-пакетів) на наявність відомих вразливостей (CVE).

### Хід виконання:
1. **Інтеграція зі Snyk:** Репозиторій було підключено до платформи Snyk для постійного моніторингу залежностей.

   ![Реєстрація у Snyk](./images/Picture21.png)

2. **Налаштування доступу:** Токен доступу `SNYK_TOKEN` успішно додано до GitHub Secrets.

   ![Додавання токена SNYK_TOKEN](./images/Picture22.png)

3. **Створення GitHub Actions Workflow:** Налаштовано конвеєр для автоматичної перевірки залежностей коду при змінах у репозиторії.

   ![Snyk Workflow процес 1](./images/Picture23.png)
   ![Snyk Workflow процес 2](./images/Picture24.png)

**Лістинг файлу автоматизації SCA:**
```yml
name: SOFTWARE COMPOSITION ANALYSIS using Snyk

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  snyk:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Run Snyk to check for vulnerabilities
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

4. **Усунення вразливостей:** Під час сканування було виявлено ряд вразливостей у залежностях, після чого було застосовано відповідні патчі (фікси) для їх усунення.

   ![Виявлені вразливості](./images/Picture25.png)
   ![Процес виправлення вразливостей](./images/Picture26.png)

---

## Лабораторна робота №3: Налаштування DAST за допомогою OWASP ZAP

**Опис:** Виконання динамічного тестування безпеки (Dynamic Application Security Testing) розгорнутого вебзастосунку для виявлення вразливостей під час його роботи.

### Хід виконання:
1. **Налаштування ZAP у GitHub Actions:** Додано конфігурацію для автоматичного запуску контейнера ZAProxy. Сценарій налаштовано на базове сканування цільового URL демо-версії застосунку.

   ![Конфігурація ZAP Actions](./images/Picture31.png)

**Лістинг файлу автоматизації DAST:**
```yml
name: DAST

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
  workflow_dispatch:

permissions:
  pull-requests: write
  issues: write

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    name: Scan the web application

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: ZAP Scan
        uses: zaproxy/action-baseline@v0.14.0
        with:
          target: '[https://demo.owasp-juice.shop](https://demo.owasp-juice.shop)'
          cmd_options: '-a'
```

2. **Отримання та аналіз звітів:** Після завершення роботи DAST-сканера було згенеровано детальні звіти щодо знайдених вразливостей конфігурації сервера та HTTP-заголовків.

   ![Звіт OWASP ZAP 1](./images/Picture32.png)
   ![Звіт OWASP ZAP 2](./images/Picture33.png)
   ![Звіт OWASP ZAP 3](./images/Picture34.png)

---

## Висновки та очікувані результати

1. **Автоматизація процесів безпеки:** Успішно впроваджено практики DevSecOps. Тепер усі етапи перевірки безпеки (SAST, SCA, DAST) виконуються автоматично в рамках єдиного CI/CD циклу на базі GitHub Actions.
2. **Звітування та моніторинг:** Налаштовано автоматичне генерування звітів про вразливості з їх подальшою візуалізацією у вкладці GitHub Security, SonarCloud та Snyk.
3. **Покращення безпеки:** Завдяки ранньому виявленню (Shift-Left Security) значно знижено ризик потрапляння вразливого коду або небезпечних залежностей у production-середовище.