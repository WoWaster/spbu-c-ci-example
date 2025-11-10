# Пример CI для C

Инструменты: CMake, clang-format, clang-tidy.
(Скорее всего потребуется установить самостоятельно)

Стиль кода: WebKit.
Соответствующий стиль включен в clang-format и проверки нейминга в clang-tidy.

## Запуск руками

### clang-format

Для запуска проверки стиля через clang-format можно использовать заклинание:
```console
$ find . -path ./build -prune -o -type f -name '*.[c|h]' -print | xargs clang-format --style=file --dry-run -Werror
```

Для форматирование запускать:
```console
$ find . -path ./build -prune -o -type f -name '*.[c|h]' -print | xargs clang-format --style=file -i
```

### Сборка

Пример собирается командами:
```console
$ cmake . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
$ cmake --build build
```

Флаг `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` потребуется для использования clang-tidy.

### clang-tidy

clang-tidy запускается командой:
```console
$ run-clang-tidy -p=build
```

## Использование в своём репозитории

Для начала стоит удостоверится, что весь проект собирается из корня общим `CMakeLists.txt`.
Затем нужно перенести себе файлы:
```
.
├── .clang-format
├── .clang-tidy
└── .github
    ├── dependabot.yml
    └── workflows
        └── build-and-lint.yml
```

Также может быть полезно добавить в `.gitignore` директории `.cache` и `build`.

### В случае отсутствия CMake

В случае отсутствия CMake в репозитории использовать clang-tidy просто так не выйдет.
Тем не менее clang-format тоже довольно полезен.

Для настройки нужно перенести себе файлы:
```
.
├── .clang-format
└── .github
    ├── dependabot.yml
    └── workflows
        └── build-and-lint.yml
```

А также исправить файл `build-and-lint.yml`
```diff
--- a/.github/workflows/build-and-lint.yml
+++ b/.github/workflows/build-and-lint.yml
@@ -12,13 +12,3 @@ jobs:
       - name: Check format
         run: |
           find . -path ./build -prune -o -type f -name '*.[c|h]' -print | xargs clang-format-18 --style=file --dry-run -Werror
-      - name: Configure
-        run: |
-          # -DCMAKE_EXPORT_COMPILE_COMMANDS=ON is important for clang-tidy
-          cmake . -B build -DCMAKE_EXPORT_COMPILE_COMMANDS=ON
-      - name: Build
-        run: |
-          cmake --build build
-      - name: Lint
-        run: |
-          run-clang-tidy-18 -p=build
```
