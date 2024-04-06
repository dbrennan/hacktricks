<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>


# ECB

(ECB) Electronic Code Book - симетрична схема шифрування, яка **замінює кожний блок відкритого тексту** на **блок шифротексту**. Це **найпростіша** схема шифрування. Основна ідея полягає в тому, щоб **розділити** відкритий текст на **блоки по N біт** (залежно від розміру блоку вхідних даних, алгоритм шифрування), а потім зашифрувати (розшифрувати) кожен блок відкритого тексту, використовуючи лише ключ.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

Використання ECB має кілька наслідків з точки зору безпеки:

* **Блоки з зашифрованого повідомлення можуть бути видалені**
* **Блоки з зашифрованого повідомлення можуть бути переміщені**

# Виявлення вразливості

Уявіть, що ви увійшли в додаток кілька разів і ви **завжди отримуєте одне й те ж печиво**. Це тому, що печиво додатка має формат **`<ім'я користувача>|<пароль>`**.\
Потім ви створюєте двох нових користувачів, обоє з **однаковим довгим паролем** та **майже** **однаковим** **ім'ям користувача**.\
Ви виявляєте, що **блоки по 8 байтів**, де **інформація обох користувачів** однакова, **рівні**. Тоді ви уявляєте, що це може бути через використання **ECB**.

Подібно до наступного прикладу. Приділіть увагу тому, що ці **2 розкодовані печива** мають кілька разів блок **`\x23U\xE45K\xCB\x21\xC8`**
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
Це тому, що **ім'я користувача та пароль цих файлів cookie містять кілька разів літеру "a"** (наприклад). **Блоки**, які **відрізняються**, - це блоки, які містять **принаймні 1 відмінний символ** (можливо, роздільник "|" або якась необхідна різниця в імені користувача).

Тепер зловмиснику просто потрібно виявити, чи формат `<ім'я користувача><роздільник><пароль>` чи `<пароль><роздільник><ім'я користувача>`. Для цього він може просто **створити кілька імен користувачів** з **схожими та довгими іменами користувачів та паролями, поки він не знайде формат та довжину роздільника:**

| Довжина імені користувача: | Довжина пароля: | Довжина Ім'я користувача + Пароль: | Довжина файлу cookie (після декодування): |
| ---------------- | ---------------- | ------------------------- | --------------------------------- |
| 2                | 2                | 4                         | 8                                 |
| 3                | 3                | 6                         | 8                                 |
| 3                | 4                | 7                         | 8                                 |
| 4                | 4                | 8                         | 16                                |
| 7                | 7                | 14                        | 16                                |

# Використання вразливості

## Видалення цілих блоків

Знаючи формат файлу cookie (`<ім'я користувача>|<пароль>`), для того щоб видаляти ім'я користувача `admin`, створіть нового користувача з ім'ям `aaaaaaaaadmin` та отримайте файл cookie та розкодуйте його:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
Ми можемо побачити шаблон `\x23U\xE45K\xCB\x21\xC8`, створений раніше з ім'ям користувача, яке містить лише `a`.\
Потім ви можете видалити перший блок 8B і отримаєте дійсний куки для імені користувача `admin`:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## Переміщення блоків

У багатьох базах даних однаково шукати `WHERE username='admin';` або `WHERE username='admin    ';` _(Зверніть увагу на додаткові пробіли)_

Таким чином, інший спосіб підробити користувача `admin` буде:

* Створити ім'я користувача таке, що: `len(<username>) + len(<delimiter) % len(block)`. З розміром блоку `8B` ви можете створити ім'я користувача під назвою: `username       `, з роздільником `|` частина `<username><delimiter>` створить 2 блоки з 8B.
* Потім створити пароль, який заповнить точну кількість блоків, що містять ім'я користувача, яке ми хочемо підробити, та пробіли, наприклад: `admin   `

Cookie цього користувача буде складатися з 3 блоків: перші 2 блоки - блоки імені користувача + роздільник, а третій - пароль (який підробляє ім'я користувача): `username       |admin   `

**Потім просто замініть перший блок останнім разом і ви підробляєте користувача `admin`: `admin          |username`**

## Посилання

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))