# Подача плагинов в каталог Anthropic

Памятка владельцу: что вписать в портал [claude.ai/directory/manage](https://claude.ai/directory/manage)
по каждому плагину и что из требований каталога уже выполнено. Сверено с документацией
на 2026-10-12: [Submit your plugin](https://claude.com/docs/plugins/submit),
[Plugin pre-submission checklist](https://claude.com/docs/plugins/pre-submission-checklist),
[Platform support](https://claude.com/docs/plugins/platform-support),
[Plugin manifest reference](https://code.claude.com/docs/en/plugins-reference).

## Главное до начала

1. **Репозиторий приватный — опубликовать листинг нельзя.** Портал разрешает Validate и
   Submit for review из приватного репо, но для **Publish** репо должен стать публичным
   («Publishing needs a public repository»). Решение владельца 12.10 — оставить private;
   значит, можно пройти проверку и ревью, а в каталоге плагины появятся только после
   открытия `stufently/claude-plugins`. Для `telegram-ai-cli-mcp` вопроса нет: его репо уже
   публичный.
2. **Для приватного репо** в портале нужно: подключить GitHub-аккаунт с правом push в
   claude.ai (в той организации, откуда подаёте), установить Claude GitHub App на
   `stufently/claude-plugins` и согласиться на выгрузку исходников («This repository is
   private.»). Без этого Validate отвечает только «Couldn't find that repository…».
3. **Одна подача = одна папка плагина.** Здесь шесть подач: пять папок `plugins/<имя>` из
   этого репо и `telegram-ai-cli-mcp` из его собственного репо. Лимит — 10 подач на
   организацию за 24 часа (черновики и отозванные тоже считаются).
4. **Нужен платный план** (Pro/Max — со своего аккаунта; Team/Enterprise — Owner). Подавать
   из той организации, которой листинг должен принадлежать навсегда: вторая организация
   ту же папку подать не сможет.
5. Каждый релиз поднимать `version` в `plugin.json` — иначе обновление не дойдёт.

## Общие поля для всех пяти плагинов из этого репо

| Шаг портала | Что вписать |
| --- | --- |
| What would you like to submit? | **Plugin bundle** |
| Repository | `stufently/claude-plugins` |
| Plugin path | `plugins/<имя плагина>` (точно с таким регистром) |
| Branch or tag | `main` (или пусто — ветка по умолчанию) |
| Listing details | читается из `plugin.json` и `README.md` папки; править там, не в портале |
| How new versions reach the directory | **GitHub push webhook** (нужны admin-права на репо) |
| Auto-publish passing versions | на усмотрение; по умолчанию каждую версию публикует ревьюер |
| Compliance | контактный email, все четыре галочки |

### Data handling — ответы

| Вопрос | Ответ | Почему |
| --- | --- | --- |
| Reads or stores personal data? | Читает данные из аккаунта пользователя в сервисе (метрики, мониторинг, тарифы; у Telegram — переписку) только по запросу. Docker-плагины ничего не хранят; **Bing** хранит локально планы записи (с аргументами, в т.ч. email/коды делегирования) и журнал аудита; Telegram — сессию и аудит `tg-ai` | Секреты хранит Claude Code в системном хранилище; zabbix/google/yandex/aviasales — `docker run --rm` без состояния |
| Sends data to services other than declared connectors? | **Нет** — только в API сервиса, указанного ниже по каждому плагину, плюс скачивание образа/пакета (ghcr.io / PyPI) | Хуков и телеметрии в плагинах нет |
| How long it keeps data | Docker-плагины — не хранят. Bing — `~/.local/state/bing-webmaster-ai-cli-mcp/` на машине пользователя, без автоудаления, пока пользователь не удалит. Telegram — см. его раздел | Удалённо ничего не хранится |
| Intended for people under 18? | Нет | |

## По плагинам

### 1. `zabbix-ai-cli-mcp`

| Поле | Значение |
| --- | --- |
| Plugin path | `plugins/zabbix-ai-cli-mcp` |
| name / displayName | `zabbix-ai-cli-mcp` / Zabbix AI CLI |
| Описание (из `plugin.json`) | Ask Zabbix what is broken and why an alert fired or never arrived. Read-only by default: the bundled server starts with writes switched off. |
| Запускает | `docker run -i --rm … ghcr.io/stufently/zabbix-ai-cli-mcp:0.4.3@sha256:58c24a47…ef887 mcp` |
| Куда шлёт данные | только на Zabbix URL пользователя (JSON-RPC API) |
| userConfig | `zabbix_url` (строка, обязательна), `zabbix_token` (**sensitive**, обязателен) |
| Безопасный дефолт | `ZABBIX_AI_CLI_MCP_ALLOW_WRITE=false` зашит в плагин |

### 2. `google-webtools-mcp`

| Поле | Значение |
| --- | --- |
| Plugin path | `plugins/google-webtools-mcp` |
| name / displayName | `google-webtools-mcp` / Google Webtools |
| Описание | Check why a page is not in Google, and get a weekly Search Console review, through the google-webtools MCP server. |
| Запускает | `docker run -i --rm -v <key file>:/creds/sa.json:ro … ghcr.io/stufently/google-webtools-mcp:1.2.1@sha256:8c92263d…19d19b` |
| Куда шлёт данные | Google Search Console, Analytics (GA4), Site Verification API (`*.googleapis.com`) |
| userConfig | `credentials_file` (тип `file`, обязателен; не sensitive — это путь, сам ключ остаётся в файле пользователя) |
| Безопасный дефолт | переключателя нет; README требует Restricted в GSC и Viewer в GA4; сценарии не зовут write-тулы |

### 3. `bing-webmaster-ai-cli-mcp`

| Поле | Значение |
| --- | --- |
| Plugin path | `plugins/bing-webmaster-ai-cli-mcp` |
| name / displayName | `bing-webmaster-ai-cli-mcp` / Bing Webmaster AI CLI |
| Описание | Check whether Bing has crawled and indexed a page and review Bing search traffic. Writes are only planned for a person to apply: the bundled server starts with BING_WM_ALLOW_WRITES=false. |
| Запускает | `uvx --from bing-webmaster-ai-cli-mcp==0.1.3 bing-webmaster-ai-cli-mcp` (PyPI) |
| Куда шлёт данные | Bing Webmaster API (`ssl.bing.com/webmaster/api.svc`), IndexNow (`api.indexnow.org`) |
| userConfig | `api_key` (**sensitive**, обязателен) |
| Безопасный дефолт | `BING_WM_ALLOW_WRITES=false` явно (с 0.1.3 это и дефолт сервера) |
| Хранит локально | планы и аудит в `~/.local/state/bing-webmaster-ai-cli-mcp/`; применение плана — человек в терминале: `uvx --from bing-webmaster-ai-cli-mcp==0.1.3 bing-wm plan apply PLAN_ID` |

### 4. `yandex-metrica-mcp`

| Поле | Значение |
| --- | --- |
| Plugin path | `plugins/yandex-metrica-mcp` |
| name / displayName | `yandex-metrica-mcp` / Yandex Metrica MCP |
| Описание | Traffic and goal-conversion reports from Yandex Metrica. Give it a token with the metrika:read scope only and the counter create/delete tools the server also exposes have nothing to act with. |
| Запускает | `docker run -i --rm … ghcr.io/stufently/yandex-metrica-mcp:0.2.2@sha256:251afcb1…ed74d403` |
| Куда шлёт данные | Yandex Metrica API (`api-metrika.yandex.net`) |
| userConfig | `oauth_token` (**sensitive**, обязателен) |
| Безопасный дефолт | переключателя нет; README требует токен только со scope `metrika:read` |

### 5. `aviasales-mcp`

| Поле | Значение |
| --- | --- |
| Plugin path | `plugins/aviasales-mcp` |
| name / displayName | `aviasales-mcp` / Aviasales MCP |
| Описание | Find the cheapest flight for a route or a budget from Aviasales (Travelpayouts) cached fares. Read-only: the server has no tool that books or pays. |
| Запускает | `docker run -i --rm … ghcr.io/stufently/aviasales-mcp:0.5.2@sha256:84a92124…e90f297` |
| Куда шлёт данные | Travelpayouts data API (`api.travelpayouts.com`); ссылки на бронь ведут на aviasales.ru |
| userConfig | `api_token` (**sensitive**, обязателен), `market` (строка, по умолчанию `ru`) |
| Безопасный дефолт | API только читает |

Почему образ, а не `uvx` (пакет 0.5.2 есть и на PyPI): сервер через `pydantic-settings`
читает `.env` из текущего каталога и переменные `PORT`/`MCP_PORT`/`MCP_HOST`/`MCP_AUTH_*`
из окружения. Под `uvx` это каталог проекта пользователя и его shell: чужой `PORT` в `.env`
переключит сервер на HTTP, и Claude Code не подключится, а чтение `.env` проекта сканер
каталога может принять за «Uses a credential from the user's machine». Контейнер видит
только два переданных значения. Образ запинен по digest — зависимости заморожены,
сборка воспроизводима; исходник и Dockerfile ревьюер читает в публичном репо сервера.

### 6. `telegram-ai-cli-mcp` (отдельный репо)

| Поле | Значение |
| --- | --- |
| Repository | `stufently/telegram-ai-cli-mcp` (публичный) |
| Plugin path | пусто — плагин в корне репо |
| Branch or tag | `main` (сейчас `e55c1e3b`, = тег `v0.1.2`) |
| name / version | `telegram-ai-cli-mcp` / 0.1.2 |
| Запускает | `tg-ai mcp` с `TGAI_PROFILE=readonly` — бинарь `tg-ai` пользователь ставит сам, в плагине его нет |
| Куда шлёт данные | Telegram MTProto (серверы Telegram) от имени аккаунта пользователя |
| userConfig | нет: вход через `tg-ai account login`, сессия хранится локально у `tg-ai` |
| Data handling | читает переписку пользователя — отвечать «да, читает персональные данные»; хранит сессию и журнал аудита `tg-ai` на машине пользователя |

Папкой плагина здесь будет весь репозиторий (180 файлов, все меньше 256 KiB, только текст),
поэтому сканер прочтёт и тесты, и Dockerfile, и `.env.example`. В этом репо
(`stufently/claude-plugins`) запись telegram — только ссылка из `marketplace.json`.

## Чеклист требований каталога

Статус для пяти плагинов из этого репо; отличия telegram — в последнем столбце.

| Требование (pre-submission checklist) | Результат при нарушении | Статус | Telegram |
| --- | --- | --- | --- |
| В папке есть `.claude-plugin/plugin.json` | Blocks | выполнено | выполнено |
| Подавать по одному плагину из marketplace-репо | Blocks | шесть отдельных подач (см. выше) | — |
| Все файлы, нужные MCP-серверу, внутри папки плагина | Blocks | выполнено: внешних файлов нет, сервер — образ/пакет | `tg-ai` вне плагина (ставится пользователем) |
| Без symlink, submodule, LFS, `.DS_Store` | Blocks | выполнено | выполнено |
| Имена файлов/папок валидны для Windows/macOS, в пути только `[A-Za-z0-9._-]` | Validation stops | выполнено | выполнено |
| Нет `export-ignore`/`filter` в `.gitattributes` | Validation stops | выполнено (файла нет) | выполнено |
| Размер репо < 50 MiB, файлы < 5 MiB | Validation stops | выполнено | выполнено |
| `name` — kebab-case, ASCII, ≤ 64 символа | Blocks | выполнено | выполнено |
| Имя не зарезервировано и не занято другой организацией | Blocks | не проверить локально — покажет Validate | то же |
| Имя/`displayName` не похоже на чужой бренд | Policy hold | **ожидаемо hold**: Zabbix, Google, Bing, Yandex, Aviasales, Telegram — чужие бренды; в README каждого плагина оговорка «community, not endorsed». Переименование `name` — решение владельца (ломает установленные копии) | ожидаемо hold |
| `description`, `author`, `version` заданы | Warning | выполнено (`version` 0.2.0) | выполнено |
| README ≥ 40 слов вне блоков кода в папке плагина | Blocks | выполнено (270–320 слов, самодостаточные) | выполнено |
| LICENSE в папке или `license` в `plugin.json` | Blocks | выполнено: и `license: MIT`, и файл `LICENSE` | выполнено |
| Файлы < 256 KiB, ≤ 512 файлов, только текст/картинки | Policy hold | выполнено | выполнено |
| MCP-сервер через `command`+`args`, не `.mcpb` | Hold/Blocks | выполнено | выполнено (папка `mcpb/` в репо — исходники сборки, не бандл; проверить отчёт Validate) |
| Пакет у лаунчера (`uvx`) запинен точной версией | Blocks | выполнено: bing `==0.1.3`; docker-образы — tag@digest | — |
| Нет `.npmrc`/`uv.toml`/своего индекса пакетов | Blocks | выполнено | выполнено |
| Секреты не в файлах; через `userConfig` с `sensitive: true` и `${user_config.KEY}` | Blocks | **выполнено** (раньше брались из окружения) | секретов в конфиге нет |
| Не читать креды из окружения пользователя (`$TOKEN`) | Policy hold | **выполнено**: в `.mcp.json` только `${user_config.*}`; `docker -e NAME` берёт значение из `env` сервера, которое задаёт Claude Code | выполнено |
| `.mcp.json` валиден, сервер соответствует схеме | Blocks | выполнено (`claude plugin validate --strict`) | выполнено |
| Удалённые серверы только `https` | Blocks | неприменимо (все stdio) | неприменимо |
| Локальный сервер — файл из плагина, без shell/`-c` | Policy hold | **ожидаемо hold** «MCP server command wasn't read»: команда — `docker`/`uvx`, не файл плагина | ожидаемо hold (`tg-ai`) |
| Пинованный `uvx`-пакет всегда смотрит ревьюер | Policy hold | ожидаемо hold для bing («Runs a pinned npx or uvx package») | — |
| Хуки/скилы/агенты: валидный frontmatter, `description` — строка | Blocks | выполнено (только скилы) | выполнено |
| README описывает всё, что плагин запускает и куда шлёт | Security scan | выполнено: раздел «What it runs and where data goes» | выполнено |
| Читаемый исходник, без минификации | Security scan | выполнено (плагин — JSON и Markdown; исходники серверов публичны) | выполнено |
| `claude plugin eval` — сравнение с плагином и без | рекомендация | **не сделано**: eval-наборов (`evals/`) нет; команда есть в Claude Code 2.1.296 | не сделано |
| Репо публичный к моменту публикации | Publish blocked | **нет — репо private по решению владельца** | выполнено |

## Где плагин работает

По [таблице платформ](https://claude.com/docs/plugins/platform-support): локальные
MCP-серверы в чате claude.ai игнорируются; в Cowork сервер с `${user_config.*}` без
`default` игнорируется (Cowork не спрашивает значения). Итого полноценно — **Claude Code**;
в чате и Cowork загрузятся только скилы, без инструментов. Портал покажет то же перед
отправкой.

## Проверено локально (2026-10-12, Claude Code 2.1.296)

- `claude plugin validate --strict` — пройден для маркетплейса, пяти папок и клона
  `telegram-ai-cli-mcp@main`. Контрольная порча (`${user_config.<необъявленный>}`)
  валидатором ловится.
- Установка в изолированный `CLAUDE_CONFIG_DIR` (в `/var/tmp`): все шесть плагинов
  ставятся; фиктивные значения через `claude plugin configure --values-stdin`; sensitive
  легли в `.credentials.json`, несекретные — в `pluginConfigs` `settings.json`;
  `claude mcp list` — все шесть серверов **Connected**.
- Пробный плагин подтвердил, что sensitive-значение и `default` доходят до окружения
  процесса MCP-сервера.
