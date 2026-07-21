# Развёртывание форка Lense/netbox-sync — инструкция

## Обновление от 21.07.2026 — если вы уже запушили lense-main

Вы уже прошли шаги 0-4 и запушили `lense-main` с первым набором фиксов
(MAC/asset_tag/cable). С тех пор добавился ещё один перенесённый апстрим-фикс —
"wrong platform evaluation" (см. `CHANGELOG_FORK.md`, раздел 1). Он был
неправильно указан как "не перенесено" в первой версии инструкции — это
исправлено, и сам фикс теперь включён в архив. Чтобы применить:

1. Распакуйте обновлённый `netbox-sync-fork-lense.zip` (или скачайте заново).
2. Скопируйте только 2 файла (остальные 4 из предыдущего раза не менялись):
   ```powershell
   $src = "D:\Работа\Platformix\demo\netbox_claude\netbox-sync-fork-lense\netbox-sync-fork"
   $dst = "D:\Работа\Platformix\demo\netbox_claude\netbox-sync-src"

   Copy-Item "$src\module\sources\vmware\connection.py" "$dst\module\sources\vmware\connection.py" -Force
   Copy-Item "$src\CHANGELOG_FORK.md" "$dst\CHANGELOG_FORK.md" -Force
   Copy-Item "$src\DEPLOY_RU.md" "$dst\DEPLOY_RU.md" -Force
   ```
3. Закоммитьте и запушьте (git identity и токен уже настроены с прошлого раза):
   ```powershell
   cd D:\Работа\Platformix\demo\netbox_claude\netbox-sync-src
   git add -A
   git commit -m "Lense: port upstream fix for wrong platform evaluation (#448/#492/#495)"
   git push
   ```
4. На сервере — просто `git pull` в `/opt/netbox-sync` вместо полного передеплоя
   (venv/зависимости этот фикс не трогает):
   ```bash
   cd /opt/netbox-sync
   sudo git pull
   . .venv/bin/activate
   python3 netbox-sync.py -n -l DEBUG2 -c settings.yaml   # dry-run проверка
   ```

Дальше в документе — шаги 0-8 в исходном виде, для установки с нуля (например,
на другом компьютере) или как справочник.

---

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

**4.1. Если это первый git-коммит на этом компьютере вообще** — сначала
представьтесь git (делается один раз, дальше не потребуется):

```powershell
git config --global user.name "Squirrel"
git config --global user.email "dmitriy.kulikov.aion@gmail.com"
```

**4.2. Если ещё не настроена аутентификация для push на GitHub** — заранее
создайте Personal Access Token, он потребуется вместо пароля:

1. Откройте https://github.com/settings/tokens/new
2. Note: любое название, например `netbox-sync-lense`; Expiration: например
   90 дней; Scopes: поставьте галочку `repo`.
3. **Generate token** → скопируйте токен (`ghp_...`) — он показывается
   только один раз.

**4.3. Коммит и push:**

```powershell
cd D:\Работа\Platformix\demo\netbox_claude\netbox-sync-src
git add -A
git commit -m "Lense: upgrade to v1.8.1 baseline + fix MAC/asset_tag/cable bugs"
git push -u origin lense-main
```

Если git спросит `Username for 'https://github.com':` — введите `Soldier2003`.
На запрос пароля (`Password for ...`) вставьте **токен из шага 4.2**, не
пароль от аккаунта GitHub (обычные пароли для push уже не принимаются).
При вставке в консоли символы не отображаются — это нормально, просто
нажмите Enter. После первого успешного push Windows Credential Manager
запомнит токен, и в следующий раз запроса не будет — если только токен не
истечёт (см. Expiration в шаге 4.2, тогда нужно будет создать новый).

Ветка `lense-main` в `github.com/Soldier2003/netbox-sync` — это то, за чем
дальше следить и куда переносить будущие апстрим-фиксы (через
`git fetch upstream && git merge upstream/main`, если позже добавите
upstream как remote).

## Шаг 5. Развернуть на сервере NetBox — сейчас, на текущей Ubuntu 22.04 / Python 3.10

Это можно делать уже сегодня, до апгрейда ОС и NetBox — весь `requirements.txt`
форка (включая `aiodns==4.0.0`) официально поддерживает Python 3.10 и ставится
из готовых wheel-пакетов (проверено на PyPI, без компиляции из исходников —
подробности в `CHANGELOG_FORK.md`, раздел 1). Апгрейд Python в этом шаге не
требуется.

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
- platform виртуальных машин с Linux — не ошибка, а стоит просто посмотреть
  глазами: после фикса "wrong platform evaluation" значения могут обновиться
  (например, добавится версия дистрибутива) — это ожидаемо, не баг.

Если всё чисто — заменить cron/systemd-задачу на новый путь и снять с
эксплуатации `netbox-sync.bak-*`.

## Шаг 6. Апгрейд ОС (Ubuntu 22.04 → 24.04) и NetBox (4.2.6 → 4.6.4)

Это отдельная, более крупная работа, подробно расписанная в прошлом чате —
файлы `00_ВЫВОДЫ_И_РЕКОМЕНДАЦИИ.md`, `01_ДИАГНОСТИКА_СИСТЕМЫ.md`,
`02_ИСТОЧНИКИ.md` в этой же папке проекта. Здесь — только краткий чек-лист,
как эти два апгрейда стыкуются с форком netbox-sync (форк на этом шаге менять
не нужно, только его окружение):

1. Расширить диск / освободить место (на момент диагностики было
   свободно всего 2.8G — мало для второго venv и бэкапа PostgreSQL).
2. Обновить NetBox 4.2.6 → 4.4.x **на текущем Python 3.10** (4.4 ещё не
   требует Python 3.12) — прогнать форк в dry-run, убедиться что новых
   ошибок не появилось.
3. Обновить Ubuntu 22.04 → 24.04 LTS (`do-release-upgrade`) — даёт
   Python 3.12 "из коробки".
4. На тестовом стенде: пересобрать venv форка под Python 3.12 (шаг 7 ниже) и
   прогнать dry-run против тестового NetBox 4.5/4.6 — только после
   подтверждения стабильности переносить в прод/демо.
5. Обновить NetBox до 4.6.4 (или более новой стабильной 4.6.x).
6. Отдельно спланировать переход на PostgreSQL 15+ и Redis 6.2+/7.x — не
   блокирует 4.6, но понадобится до NetBox 4.7 (PG14/Redis 5.x будут
   deprecated).

## Шаг 7. После апгрейда NetBox → 4.6.4 и ОС → Ubuntu 24.04

Изменений в коде форка этот шаг не требует (совместимость с 4.3-4.6 проверена
статически — см. `CHANGELOG_FORK.md`, раздел 3), но обязательно:

1. Пересоздать venv под Python 3.12:
   ```bash
   cd /opt/netbox-sync
   rm -rf .venv
   python3.12 -m venv .venv
   . .venv/bin/activate
   pip3 install -r requirements.txt
   ```
2. Прогнать ещё раз dry-run (`-n -l DEBUG2`) на тестовом стенде с уже
   обновлённым NetBox 4.6.x, прежде чем переносить в прод/демо.
3. Проверить, не осталось ли ошибок `Cannot unassign MAC Address` — если
   NetBox 4.6.x уже содержит фикс netbox-community/netbox#18784, часть
   первопричины устранится и на стороне NetBox тоже; наш фикс (п. 2.1
   в `CHANGELOG_FORK.md`) остаётся корректным и нужным независимо от этого.

## Шаг 8. Разовая очистка данных в NetBox (не код, ручная операция)

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

## Шаг 9. Как поддерживать форк в актуальном состоянии (постоянный процесс)

netbox-sync больше не сопровождается официально в прежнем режиме, но
репозиторий bb-Ricardo/netbox-sync не заброшен полностью — релизы
(например v1.8.1, 18.03.2026) продолжают выходить. Чтобы форк не отставал:

**9.1. Один раз — подключить upstream как второй remote:**
```bash
cd /opt/netbox-sync   # или там, где лежит ваш локальный клон lense-main
git remote add upstream https://github.com/bb-Ricardo/netbox-sync.git
git remote -v   # проверить: origin = ваш форк, upstream = bb-Ricardo/netbox-sync
```

**9.2. Периодически — проверять, что нового вышло:**
```bash
git fetch upstream
git log lense-main..upstream/main --oneline   # список новых коммитов upstream
git tag -l | sort -V | tail -5                # локальные теги (после fetch --tags)
```
Или просто открывать https://github.com/bb-Ricardo/netbox-sync/releases —
там видно номер версии и changelog каждого релиза.

**9.3. При появлении нового релиза — что делать:**
1. Прочитать changelog релиза — понять, что изменилось и не пересекается ли
   это с уже перенесёнными фиксами (список — `CHANGELOG_FORK.md`, раздел 2).
2. Если правки в тех же файлах, что и патчи форка (сейчас это `netbox-sync.py`,
   `module/__init__.py`, `module/netbox/connection.py`,
   `module/netbox/object_classes.py`, `module/sources/vmware/connection.py`,
   `requirements.txt`) — либо слить вручную (`git merge upstream/vX.Y.Z` и
   разрешить конфликты, сверяясь с `CHANGELOG_FORK.md`), либо прислать мне
   новую версию upstream и попросить пересобрать патчи поверх неё — это
   быстрее и надёжнее, чем разбирать конфликты вручную.
3. Обновить `module/__init__.py` (`__version__`, `__version_date__`) и
   дописать соответствующую запись в `CHANGELOG_FORK.md`.
4. Обязательно прогнать dry-run (`-n -l DEBUG2`) на тестовом стенде перед
   заменой рабочей версии — как в Шаге 5.

**9.4. Если нужно, чтобы я регулярно проверял за вас:**
Могу настроить периодическую задачу (например, раз в 2 недели), которая
будет проверять https://github.com/bb-Ricardo/netbox-sync/releases на новые
теги и присылать вам сводку — что вышло и стоит ли переносить в форк. Скажите
слово — и я это настрою (нужно будет один раз подтвердить расписание).

## Что не входит в этот форк

- Замена netbox-sync на собственный скрипт (pyvmomi + pynetbox) — по вашему
  подтверждению это пока не требуется, vCenter-синхронизация остаётся через
  netbox-sync.
- Поддержка Proxmox VE — по прошлому плану для этого предполагается
  отдельный инструмент, netbox-proxbox (не входит в этот форк).
- Фиксы upstream v1.8.1, не относящиеся к вашей конфигурации: check_redfish 2.0
  (#478, источник не используется), конфликт зависимостей
  vsphere-automation-sdk (#481/#497, VMware tag sync не настроен в
  `settings.yaml`). Если что-то из этого понадобится позже — проще подтянуть
  напрямую `git merge upstream/main` в форк, чем переносить руками (эти файлы
  мы намеренно не трогали, поэтому merge пройдёт без конфликтов).
  Фикс "wrong platform evaluation" (#448/#492/#495), в отличие от этих двух,
  **уже перенесён** в форк 21.07.2026 — он относится к вашей среде (VMware
  vCenter с разными гостевыми ОС), см. `CHANGELOG_FORK.md`, раздел 2, пункт 5.
