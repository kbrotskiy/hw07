## Homework VII

### Задание
Создадим настройки менеджера пакетов **Hunter** для репозитория с ДЗ № 5
Настройка git-репозитория **hw07** для работы

```sh
% git clone https://github.com/${GITHUB_USERNAME}/hw05 hw07
% cd hw07
% git remote remove origin
% git remote add origin https://github.com/${GITHUB_USERNAME}/lab07

```
Скачивание и подключение модуля `HunterGate`
```sh
% mkdir -p cmake # Создание директории где будут храниться файлы Hunter
% wget https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake -O cmake/HunterGate.cmake
% gsed -i "" '/cmake_minimum_required(VERSION 3.10)/a\
include("cmake/HunterGate.cmake")
HunterGate(
    URL "https:\//github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
)
' CMakeLists.txt

```
Теперь не нужно скачивать **GTest** самостоятельно. **Hunter** сам подтянет добавленные с помощью функции `hunter_add_package`.
```sh
% git rm -rf third-party/gtest
rm 'third-party/gtest'
% gsed -i "" '/option(BUILD_TESTS "Build tests" OFF)/a\

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
' CMakeLists.txt
% gsed -i "" 's/add_subdirectory(third-party/gtest)//' CMakeLists.txt
% gsed -i "" 's/gtest_main/GTest::gtest_main/' CMakeLists.txt

```
Сборка прокта при помощи **Hunter**.
```sh

% cmake -H. -B_builds -DBUILD_TESTS=ON
% cmake --build _builds --target test
% ls -la $HOME/.hunter

```sh
% mkdir cmake/Hunter
% cat > cmake/Hunter/config.cmake <<EOF
hunter_config(GTest VERSION 1.7.0-hunter-9)
EOF
# add LOCAL in HunterGate function
```

Добавление подмодуля **polly**, который содержит инструкции для сборки проектов с установленным **Hunter**.
```sh
% mkdir tools
% git submodule add https://github.com/ruslo/polly tools/polly
% tools/polly/bin/polly.py --test
% tools/polly/bin/polly.py --toolchain clang-cxx14

```
Добавим непрерывную интеграцию с **Appveyor**
Создание `appveyor.yml`
```sh
% cat >> appveyor.yml <<EOF
image: Visual Studio 2019
platform:
  - x86
  - x64
configuration: Release

build_script:
  - cmd: cmake -H. -B_build -DBUILD_TESTS=ON
  - cmd: cmake --build _build
  - cmd: cmake --build _build --target test
  - cmd: _build/check
  - cmd: cmake --build _build --target test -- ARGS=--verbose
EOF
```

## Links

- [Create Hunter package](https://docs.hunter.sh/en/latest/creating-new/create.html)
- [Custom Hunter config](https://github.com/ruslo/hunter/wiki/example.custom.config.id)
- [Polly](https://github.com/ruslo/polly)

```
Copyright (c) 2015-2020 The ISC Authors
```
