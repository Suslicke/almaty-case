<div align="center">

# 🎰 Almaty Walk Case

**Рулетка в стиле CS:GO-кейса для выбора вечерней прогулки в Алматы.**
**42 предмета · 7 редкостей · история выпадений · исключение вариантов.**

[![Status](https://img.shields.io/badge/status-production-22c55e?style=for-the-badge)](#)
[![Cloudflare Workers](https://img.shields.io/badge/Cloudflare-Workers-f38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/)
[![Vanilla JS](https://img.shields.io/badge/vanilla-JS-f7df1e?style=for-the-badge&logo=javascript&logoColor=black)](#)
[![Items](https://img.shields.io/badge/items-42-blue?style=for-the-badge)]()
[![Rarities](https://img.shields.io/badge/rarities-7-purple?style=for-the-badge)]()

</div>

---

## 📚 Что это

Один HTML-файл, в котором крутится рулетка в стиле CS:GO-кейса и выдаёт идею для вечерней прогулки в Алматы. 40 обычных активностей + 2 редких дропа. Всё — чистый HTML + vanilla JS, без фреймворков, без бэкенда. Пользовательские данные (история выпадений, исключённые места) лежат в `localStorage` браузера.

<table>
<tr>
<td width="50%" valign="top">

### 🎯 Что внутри
- 42 места Алматы: парки, кафе, смотровые, кино, квесты
- 7 уровней редкости (Consumer → Rare, как в CS:GO)
- Настраиваемые веса выпадения через `const weights`
- Google Maps-ссылка у каждого варианта
- Кастомные советы («💡 tip») для каждого места

</td>
<td width="50%" valign="top">

### ⚡ Фичи
- 🎞️ Анимация прокрутки рулетки с остановкой на редкости
- 📜 История выпадений с форматом «Сегодня / Вчера / дата»
- 🚫 Исключение вариантов (не нравится → выкини из пула)
- 🔁 Восстановление исключённых одним кликом
- 📊 Google Analytics 4 с `anonymize_ip`
- 📱 Адаптивка под мобилу

</td>
</tr>
</table>

---

## 🖥️ Локальная разработка

```bash
git clone https://github.com/Suslicke/almaty-case.git
cd almaty-case
npm install
npm run dev
```

`wrangler dev` поднимает локальный воркер с живыми статикам из `./public` на `http://localhost:8787`.

Альтернатива без Node:

```bash
cd public
python3 -m http.server 8000
# открыть http://localhost:8000
```

### Деплой на Cloudflare Workers

В репозитории лежит `wrangler.jsonc` + нужная `devDependency`, поэтому деплой одной командой:

```bash
npm run deploy
```

Первый раз: `npx wrangler login` для авторизации.

По умолчанию воркер выкатится на `https://almaty-walk-case.<account>.workers.dev`. Свой домен подключается через **Cloudflare Dashboard → Workers → almaty-walk-case → Custom Domains**.

---

## 📊 Google Analytics 4

В `public/index.html` две строки нужно заменить на свой Measurement ID (формат `G-XXXXXXXXXX`):

```html
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  ...
  gtag('config', 'G-XXXXXXXXXX', { 'anonymize_ip': true });
</script>
```

**Как получить Measurement ID:**
1. [analytics.google.com](https://analytics.google.com) → создать аккаунт и property
2. Admin → Data Streams → Web → создать поток → скопировать `G-XXXXXXXXXX`

### Какие события трекаются

| Event | Когда | Параметры |
|---|---|---|
| `open_case` | нажал «Открыть кейс» | `active_items`, `total_items` |
| `case_result` | после крутки показан результат | `item_name`, `rarity`, `category`, `is_rare` |
| `map_click` | нажал «Открыть в картах» | `item_name`, `rarity` |
| `item_excluded` | исключил предмет | `item_name`, `active_items` |
| `item_restored` | вернул исключённый | `item_name`, `active_items` |
| `history_cleared` | очистил историю | `items_cleared` |

### Где смотреть

GA4 → **Reports → Realtime** (видно мгновенно) или **Reports → Engagement → Events** (агрегация 24ч).

Детальные отчёты «какое место чаще всего выпадает»:
**Explore → Free form → Dimension: `item_name` + Metric: `Event count` (filter: `event_name = case_result`)**

---

## ⚙️ Кастомизация

Все данные в массиве `const places = [...]` в конце `public/index.html`. Поля:

```js
{
  name: "Кок-Тобе",              // название
  cat: "Вид · канатка",          // категория (показывается в карточке)
  icon: "🌃",                    // эмодзи
  rarity: "industrial",          // consumer | industrial | milspec | restricted | classified | covert | rare
  desc: "Описание...",
  tip: "💡 Совет...",
  gmaps: "https://..."           // ссылка на Google Maps
}
```

Веса редкостей настраиваются в `const weights = {...}`. По умолчанию все категории равновероятны, кроме `rare` (0.3 от остальных).

---

## 🎨 Стилизация

Цвета — CSS-переменные в `:root`. Шрифты — Chakra Petch + Rajdhani (Google Fonts). Темы редкости переключаются через `--rarity` на каждой карточке.

---

## 🧱 Структура проекта

```
.
├── public/                # всё, что выкатывается в прод
│   ├── index.html         # основное приложение (HTML + CSS + JS в одном файле)
│   ├── 404.html           # страница 404 в стиле проекта
│   ├── favicon.svg        # иконка
│   ├── _headers           # заголовки (security + cache)
│   └── robots.txt         # разрешаем индексацию
├── wrangler.jsonc         # Cloudflare Workers + Static Assets
├── package.json
├── .gitignore
└── README.md
```

---

## 🔐 Приватность

- Прогресс (история выпадений, исключённые места) хранится только в `localStorage` браузера.
- GA4 работает с `anonymize_ip: true`.
- На сервер ничего не уходит — бэкенда нет.

---

<div align="center">

Создал **Andrei** · [@Suslicke](https://t.me/Suslicke)

_Лицензия: делай что хочешь, копируй, форкай._

</div>
