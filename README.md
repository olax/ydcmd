# ydcmd

Консольный клиент Linux/FreeBSD для работы с облачным хранилищем [Яндекс.Диск](https://disk.yandex.ru/) посредством [REST API](http://api.yandex.ru/disk/api/concepts/about.xml).

## Подготовка к работе

Для работы клиента необходимо получить отладочный OAuth токен. Для его получения, [зарегистрируйте приложение на Яндексе](https://oauth.yandex.ru/client/new):

* `Название` - `ydcmd` (может быть любым)
* `Права` - `Яндекс.Диск REST API`
* `Клиент для разработки` - установить флажок

После регистрации приложения скопируйте `id приложения` и перейдите по ссылке:

* `https://oauth.yandex.ru/authorize?response_type=token&client_id=<id_приложения>`

После разрешения доступа сервис перенаправит вас по ссылке вида:

* `https://oauth.yandex.ru/verification_code?dev=True#access_token=<токен>`

Значение "токен" и есть требуемое. Подробнее можно ознакомиться по ссылке [получение отладочного токена вручную](http://api.yandex.ru/oauth/doc/dg/tasks/get-oauth-token.xml).

## Конфигурационный файл

Для удобства работы рекомендуется создать конфигурационный файл с именем `.ydcmd.cfg` в домашней директории пользователя и установить на него права `0600`. Формат файла:

```
[ydcmd]
# комментарий
<option>=<value>
```

Дополнительно можно задать значения параметров по умолчанию из командной строки (см. ниже).

## Работа

Вывод краткой справки в консоли можно получить запуском скрипта без параметров или с командой `help`. Общий формат вызова:

```
ydcmd [команда] [опции] [аргументы]
```

Команды:

* `help` - получение краткой справки по командам и опциям приложения;
* `ls` - получение списка файлов и директорий;
* `rm` - удаление файла или директории;
* `cp` - копирование файла или директории;
* `mv` - перемещение файла или директории;
* `put` - загрузка файла в хранилище;
* `get` - получение файла или директории из хранилища;
* `mkdir` - создание директории;
* `stat` - получение метаинформации об объекте.

Опции:

* `--timeout=<N>` - таймаут в секундах выполнения операции;
* `--retries=<N>` - количество перезапросов api;
* `--delay=<N>` - таймаут между перезапросами api в секундах;
* `--token=<S>` - oauth токен, в целях безопасности рекомендуется указывать в конфигурационном файле;
* `--quiet` - подавление вывода об ошибках, результат успеха операции определяется по коду возврата;
* `--verbose` - вывод расширенной информации;
* `--debug` - вывод отладочной информации;
* `--chunk=<N>` - размер блока данных в КБ для операций ввода/вывода;
* `--ca-file=<S>` - имя файла с сертификатами доверенных CA (при пустом значении проверка валидности сертификата не производится);
* `--ciphers=<S>` - набор алгоритмов шифрования.

### Получение списка файлов и директорий

```
ydcmd ls [опции] [disk:/объект]
```

Опции:

* `--human` - вывод размера файла в человеко-читаемом виде;
* `--short` - вывод списка файлов и директорий без дополнительной информации (одно имя в одну строку);
* `--long` - вывод расширенного списка (время создания, время модификации, размер, имя файла);
* `--limit` - количество строк запрашиваемое за один вызов api (может как ускорить получение длинных списков, так и приводить к таймауту операции).

Обратите внимание:

* Если целевой объект не указан, то будет использоваться корневая директория хранилища.

### Удаление файла или директории

```
ydcmd rm disk:/объект
```

Опции:

* `--poll=<N>` - время в секундах между опросом состояния при выполнении асинхронных операций.

Обратите внимание:

* Файлы удаляются без возможности восстановления;
* Директории удаляются рекурсивно (включая вложенные файлы и директории).

### Копирование файла или директории

```
ydcmd cp disk:/объект1 disk:/объект2
```

Опции:

* `--poll=<N>` - время в секундах между опросом состояния при выполнении асинхронных операций.

Обратите внимание:

* В случае совпадения имен, директории и файлы будут перезаписаны;
* Директории копируются рекурсивно (включая вложенные файлы и директории).

### Перемещение файла или директории

```
ydcmd mv disk:/объект1 disk:/объект2
```

Опции:

* `--poll=<N>` - время в секундах между опросом состояния при выполнении асинхронных операций.

Обратите внимание:

* В случае совпадения имени, директории и файлы будут перезаписаны.

### Загрузка файла в хранилище

```
ydcmd put <файл> [disk:/объект]
```

Опции:

* `--rsync` - синхронизация дерева файлов и директорий в хранилище с локальным деревом;

Обратите внимание:

* Если целевой объект не указан, то для загрузки файла будет использоваться корневая директория хранилища;
* Если целевой объект указывает на директорию (заканчивается на `/`), то к имени директории будет добавлено имя исходного файла;
* Если целевой объект существует, то он будет перезаписан без запроса подтверждения;
* Символические ссылки игнорируются.

### Получение файла из хранилища

```
ydcmd get <disk:/объект> [файл]
```

Опции:

* `--rsync` - синхронизация локального дерева файлов и директорий с деревом в хранилище;

Обратите внимание:

* Если не указано имя целевого файла, будет использовано имя файла в хранилище;
* Если целевой объект существует, то он будет перезаписан без запроса подтверждения.

### Создание директории

```
ydcmd mkdir disk:/путь
```

### Получение метаинформации об объекте

```
ydcmd stat [disk:/объект]
```

Обратите внимание:

* Если целевой объект не указан, то будет использоваться корневая директория хранилища.

### Расширенные опции

* `--async` - выполнение асинхронной команды (`rm`, `mv`, `cp`) без ожидания завершения (`poll`) операции.
