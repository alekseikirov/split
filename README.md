# Делим счёт: как запустить

Калькулятор для деления счёта в ресторане: по позициям, с чаевыми, сдачей и сканированием чека.
Сайт лежит на GitHub Pages, а сканирование идёт через посредника на Railway или Cloudflare. Ключ Claude API хранится только в переменных посредника.

## Что в архиве

| Файл | Куда |
|---|---|
| `index.html`, `manifest.webmanifest`, `sw.js`, `icon-180.png`, `icon-512.png`, `icon-512-maskable.png` | в репозиторий GitHub |
| папка `railway/` (`server.js`, `worker.js`, `package.json`) | в Railway: отдельный приватный репозиторий |
| `worker.js` | вместо Railway, если выберешь Cloudflare |

Посредник нужен один: **Railway (шаг 1А)** или **Cloudflare (шаг 1Б)**.

## Шаг 1А. Railway (10 минут)

1. Создать на GitHub **приватный** репозиторий `split-scanner` и загрузить в него три файла из папки `railway/`.
2. В Railway: **New Project → Deploy from GitHub repo** → выбрать `split-scanner`. Railway сам увидит `package.json` и запустит `npm start`.
3. В сервисе открыть **Variables** и добавить:
   - `ANTHROPIC_API_KEY` — ключ из Claude Platform;
   - `PIN` — любой код из 4–6 цифр;
   - `ALLOWED_ORIGIN` — `https://alekseikirov.github.io`.
4. **Settings → Networking → Generate Domain**. Получится адрес вида `https://split-scanner-production.up.railway.app`. Если открыть его в браузере, должно показать `ok`.
5. Этот адрес и есть `WORKER_URL` для шага 2.

## Шаг 1Б. Cloudflare Worker (вместо Railway)

1. Зарегистрироваться на dash.cloudflare.com (бесплатный тариф: 100 000 запросов в день).
2. **Workers & Pages → Create → Create Worker**, назвать `split-scanner`, нажать **Deploy**.
3. **Edit code**: удалить всё, вставить содержимое `worker.js`, нажать **Deploy**.
4. **Settings → Variables and Secrets → Add** и добавить три переменные:
   - `ANTHROPIC_API_KEY`: тип **Secret**, значение — ключ из Claude Platform;
   - `PIN`: тип **Secret**, любой код из 4–6 цифр (его будешь вводить на телефоне);
   - `ALLOWED_ORIGIN`: тип **Text**, значение `https://alekseikirov.github.io`.
5. Скопировать адрес Worker'а вида `https://split-scanner.<твой-аккаунт>.workers.dev`.

## Шаг 2. Вписать адрес Worker'а в index.html

Найти в `index.html` строку

```js
const WORKER_URL=''; // ← ВСТАВЬТЕ адрес Worker'а
```

и вставить адрес между кавычками. Ключ API сюда не вставлять: файл публичный.

## Шаг 3. GitHub Pages

1. Создать публичный репозиторий `split` в аккаунте `alekseikirov`.
2. Загрузить файлы сайта из таблицы выше (Add file → Upload files).
3. **Settings → Pages → Build and deployment**: Source — *Deploy from a branch*, Branch — `main` / `(root)`, нажать Save.
4. Через 1–2 минуты сайт будет доступен по адресу `https://alekseikirov.github.io/split/`.

## Шаг 4. Установить на iPhone

1. Открыть ссылку в **Safari**.
2. Нажать «Поделиться» → **«На экран Домой»**.
3. Открыть приложение с иконки, нажать «Сфотографировать чек» и один раз ввести PIN.

После первого запуска калькулятор открывается и без интернета. Без интернета не работает только сканирование.

## Как обновлять

После любого изменения `index.html` откройте `sw.js` и увеличьте версию: `split-v1` → `split-v2`. Иначе телефоны могут показывать старую версию из кэша.

## Безопасность и расходы

- Ключ API хранится только у посредника (Railway или Cloudflare). Если ключ всё же утечёт, удалите его в Claude Platform и создайте новый.
- Посредник принимает запросы только с твоего сайта и только с правильным PIN. Чтобы сменить PIN, поменяй его в переменных посредника.
- Поставь месячный лимит расходов в Claude Platform, например $5. Одно сканирование стоит меньше цента.
- По умолчанию используется модель `claude-sonnet-5-5`. Для экономии можно добавить переменную `MODEL` со значением `claude-haiku-5-5`.

## Если не работает

- **«Неверный PIN»**: проверь переменную `PIN` в Worker'е.
- **«запрос не с разрешённого сайта»**: в `ALLOWED_ORIGIN` должно быть ровно `https://alekseikirov.github.io`, без `/split`.
- **«сервис распознавания ответил 401»**: неверный или истёкший ключ API.
- **«…ответил 400»** или **«…404»**: проверь название модели в переменной `MODEL`.
- **Кнопки сканирования нет**: в `index.html` не заполнен `WORKER_URL`.
- Подробные ошибки видны в логах: в Cloudflare — Worker → **Logs**, в Railway — сервис → **Deployments → View logs**.
