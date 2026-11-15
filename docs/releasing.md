# Выпуск релиза

Релиз публикуется автоматически workflow [`release.yml`](../.github/workflows/release.yml) при пуше тега-версии. Версия состоит из четырех чисел через точку, например `2.8.1.0`.

## Порядок

1. Поднять `AssemblyVersion` в `src/PilotLookUp.csproj` и закоммитить.
2. Создать аннотированный тег с описанием изменений:
   ```
   git tag -a 2.8.1.0 -m "- что изменилось"
   ```
   Текст тега станет описанием релиза. У легковесного тега описанием будет сообщение коммита.
3. Отправить тег:
   ```
   git push origin 2.8.1.0
   ```

## Что делает workflow

Запускается на windows-latest с .NET SDK 10.0.x и выполняет шаги:

1. Проверяет, что имя тега совпадает с `AssemblyVersion` в `src/PilotLookUp.csproj`. При расхождении сборка падает.
2. Собирает `dotnet build src/PilotLookUp.csproj -c Release`. Таргет `CreateZip` после сборки создает `bin/Release/PilotLookUp.zip`.
3. Проверяет результат:
   - версия `bin/Release/net48/PilotLookUp.ext2.dll` равна имени тега;
   - в корне архива есть `PilotLookUp.ext2.dll`;
   - в архиве нет файлов `.pdb`.
4. Подтягивает тег как аннотированный и создает GitHub Release `PilotLookUp <тег>` с файлом `PilotLookUp.zip` и описанием из тега (`--notes-from-tag`).

Тесты в workflow не запускаются: им нужен живой клиент Pilot. Перед тегом проверьте сборку локально или по результату CI (`ci.yml` выполняет те же сборку и проверку архива, но сверяет версию DLL с `AssemblyVersion`, а не с тегом).

## Что попадает в архив

`CreateZip` работает только в конфигурации Release. Он удаляет `*.pdb` и `*.xml` из корня выходной папки `bin/Release/net48` и упаковывает ее содержимое (`Compress-Archive`) в `bin/Release/PilotLookUp.zip`.

Состав архива при проверочной сборке версии 2.8.1.0:

- `PilotLookUp.ext2.dll`;
- зависимости из NuGet: `Ascon.Pilot.Bim.SDK.dll`, `Ascon.Pilot.Bim.Search.SDK.dll`, `Ascon.Pilot.SDK.dll`, `Ascon.Pilot.SDK.Common.dll`, `Ascon.Pilot.Theme.dll`, `ExCSS.dll`, `Microsoft.Xaml.Behaviors.dll`, `Newtonsoft.Json.dll`, `SimpleInjector.dll`, `Svg.dll`, `System.Buffers.dll`, `System.Composition.*.dll`, `System.Data.SQLite.dll`, `System.Memory.dll`, `System.Numerics.Vectors.dll`, `System.Runtime.CompilerServices.Unsafe.dll`;
- `x64\SQLite.Interop.dll` и `x86\SQLite.Interop.dll`.

Точный состав зависит от версий пакетов в `PilotLookUp.csproj`. Установка архива описана в [README](../README.md#установка).
