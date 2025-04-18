---
title: Спільний доступ до робочого столу через RDP
author: Ezequiel Bruni
contributors: Steven Spencer, Ganna Zhyrnova, Zhang Zhuyue
---

## Вступ

Якщо ви хочете поділитися своїм робочим столом (Gnome) у Rocky Linux або отримати доступ до інших спільних робочих столів, цей посібник для вас.

Для початківців ви збираєтеся використовувати RDP. RDP розшифровується як Remote Desktop Protocol, і він робить саме те, що це означає: він дозволяє переглядати комп’ютери та взаємодіяти з ними на відстані, і все це за допомогою графічного інтерфейсу. Однак ви повинні швидко зануритися в командний рядок, щоб налаштувати його.

!!! note "Примітка"

```
За замовчуванням Rocky Linux дозволяє надавати спільний доступ до робочого столу через інший протокол VNC. VNC досить зручний, але RDP зазвичай пропонує набагато більш плавний досвід і може працювати з дивною роздільною здатністю монітора.
```

## Припущення

Для цього посібника припускається, що ви вже налаштували наступне:

- Rocky Linux з Gnome
- Flatpak і Flathub встановлені та працюють
- Обліковий запис некореневого користувача
- Доступ адміністратора або sudo та готовність вставляти команди в термінал
- Сервер X (для спільного використання робочого столу)

!!! info "примітка"

```
Наразі реалізується кілька проектів, спрямованих на те, щоб сервер відображення Wayland і RDP добре працювали, а новіші версії Gnome постачаються з вбудованим сервером RDP, який справляється з цим. Однак версія Gnome від Rocky Linux не має такої функції, тому набагато простіше запускати сеанс RDP за допомогою x11.
```

## Спільне використання робочого столу Rocky Linux Gnome за допомогою RDP

Щоб зробити ваш робочий стіл Rocky Linux віддалено доступним, вам потрібен сервер RDP. Для наших цілей 'xrdp' буде більш ніж достатньо. Однак для цього вам потрібно буде використовувати термінал, оскільки це програма, яка працює лише з CLI.

!!! info "примітка"

````
Пакет xrdp знаходиться в [репозиторії EPEL](https://wiki.rockylinux.org/rocky/repo/#community-approved-repositories), який забезпечує перебір пакетів Fedora для всіх підтримуваних Enterprise Linux. Якщо ви не включили його, використовуйте наступні команди. Вам [также слід включити CRB](https://wiki.rockylinux.org/rocky/repo/#notes-on-epel) (називається «PowerTools» у Rocky Linux 8) перед додаванням репозиторію EPEL.

У Rocky Linux 8 використовуйте ці команди, щоб додати репозиторій EPEL:

```bash
sudo dnf config-manager --set-enabled powertools
sudo dnf install epel-release
```

У Rocky Linux 9 використовуйте ці команди, щоб додати репозиторій EPEL:

```bash
sudo dnf config-manager --set-enabled crb
sudo dnf install epel-release
```
````

Після додавання репозиторію EPEL (або якщо ви його вже додали), скористайтеся такою командою для встановлення xrdp:

```bash
sudo dnf install xrdp
```

Після того, як ви це встановили, вам потрібно ввімкнути службу:

```bash
sudo systemctl enable --now xrdp
```

Сервер RDP має бути встановлено, увімкнено та запущено, якщо все йде добре. Але ви ще не можете підключитися; вам потрібно спочатку відкрити правильний порт на брандмауері.

Якщо ви хочете дізнатися більше про роботу брандмауера Rocky Linux, `firewalld`, перегляньте наш [посібник для початківців з `firewalld`](../../guides/security/firewalld-beginners.md). Якщо ви просто хочете йти далі, виконайте ці команди:

```bash
sudo firewall-cmd --zone=public --add-port=3389/tcp --permanent
sudo firewall-cmd --reload
```

Для початківців ці команди відкривають порт RDP у вашому брандмауері, щоб ви могли приймати вхідні з’єднання RDP. Потім перезапустіть брандмауер, щоб застосувати зміни. Якщо ви відчуваєте таку схильність, ви можете перезавантажити свій комп'ютер просто для безпеки.

Ви можете вийти, якщо не хочете перезавантажуватись. RDP використовує облікові дані вашого облікового запису користувача для безпеки. Увійти віддалено, якщо ви вже ввійшли на свій робочий стіл локально, неможливо. Принаймні, не в тому самому обліковому записі користувача.

!!! info

```
Ви також можете використовувати програму Firewall, щоб керувати `firewalld` і відкривати будь-які порти, які хочете. Слідкуйте за посиланням на мій посібник із встановлення та використання програми Firewall.
```

## Доступ до робочого столу Rocky Linux та/або інших робочих столів за допомогою RDP

Ви бачили, як інсталювати сервер RDP, і тепер вам знадобиться клієнтська програма RDP. У Windows програма підключення до віддаленого робочого стола чудово справляється з цим завданням. Якщо ви хочете отримати доступ до своєї машини Rocky Linux з іншої машини Linux, вам потрібно буде встановити опцію третьої сторони.

У Gnome Remmina отримує мою найкращу рекомендацію. Він не складний у використанні; він стабільний і в цілому працює.

Якщо у вас встановлено Flatpak/Flathub, відкрийте програму Software і знайдіть Remmina.

![The Gnome Software app on the Remmina page](images/rdp_images/01-remmina.png)

Встановіть його та запустіть. Важливо: це процес додавання з’єднання RDP у Remmina, але він подібний майже до будь-якої іншої клієнтської програми RDP, яку ви, ймовірно, знайдете.

Натисніть кнопку з плюсом у верхньому лівому куті, щоб додати з’єднання. У полі імені назвіть його як завгодно та введіть IP-адресу віддаленого комп’ютера, а також ім’я користувача та пароль віддаленого облікового запису користувача. Пам’ятайте: якщо ваші комп’ютери в одній мережі, вам потрібно використовувати їх локальну IP-адресу, а не ту, яку ви бачите на сайті, наприклад «whatsmyip.com».

![The Remmina connection profile form](images/rdp_images/02-remmina-config.png)

Якщо ваші комп’ютери знаходяться в різних мережах, я сподіваюся, що ви знаєте, як зробити переадресацію портів, або що віддалений комп’ютер має якусь статичну IP-адресу. Це все виходить за рамки цього документа.

Прокрутіть униз, щоб переглянути такі параметри, як підтримка кількох моніторів, користувацька роздільна здатність тощо. Крім того, параметр «Тип підключення до мережі» у вашому клієнті RDP дозволяє збалансувати використання пропускної здатності та якість зображення.

Використовуйте локальну мережу для найкращої якості, якщо ваші комп’ютери в одній мережі.

Потім натисніть ++"Save"++ і ++"Connect"++.

Ось як це виглядає з клієнтом підключення до віддаленого робочого стола Windows. Автор написав цей документ на своєму локальному сервері Rocky Linux за допомогою RDP.

![A screenshot of my docs-writing environment, at a 5120x1440p resolution](images/rdp_images/03-rdp-connection.jpg)

## Висновок

Це все, що вам потрібно знати, щоб запустити RDP на Rocky Linux і поділитися робочим столом. Це допоможе, якщо вам потрібен лише віддалений доступ до деяких файлів і програм.
