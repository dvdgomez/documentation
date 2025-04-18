---
title: Встановлення та налаштування GitHub CLI на Rocky Linux
author: Wale Soyinka
Contributor: Ganna Zhyrnova
tags:
  - GitHub CLI
  - gh
  - git
  - github
---

## Вступ

Цей Gemstone охоплює встановлення та базове налаштування інструменту GitHub CLI (gh) у системі Rocky Linux. Цей інструмент дозволяє користувачам взаємодіяти зі сховищами GitHub безпосередньо з командного рядка.

## Опис проблеми

Користувачам потрібен зручний спосіб взаємодії з GitHub, не виходячи з середовища командного рядка.

## Передумови

- Система під керуванням Rocky Linux
- Доступ до терміналу
- Базове знайомство з операціями командного рядка
- Наявний обліковий запис Github

## Процедура

1. **Встановіть репозиторій GitHub CLI за допомогою curl**:
  Використовуйте команду curl, щоб завантажити офіційний файл сховища для `gh`. Завантажений файл буде збережено в каталозі /etc/yum.repos.d/. Після завантаження використовуйте команду dnf, щоб установити `gh` зі сховища. Впишіть:

  ```bash
  curl -fsSL https://cli.github.com/packages/rpm/gh-cli.repo | sudo tee /etc/yum.repos.d/github-cli.repo
  sudo dnf -y install gh
  ```

2. **Перевірте встановлення**:
  Переконайтеся, що `gh` встановлено правильно. Впишіть:

  ```bash
  gh --version
  ```

3. **Автентифікація за допомогою GitHub**:
  Увійдіть у свій обліковий запис GitHub. Впишіть:

  ```bash
  gh auth login
  ```

  Дотримуйтеся вказівок для автентифікації.

## Висновок

Тепер у вашій системі Rocky Linux 9.3 має бути встановлено та налаштовано GitHub CLI, що дозволить вам взаємодіяти зі сховищами GitHub безпосередньо з вашого терміналу.

## Додаткова інформація (необов'язково)

- GitHub CLI надає різні команди, такі як `gh repo clone`, `gh pr create`, `gh issue list` тощо.
- Щоб дізнатися більше про використання, зверніться до [офіційної документації GitHub CLI] (https://cli.github.com/manual/).
