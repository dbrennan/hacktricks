<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


# Відмітки часу

Атакуючому може бути цікаво **змінити відмітки часу файлів**, щоб уникнути виявлення.\
Можна знайти відмітки часу всередині MFT в атрибутах `$STANDARD_INFORMATION` __ та __ `$FILE_NAME`.

Обидва атрибути мають 4 відмітки часу: **Модифікація**, **доступ**, **створення** та **модифікація реєстру MFT** (MACE або MACB).

**Провідник Windows** та інші інструменти показують інформацію з **`$STANDARD_INFORMATION`**.

## TimeStomp - Анти-форензичний Інструмент

Цей інструмент **змінює** інформацію про відмітки часу всередині **`$STANDARD_INFORMATION`** **але** **не** інформацію всередині **`$FILE_NAME`**. Тому можна **виявити** **підозрілу** **активність**.

## Usnjrnl

**Журнал USN** (Журнал номерів послідовності оновлень) - це функція файлової системи NTFS (система файлів Windows NT), яка відстежує зміни обсягу. Інструмент [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) дозволяє переглядати ці зміни.

![](<../../.gitbook/assets/image (449).png>)

Попереднє зображення - це **вихід**, показаний **інструментом**, де можна побачити, що до файлу було внесено деякі **зміни**.

## $LogFile

**Усі зміни метаданих в файловій системі реєструються** в процесі, відомому як [логування перед записом](https://en.wikipedia.org/wiki/Write-ahead_logging). Зареєстровані метадані зберігаються в файлі з назвою `**$LogFile**`, розташованому в кореневому каталозі файлової системи NTFS. Інструменти, такі як [LogFileParser](https://github.com/jschicht/LogFileParser), можуть використовуватися для розбору цього файлу та ідентифікації змін.

![](<../../.gitbook/assets/image (450).png>)

Знову ж таки, у виході інструменту можна побачити, що **було виконано деякі зміни**.

За допомогою цього ж інструменту можна визначити, **коли були змінені відмітки часу**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Час створення файлу
* ATIME: Час модифікації файлу
* MTIME: Час модифікації реєстру MFT файлу
* RTIME: Час доступу до файлу

## Порівняння `$STANDARD_INFORMATION` та `$FILE_NAME`

Ще один спосіб виявлення підозрілих змінених файлів - порівняти час в обох атрибутах, шукаючи **неспівпадіння**.

## Наносекунди

Відмітки часу **NTFS** мають **точність** **100 наносекунд**. Тому знаходження файлів з відмітками часу, наприклад, 2010-10-10 10:10:**00.000:0000, є дуже підозрілим**.

## SetMace - Анти-форензичний Інструмент

Цей інструмент може змінювати обидва атрибути `$STARNDAR_INFORMATION` та `$FILE_NAME`. Однак починаючи з Windows Vista, для зміни цієї інформації потрібна жива ОС.

# Приховування Даних

NTFS використовує кластер та мінімальний розмір інформації. Це означає, що якщо файл займає кластер та півтора, **залишкова половина ніколи не буде використана**, поки файл не буде видалено. Тому можна **приховати дані в цьому вільному просторі**.

Є інструменти, такі як slacker, які дозволяють приховувати дані в цьому "прихованому" просторі. Однак аналіз `$logfile` та `$usnjrnl` може показати, що деякі дані було додано:

![](<../../.gitbook/assets/image (452).png>)

Тоді можна відновити вільний простір за допомогою інструментів, таких як FTK Imager. Зверніть увагу, що цей тип інструменту може зберігати вміст в обфускованому або навіть зашифрованому вигляді.

# UsbKill

Це інструмент, який **вимкне комп'ютер, якщо виявлені будь-які зміни в USB** портах.\
Щоб виявити це, слід перевірити запущені процеси та **переглянути кожний запущений скрипт Python**.

# Живі Дистрибутиви Linux

Ці дистрибутиви **виконуються всередині оперативної пам'яті RAM**. Єдиний спосіб виявити їх - **у випадку, якщо файлова система NTFS монтується з правами на запис**. Якщо вона монтується лише з правами на читання, то виявити вторгнення буде неможливо.

# Безпечне Видалення

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Конфігурація Windows

Можливо вимкнути кілька методів журналювання Windows, щоб ускладнити розслідування форензики.

## Вимкнення Відміток Часу - UserAssist

Це реєстровий ключ, який зберігає дати та години, коли кожний виконуваний файл був запущений користувачем.

Для вимкнення UserAssist потрібно виконати два кроки:

1. Встановіть два реєстрових ключа, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` та `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, обидва на нуль, щоб позначити, що ми хочемо вимкнути UserAssist.
2. Очистіть гілки реєстру, які виглядають як `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<хеш>`.

## Вимкнення Відміток Часу - Prefetch

Це збереже інформацію про виконані програми з метою покращення продуктивності системи Windows. Однак це також може бути корисним для практик форензики.

* Виконайте `regedit`
* Виберіть шлях файлу `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Клацніть правою кнопкою миші на `EnablePrefetcher` та `EnableSuperfetch`
* Виберіть Змінити для кожного з них, щоб змінити значення з 1 (або 3) на 0
* Перезавантажте

## Вимкнення Відміток Часу - Останній Час Доступу

Кожного разу, коли відкривається папка з тома NTFS на сервері Windows NT, система витрачає час на **оновлення полі відмітки часу на кожній перерахованій папці**, яке називається останній час доступу. На сильно використовуваному томі NTFS це може вплинути на продуктивність.

1. Відкрийте Редактор реєстру (Regedit.exe).
2. Перейдіть до `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Знайдіть `NtfsDisableLastAccessUpdate`. Якщо його немає, додайте цей DWORD та встановіть його значення на 1, що вимкне процес.
4. Закрийте Редактор реєстру та перезавантажте сервер.
## Видалення історії USB

Всі **записи пристроїв USB** зберігаються в реєстрі Windows під ключем реєстру **USBSTOR**, який містить підключі, що створюються кожного разу, коли ви підключаєте пристрій USB до свого ПК або ноутбука. Ви можете знайти цей ключ тут `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Видалення цього** ключа призведе до видалення історії USB.\
Ви також можете скористатися інструментом [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html), щоб переконатися, що ви їх видалили (і видалити їх).

Іншим файлом, який зберігає інформацію про USB, є файл `setupapi.dev.log` всередині `C:\Windows\INF`. Його також слід видалити.

## Вимкнення тіньових копій

**Вивести** тіньові копії за допомогою `vssadmin list shadowstorage`\
**Видалити** їх, виконавши `vssadmin delete shadow`

Ви також можете видалити їх через GUI, слідуючи за кроками, запропонованими за посиланням [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Щоб вимкнути тіньові копії [кроки звідси](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows):

1. Відкрийте програму Служби, набравши "services" у текстовому полі пошуку після натискання кнопки запуску Windows.
2. Зі списку знайдіть "Тіньову копію тому", виберіть її, а потім отримайте доступ до властивостей, клацнувши правою кнопкою миші.
3. Виберіть "Вимкнено" зі списку випадаючих меню "Тип запуску", а потім підтвердіть зміну, клацнувши "Застосувати" і "OK".

Також можливо змінити конфігурацію файлів, які будуть скопійовані в тіньову копію, у реєстрі `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

## Перезапис видалених файлів

* Ви можете скористатися **інструментом Windows**: `cipher /w:C`. Це вказує cipher видалити будь-які дані з вільного місця на диску в диску C.
* Ви також можете скористатися інструментами, такими як [**Eraser**](https://eraser.heidi.ie)

## Видалення журналів подій Windows

* Windows + R --> eventvwr.msc --> Розгорнути "Журнали Windows" --> Клацніть правою кнопкою миші на кожній категорії та виберіть "Очистити журнал"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Вимкнення журналів подій Windows

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* У розділі служб вимкніть службу "Журнал подій Windows"
* `WEvtUtil.exec clear-log` або `WEvtUtil.exe cl`

## Вимкнення $UsnJrnl

* `fsutil usn deletejournal /d c:`