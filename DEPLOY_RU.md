# Развёртывание форка Lense/netbox-sync — инструкция

## Шаг 0. Откатить то, что уже сделано

Вы выполнили только:
```
git clone https://github.com/Soldier2003/netbox-sync.git
cd netbox-sync
git checkout -b platformix-main
```
Коммитов и `git push` не было — значит **на GitHub (Soldier2003/netbox-sync)
ничего не менялось**, ваш форк там по-прежнему чистая копия оригинала.
Всё, что нужно откатить, — это локальная папка на вашем компьютере.

1. В открытом терминале посмотрите, где вы находитесь:
   ```
   cd
   ```
   (в PowerShell это покажет текущий путь — например
   `C:\Windows\system32\netbox-sync`, если терминал был открыт "от имени
   администратора" и вы никуда не переходили перед `git clone`, либо
   `C:\Users\<вы>\netbox-sync`, если переходили).
2. Поднимитесь на уровень выше и удалите папку `netbox-sync` целиком:
   ```
   cd ..
   Remove-Item -Recurse -Force netbox-sync
   ```
   Если папка оказалась внутри `C:\Windows\System32` — тоже спокойно
   удаляйте, права администратора это позволят, ничего системного вы не
   трогали (просто новая папка со свежим `git clone`).
3. Дальше пункт 1 ниже — начинаем с нуля, уже без admin-терминала (для
   `git clone` и работы с файлами права администратора не нужны).

## Шаг 1. Клонировать форк заново, в понятное место

Откройте обычный (не admin) PowerShell и выполните:

```powershell
cd D:\Работа\Platformix\demo\netbox_claude
git clone https://github.com/Soldier2003/netbox-sync.git netbox-sync-src
cd netbox-sync-src
git checkout -b lense-main
```

Теперь у вас есть папка:
```
D:\Работа\Platformix\demo\netbox_claude\netbox-sync-src\
```
и внутри неё — обычная структура репозитория: `module\`, `netbox-sync.py`,
`requirements.txt`, `README.md` и т.д. Это и есть та самая "папка, куда
клонировали" — именно **сюда**, поверх этих файлов, нужно скопировать
исправленные файлы из архива.

## Шаг 2. Распаковать архив с фиксами (если ещё не распакован)

Распакуйте `netbox-sync-fork-lense.zip` — внутри будет папка
`netbox-sync-fork`. Полный путь к ней у вас получится примерно такой:
```
D:\Работа\Platformix\demo\netbox_claude\netbox-sync-fork-lense\netbox-sync-fork\
```

## Шаг 3. Скопировать исправленные файлы — куда именно

Исправлено (и должно быть скопировано) только **6 файлов** — не вся папка
целиком. Остальное в архиве — это просто копия вашей текущей рабочей
установки 1.8.0 для полноты дерева, и накладывать её поверх свежего клона
не нужно: так вы бы случайно затёрли то, что уже актуальнее в самом
GitHub-репозитории.

Соответствие "откуда → куда" (пути откуда — из папки архива на шаге 2,
пути куда — из папки клона на шаге 1):

| Откуда (из архива) | Куда (в папку клона `netbox-sync-src\`) |
|---|---|
| `netbox-sync-fork\module\__init__.py` | `netbox-sync-src\module\__init__.py` |
| `netbox-sync-fork\requirements.txt` | `netbox-sync-src\requirements.txt` |
| `netbox-sync-fork\netbox-sync.py` | `netbox-sync-src\netbox-sync.py` |
| `netbox-sync-fork\module\netbox\connection.py` | `netbox-sync-src\module\netbox\connection.py` |
| `netbox-sync-fork\module\netbox\object_classes.py` | `netbox-sync-src\module\netbox\object_classes.py` |
| `netbox-sync-fork\module\sources\vmware\connection.py` | `netbox-sync-src\module\sources\vmware\connection.py` |

Проще всего сделать это одной командой (PowerShell, замените пути на свои,
если распаковали не в `netbox_claude`):

```powershell
$src = "D:\Работа\Platformix\demo\netbox_claude\netbox-sync-fork-lense\netbox-sync-fork"
$dst = "D:\Работа\Platformix\demo\netbox_claude\netbox-sync-src"

Copy-Item "$src\module\__init__.py"                         "$dst\module\__init__.py" -Force
Copy-Item "$src\requirements.txt"                            "$dst\requirements.txt" -Force
Copy-Item "$src\netbox-sync.py"                               "$dst\netbox-sync.py" -Force
Copy-Item "$src\module\netbox\connection.py"                  "$dst\module\netbox\connection.py" -Force
Copy-Item "$src\module\netbox\object_classes.py"              "$dst\module\netbox\object_classes.py" -Force
Copy-Item "$src\module\sources\vmware\connection.py"          "$dst\module\sources\vmware\connection.py" -Force
```

Также скопируйте для истории (не обязательно, но полезно — они не
конфликтуют ни с чем в репозитории):
```powershell
Copy-Item "$src\..\CHANGELOG_FORK.md" "$dst\CHANGELOG_FORK.md" -Force
Copy-Item "$src\..\DEPLOY_RU.md"      "$dst\DEPLOY_RU.md" -Force
```

## Шаг 4. Закоммитить и запушить в свой форк

```powershell
cd D:\Работа\Platformix\demo\netbox_claude\netbox-sync-src
git add -A
git commit -m "Lense: upgrade to v1.8.1 baseline + fix MAC/asset_tag/cable bugs"
git push -u origin lense-main
```

Ветка `lense-main` в `github.com/Soldier2003/netbox-sync` — это то, за чем
дальше следить и куда переносить будущие апстрим-фиксы (через
`git fetch upstream && git merge upstream/main`, если позже добавите
upstream как remote).

## Шаг 5. Развернуть на сервере NetBox

**Сейчас (пока NetBox всё ещё 4.2.6, до апгрейда ОС/NetBox):**

```bash
cd /opt
sudo mv netbox-sync netbox-sync.bak-$(date +%Y%m%d)   # бэкап текущей рабочей версии
sudo git clone -b lense-main https://github.com/Soldier2003/netbox-sync.git
cd netbox-sync
sudo cp ../netbox-sync.bak-*/settings.yaml .           # вернуть свои реальные секреты/настройки
python3 -m venv .venv
. .venv/bin/activate
pip3 install --upgrade pip
pip3 install -r requirements.txt
```

**Обязательно проверить перед боевым запуском:**

```bash
. .venv/bin/activate
python3 netbox-sync.py -n -l DEBUG2 -c settings.yaml
```
`-n` — dry-run, ничего не меняет в NetBox, только показывает, что было бы
сделано. Смотреть, что:
- пропали ошибки `Cannot unassign MAC Address...`
- пропали ошибки `device with this asset tag already exists` (для новых
  хостов; старые записи в NetBox нужно почистить вручную один раз — см.
  `CHANGELOG_FORK.md`, п. 2.2)
- если ошибка про кабель ещё встретится в dry-run — это нормально: сам
  self-heal (удаление кабеля) — не dry-run-safe операция и в `-n` режиме
  не выполняется (изменения в NetBox не пишутся). Проверить его нужно на
  обычном (не dry-run) прогоне на тестовом стенде.

Если всё чисто — заменить cron/systemd-задачу на новый путь и снять с
эксплуатации `netbox-sync.bak-*`.

## Шаг 6. После апгрейда NetBox → 4.6.4 и ОС → Ubuntu 24.04

Требований к изменению кода форка это не добавляет (см. `CHANGELOG_FORK.md`,
раздел 3 — совместимость проверена статически). Но обязательно:

1. Пересоздать venv под Python 3.12 (после перехода на Ubuntu 24.04):
   ```bash
   cd /opt/netbox-sync
   rm -rf .venv
   python3.12 -m venv .venv
   . .venv/bin/activate
   pip3 install -r requirements.txt
   ```
2. Прогнать ещё раз dry-run (`-n -l DEBUG2`) на тестовом стенде с уже
   обновлённым NetBox 4.6.x, прежде чем переносить в прод/демо — этого
   явно просил придерживаться план из прошлого чата
   (`00_ВЫВОДЫ_И_РЕКОМЕНДАЦИИ.md`, п. 4 рекомендуемой последовательности).
3. Проверить, не осталось ли ошибок `Cannot unassign MAC Address` — если
   NetBox 4.6.x уже содержит фикс netbox-community/netbox#18784, часть
   первопричины устранится и на стороне NetBox тоже; наш фикс (п. 2.1
   в `CHANGELOG_FORK.md`) остаётся корректным и нужным независимо от этого.

## Шаг 7. Разовая очистка данных в NetBox (не код, ручная операция)

Перед первым чистым прогоном рекомендуется вручную поправить в NetBox
(UI или API):

- **asset_tag = "Base Board Asset Tag"** на устройствах esxi05/esxi06
  (id 409, 411 по логам от 19.07.2026) — очистить поле, дальше синхронизация
  подхватит уже отфильтрованное состояние (patch 2.2 не даст записать его
  заново).
- **Кабели на интерфейсах** dcim/interfaces id 4483, 4490, 4491, 4492, 4494,
  4496, 4497 — можно оставить как есть: self-heal (patch 2.3) снимет их
  автоматически при первом же не-dry-run прогоне. Ручное вмешательство нужно
  только если хочется разобраться, почему они там оказались, до автоматической
  очистки.

## Что не входит в этот форк

- Замена netbox-sync на собственный скрипт (pyvmomi + pynetbox) — по вашему
  подтверждению это пока не требуется, vCenter-синхронизация остаётся через
  netbox-sync.
- Поддержка Proxmox VE — по прошлому плану для этого предполагается
  отдельный инструмент, netbox-proxbox (не входит в этот форк).
- Фиксы upstream v1.8.1, не относящиеся к вашей конфигурации: "wrong platform
  evaluation" (#448/#492/#495), check_redfish 2.0 (#478, источник не
  используется), конфликт зависимостей vsphere-automation-sdk (#481/#497,
  VMware tag sync не настроен в `settings.yaml`). Если что-то из этого
  понадобится позже — проще подтянуть напрямую `git merge upstream/main` в
  форк, чем переносить руками (эти файлы мы намеренно не трогали, поэтому
  merge пройдёт без конфликтов).
