# Разработка

## Требования

- Windows: проект собирается под `net48` с WPF (`UseWPF`).
- .NET SDK 10.0.x: такая версия стоит в [CI](../.github/workflows/ci.yml) и в [release.yml](../.github/workflows/release.yml). Язык C# 10 (`LangVersion`).
- NuGet-пакеты подтягиваются при сборке: `Ascon.Pilot.SDK` и `Ascon.Pilot.Bim.SDK` 26.15.0, `SimpleInjector`, `Svg`, `Microsoft.Xaml.Behaviors.Wpf`, `System.Composition`, `Microsoft.CSharp`.
- Для отладки: установленный клиент Pilot-BIM или Pilot-ICE Enterprise.

## Структура

| Путь | Что там |
|---|---|
| `PilotLookUp.sln` | Решение, содержит только `src/PilotLookUp.csproj` |
| `src/PilotLookUp.csproj` | Библиотека `PilotLookUp.ext2.dll` (`AssemblyName`), версия в `AssemblyVersion` |
| `src/App.cs` | Точка входа расширения: MEF-экспорт `IMenu<...>` и `IToolbar<...>` для контекстов Pilot |
| `src/Model` | `ServiceContainer` (SimpleInjector), сервисы (`Services`) и фабрики (`Factories`) |
| `src/Interfaces` | Интерфейсы сервисов и фабрик |
| `src/Core`, `src/Objects` | Загрузка и просмотр объектов Pilot (`ObjReflection`, `PilotObjectHelper`, помощники по типам в `Objects/TypeHelpers`) |
| `src/View`, `src/ViewModel` | WPF-окна и страницы (XAML) и их модели представления |
| `src/Resources` | Стили и иконки (подключены как `Resource`) |
| `tests/PilotLookUp.Tests` | Тесты, которые выполняются внутри клиента Pilot (см. ниже) |

## Сборка

```
dotnet build src/PilotLookUp.csproj -c Release
```

Результат: `bin/Release/net48/PilotLookUp.ext2.dll` с зависимостями и архив `bin/Release/PilotLookUp.zip` (создает таргет `CreateZip`, подробнее в [releasing.md](releasing.md)). Сборка выдает предупреждения об устаревших API SDK (`ISignature`, `IAttribute.Configuration`, `TaskFields`), ошибок нет.

Сборка Release на каждый push в `master` и на каждый pull request проверяется в CI (`.github/workflows/ci.yml`). Тесты в CI не запускаются.

## Отладка в Pilot-BIM и Pilot-ICE

Сборка Debug (`dotnet build src/PilotLookUp.csproj` или Visual Studio) кладет результат в `bin/Debug/net48`, а таргет `CopyToDebugDirectory` после сборки копирует все файлы из выходной папки в папки расширений клиентов (папки создаются, если их нет):

- `%USERPROFILE%\AppData\Local\ASCON\Pilot-BIM\Development\PilotLookUp`
- `%USERPROFILE%\AppData\Local\ASCON\Pilot-ICE Enterprise\Development\PilotLookUp`

Это папки `Development` из [инструкции по установке](../README.md#установка). Копирование выполняется в обе папки независимо от того, какой клиент установлен.

В конфигурации Debug в csproj задан `StartProgram` - `C:\Program Files\ASCON\Pilot-BIM\Ascon.Pilot.PilotBIM.exe`: запуск отладки из Visual Studio стартует Pilot-BIM.

## Тесты

Проект `tests/PilotLookUp.Tests` использует `[PilotFact]` и `ITestContext` из `PilotTestHost.Sdk`: тесты выполняются внутри живого клиента Pilot и обращаются к реальному `IObjectsRepository`. Поэтому:

- в `PilotLookUp.sln` проект тестов не входит, а `dotnet build` для него требует внешнего репозитория `PilotTestHost`: `ProjectReference` указывает на `..\..\..\..\PilotTestHost\src\PilotTestHost.Sdk\PilotTestHost.Sdk.csproj`, путь считается от папки проекта тестов (четыре уровня вверх, затем `PilotTestHost\src\...`), то есть `PilotTestHost` должен лежать вне этого репозитория;
- сборка Debug тестов копирует `PilotLookUp.Tests.dll` и `PilotLookUp.ext2.dll` в `%USERPROFILE%\AppData\Local\ASCON\Pilot-BIM\Development\PilotTestHost\Tests` и в такую же папку Pilot-ICE Enterprise;
- запуск тестов выполняется средствами PilotTestHost в клиенте, в CI они не гоняются.
