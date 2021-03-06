Распаковка бинарного слепка VFS
===============================

Виртуальная файловая система (англ. virtual file system — VFS) распространённый
подход к построению интерфейса хранилищ и источников данных. Например, procfs,
FUSE, NFS, Samba. Эта задача составлена на основе формата, используемого в
проекте Облако Mail.Ru.

Для решения этой задачи вам предстоит написать преобразователь из бинарного
формата представления дерева файлов и каталогов во внутренние структуры PERL и
JSON.

Бинарный формат
---------------

Дерево сериализовано в последовательность команд. Команда кодируется одним байтом, за ней могут следовать аргументы, в зависимости от типа команды.

* D — создание директории. Любое непустое дерево должно начинаться с этой команды и следующей за ней командой I. За командой следуют следующие аргументы, в указанном порядке:
  + длинна имени директории — 2-х байтное беззнаковое целое, big endian.
  + имя директории — указанное в предыдущем поле число байт, utf8
  + права доступа — 2-х байтное беззнаковое целое, big endian.
* F — создание файла
  + длинна имени файла — 2-х байтное беззнаковое целое, big endian.
  + имя файла — указанное в предыдущем поле число байт, utf8
  + права доступа — 2-х байтное беззнаковое целое, big endian.
  + размер файла — 4-х байтное беззнаковое целое, big endian.
  + sha1 от содержимого файла — 20 байт (не hex!)
* I — спуск по дереву вниз, в директорию, созданную предыдущей командой. Вторая команда в любом не пустом дереве.
* U — подъём по дереву вверх. Подняться на уровень корневой директории можно только в предпоследней команде в последовательности.
* Z — Признак окончания дерева. Должно быть последней командой, в случае пустого дерева — единственной

Права доступа сериализуются по следующему правилу:

* other:
  + execute => 1 бит
  + write   => 2 бит
  + read    => 4 бит
* group
  + execute => 8 бит
  + write   => 16 бит
  + read    => 32 бит
* user
  + execute => 64 бит
  + write   => 128 бит
  + read    => 256 бит

Представление в JSON
--------------------

Пустое дерево сериализуется как пустой объект — «{}» Не пустое дерево представляет собой директорию, содержащую в поле list объекты уровнем ниже — директории и файлы Вложенные директории могут, в свою очередь, содержать вложенные объекты. Пример:

```json
{
   "type" : "directory",
   "name" : "root",
   "mode" : {
      "other" : {
         "execute" : true,
         "read" : true,
         "write" : false
      },
      "user" : {
         "write" : true,
         "execute" : true,
         "read" : true
      },
      "group" : {
         "write" : true,
         "execute" : true,
         "read" : true
      }
   },
   "list" : [
      {
         "type" : "directory",
         "name" : "Документы",
         "mode" : {
            "other" : {
               "execute" : true,
               "write" : false,
               "read" : true
            },
            "group" : {
               "execute" : true,
               "write" : true,
               "read" : true
            },
            "user" : {
               "execute" : true,
               "write" : true,
               "read" : true
            }
         },
         "list" : [
            {
               "type" : "file",
               "name" : "предложение о работе.docx",
               "mode" : {
                  "user" : {
                     "execute" : false,
                     "write" : true,
                     "read" : true
                  },
                  "group" : {
                     "execute" : false,
                     "write" : true,
                     "read" : true
                  },
                  "other" : {
                     "execute" : false,
                     "write" : false,
                     "read" : true
                  }
               },
               "size" : 168756,
               "hash" : "35472bcf3693bee9f3d23f0f07744f2be4741b2c"
            }
         ]
      }
   ]
}
```

Задание
-------

Файлы:
* lib/VFS.pm - место для реализации функции
* bin/vfs_dumper.pl - запуск поиска анаграмм по словарю dict.txt и печать результата.

Тест:
```bash
perl Makefile.PL && make test
```

Напишите функцию для распаковки бинарных данных.

Входные данные для функции:
* Для скрипта vfs_dumper.pl: файл с отпечатком VFS в бинарном формате, примеры вы можете найти в каталоге «data»
* Для функции распаковки: буфер с отпечатком VFS в бинарном формате (строка).

Выходные данные:
* Для скрипта vfs_dumper.pl: JSON описанного выше формата
* Для функции распаковки: хэш аналогичной JSON структуры, для представления true и false используйте JSON::XS::true и JSON::XS::False
