# SkillTrendium

**AI-помощник для анализа трендов навыков на рынке труда.**

Система автоматически собирает свежие вакансии с HeadHunter, извлекает требования к навыкам с помощью LLM, ищет актуальные тренды в интернете через Tavily и формирует аналитический отчёт с графиками, рейтингом технологий и текстовым анализом трендов. Готовый отчёт можно скачать прямо из формы или получить на почту.

---

## Кому это полезно

| Аудитория | Проблема | Что даёт SkillTrendium |
|-----------|---------|------------------------|
| **Соискатели** | 4-6 часов на ручной анализ рынка при смене направления | Полная картина за 20-30 минут: что учить, какие технологии растут, чего ожидают работодатели |
| **HR-команды** | 6-12 часов работы аналитика на один обзор рынка, 50-200 вакансий вручную | Автоматический отчёт с рейтингом технологий, распределением по опыту и зарплатам |
| **Рекрутинговые агентства** | Решения на основе неполных или устаревших данных | Данные из сотен вакансий + веб-тренды, сведённые в один документ с графиками |
| **Образовательные платформы** | Программы обучения отстают от рынка | Актуальный срез: какие технологии реально требуются прямо сейчас |
| **Карьерные консультанты** | Рекомендации «на глаз» без статистики | Конкретные цифры: % вакансий по каждой технологии, тренды, blind spots |

> **Экономика:** Ручной анализ обходится HR-команде в 18 000 -- 144 000 руб./мес. SkillTrendium сводит это к стоимости GPU-инференса (~5 000 -- 15 000 руб./мес.) с окупаемостью за 1-2 месяца.

---

## Почему это крутое решение

- **End-to-end автоматизация** -- от ввода названия вакансии до готового отчёта с графиками. Ни одного ручного шага.
- **Кросс-анализ двух источников** -- данные из реальных вакансий HeadHunter пересекаются с веб-трендами (Tavily), что позволяет находить будущие тренды, которых ещё нет в вакансиях.
- **Масштаб:** до 2 000 вакансий за один запуск, до 20 страниц результатов HeadHunter.
- **Полностью open-source стек** -- Gemma 3 27B на vLLM, n8n self-hosted, HH API без аутентификации. Никаких vendor lock-in.
- **Два канала доставки** -- скачивание из формы или автоматическая отправка на email (обработка в фоне, мгновенный ответ пользователю).
- **Retry-механизм** -- вакансии, для которых LLM не извлекла технологии с первого раза, автоматически отправляются на повторную обработку меньшими батчами.
- **Продуктовый UI** -- кастомная тёмная тема формы с glassmorphism, анимации, адаптивная вёрстка отчёта, поддержка печати (Ctrl+P переключает на светлую тему).

---

## Демо-отчёты

Примеры реальных отчётов, сгенерированных SkillTrendium (открываются в браузере):

| Запрос | Отчёт |
|--------|-------|
| ML Engineer | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_ML_Engineer.html) |
| ML | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_ML.html) |
| AI-архитектор | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_AIархитектор.html) |
| Менеджер проекта | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_Менеджер_проекта.html) |
| 1С | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_1с.html) |
| LLM-разработчик | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_LLM_разработчик.html) |

---

## Как это работает

Пайплайн реализован в **n8n** и состоит из 10 этапов:

```
Форма → Email-роутинг → Ключевые слова (LLM) → Поиск вакансий (HH API) →
→ Описания → Технологии (LLM, батчи) → Retry → AI-анализ трендов (LLM) →
→ Веб-анализ (Tavily + LLM) → Отчёт + Доставка
```

### Подробный алгоритм

**1. Ввод запроса (форма)**
Пользователь заполняет форму: название вакансии, период анализа (до 30 дней) и опционально — email для доставки. Форма стилизована кастомным CSS (тёмная тема, glassmorphism, Inter).

**2. Email-роутинг**
Если пользователь выбрал отправку на email, система мгновенно отвечает «Запрос принят!» и продолжает обработку в фоне. Если нет — пользователь ждёт завершения и скачивает отчёт прямо из формы.

**3. Генерация ключевых слов (LLM)**
Модель получает название вакансии и генерирует до 20 поисковых ключевых слов на русском и английском, разделённых оператором `OR`. Это расширяет охват поиска — вместо одного запроса система ищет по всем вариациям.

**4. Поиск вакансий через HH API**
Запрос к [API HeadHunter](https://api.hh.ru/vacancies) с параметрами:
- `text` — ключевые слова из шага 3
- `search_field=name` — поиск по названию вакансии
- `per_page=100` — по 100 вакансий на страницу
- `period` — количество дней (задано пользователем)

Система обходит до **20 страниц** результатов (до **2 000 вакансий**), дедуплицирует по ID.

**5. Получение детальных описаний**
Для каждой найденной вакансии выполняется запрос `GET /vacancies/{id}` к HH API (батчами по 10 параллельных запросов). Из ответа извлекается полное описание (`description`), ключевые навыки (`key_skills`), профессиональные роли, зарплата, опыт, город. HTML-теги очищаются скриптом.

**6. Извлечение технологий (LLM, батчами по 50)**
Очищенные вакансии группируются в **батчи по 50 штук**. Каждый батч отправляется в LLM с промптом на извлечение технологий. Модель возвращает JSON с нормализованными названиями (например, `JS → JavaScript`, `K8s → Kubernetes`). HTTP-запросы к LLM отправляются батчами по 10 параллельно.

**7. Retry-механизм**
После первичного извлечения система проверяет, для каких вакансий технологии не были извлечены. Такие вакансии автоматически отправляются на повторную обработку **батчами по 10** с интервалом 2 секунды между запросами. Результаты мержатся с основными данными.

**8. AI-анализ трендов (LLM)**
Агрегированные данные по технологиям отправляются в LLM для текстового анализа. Модель получает топ-50 технологий с частотами, распределение по опыту и локациям. Анализ охватывает:
- **Неожиданные и растущие тренды** — что нового и необычного?
- **Ожидаемый стек** — что предсказуемо для этой роли?
- **Набирающие популярность** — что растёт в спросе?
- **Blind spots** — чего не хватает в данных?
- **Рекомендации** — что изучать в первую очередь?

**9. Веб-анализ трендов (Tavily + LLM)**
На основе названия вакансии и топ-5 технологий формируется поисковый запрос для [Tavily Search API](https://tavily.com). Результаты веб-поиска анализируются LLM в кросс-сопоставлении с данными из вакансий:
- **Кратко по веб-источникам** — главное из каждого результата поиска
- **Совпадения** — что подтверждается нашими данными
- **Новое из веба** — тренды, которых ещё нет в вакансиях (будущие тренды!)
- **Главные находки и неожиданности** — самое важное из обоих источников
- **Итоговые рекомендации** — конкретные действия

**10. Формирование отчёта и доставка**
Результаты агрегируются в два файла:

**HTML-отчёт** (основной) — содержит:

| Секция | Содержание |
|--------|-----------|
| **Сводная статистика** | Карточки: вакансий, технологий, городов, веб-источников |
| **Топ-25 технологий** | Горизонтальная столбчатая диаграмма (SVG) |
| **Распределение по опыту** | Кольцевая диаграмма (SVG, donut chart) |
| **Зарплатные диапазоны** | Кольцевая диаграмма по диапазонам (если данные есть) |
| **Топ-15 локаций** | Горизонтальная столбчатая диаграмма (SVG) |
| **AI-анализ трендов** | Текстовый анализ LLM по данным вакансий |
| **Источники Tavily** | Список веб-источников с описаниями |
| **Сводный анализ** | Объединённый анализ: вакансии + веб-тренды |
| **Таблица технологий** | Полный рейтинг с прогресс-барами |
| **Список вакансий** | Вакансии с тегами технологий (до 200) |

**Excel XML** (дополнительный экспорт, встроен в HTML кнопкой) — 6 листов: Technologies, Vacancies, By Experience, By Location, Trend Analysis, Summary.

Доставка: скачивание из формы **или** отправка на email (HTML + Excel вложениями).

Графики — **инлайн SVG**, без внешних зависимостей. Тёмная тема, glassmorphism, адаптивная вёрстка, поддержка печати (`Ctrl+P` → светлая тема).

---

## Бизнес-логика: группировка и агрегация данных

### Как формируются батчи

1. **Получение вакансий:** HH API возвращает до 100 вакансий на страницу. Система запрашивает до 20 страниц, дедуплицирует по ID — на выходе до 2 000 уникальных вакансий.

2. **Детальные описания:** Каждая вакансия обогащается полным описанием через `GET /vacancies/{id}`. Запросы идут батчами по 10 параллельных. Если детали недоступны — используются snippet из поисковой выдачи (fallback).

3. **Группировка для LLM:** Вакансии разбиваются на батчи по 50. В каждый батч попадают ID, название и описание (до 1 500 символов) каждой вакансии. LLM получает все 50 вакансий в одном промпте и возвращает JSON с технологиями для каждого ID.

4. **Retry:** Вакансии с пустым списком технологий и описанием > 30 символов отправляются повторно батчами по 10 (с полным описанием до 2 000 символов).

### Как считается статистика

- **Частота технологии** = количество вакансий, в которых она упомянута (не количество упоминаний)
- **% от вакансий** = частота / общее число вакансий × 100
- **Распределение по опыту** — из поля `experience` каждой вакансии (Нет опыта / 1-3 года / 3-6 лет / 6+ лет)
- **Зарплатные диапазоны** — среднее от `salary_from` и `salary_to`, разбивка по бакетам: < 50K, 50-100K, 100-150K, 150-200K, 200-300K, 300K+
- **Локации** — из поля `area.name`, топ-15 городов

---

## LLM-промпты

В пайплайне 4 типа вызовов к LLM (+ retry). Ниже — полные промпты для каждого.

### 1. Генерация ключевых слов

**System:**
```
You are a job search keyword specialist. Return ONLY search keywords separated
by OR. No other text, no quotes, no numbering.
```

**User:**
```
You are an expert recruiter and job search specialist. For the job title
"{vacancy_name}", generate a list of up to 20 relevant search keywords or
phrases (in Russian AND English) for finding related job postings on
HeadHunter (hh.ru).

STRICT RULES:
1) Every keyword MUST be directly related to this specific job role.
2) Do NOT include generic words like "разработчик", "developer", "engineer",
   "программист" without a qualifier — they are too broad and return
   irrelevant results.
3) Do NOT include technologies or tools that are only loosely associated —
   only include what is commonly required for this exact role.
4) Prefer specific role titles and their direct synonyms over generic terms.
5) Quality over quantity — fewer precise keywords are better than many
   vague ones.

Return ONLY the keywords separated by the word OR. No quotes, no extra text,
no numbering, no explanation.
```

**Параметры:** `temperature: 0.3`

### 2. Извлечение технологий (батчами)

**System:**
```
You extract technology stacks from job vacancy descriptions. Return only
valid JSON.
```

**User:**
```
You are a technology stack extraction expert. Analyze the following {N} job
vacancy descriptions and extract ALL mentioned technologies, programming
languages, frameworks, libraries, tools, platforms, databases, cloud services,
DevOps tools, and technical skills.

For each vacancy, return a JSON array of normalized technology/skill names.

--- Vacancy 1 (ID: 12345) ---
Title: ...
Description: ...
Key Skills: ...

--- Vacancy 2 (ID: 67890) ---
...

Return ONLY valid JSON (no markdown fences):
{"results":[{"id":"vacancy_id","technologies":["Tech1","Tech2"]}]}

Rules:
- Normalize names (JS -> JavaScript, PG -> PostgreSQL, K8s -> Kubernetes,
  TS -> TypeScript)
- Include: programming languages, frameworks, libraries, databases, cloud
  platforms, DevOps tools, methodologies (Agile, Scrum, etc.)
- Only extract explicitly mentioned technologies from the text
- If a vacancy has no clear technologies, return an empty array for it
```

**Параметры:** `temperature: 0`, `response_format: { type: "json_object" }`

### 3. Анализ трендов по вакансиям

**System:**
```
You are a senior technology market analyst. RULES: Write ONLY in Russian.
Be EXTREMELY concise. No introductions, no conclusions, no filler. Every
sentence must cite specific data. Use bullet points. Max 3-5 points per
section.
```

**User:**
```
You are a senior technology market analyst specializing in IT job market
trends. Analyze the following data extracted from {N} REAL job vacancies
for the role "{vacancy_name}" on HeadHunter (Russia's largest job platform)
over the last {period} days.

TOP TECHNOLOGIES BY FREQUENCY:
1. Python — 189 mentions (75.6% of vacancies)
2. SQL — 156 mentions (62.4% of vacancies)
...

EXPERIENCE DISTRIBUTION: Нет опыта: 12, От 1 года до 3 лет: 89, ...
TOP LOCATIONS: Москва: 120, Санкт-Петербург: 45, ...

Your task: provide a CONCISE, DATA-DRIVEN trend analysis in RUSSIAN.
STRICT RULES: NO introductions, NO preambles, NO generic statements like
"В целом рынок...", NO repetition, NO filler. ONLY specific facts with
numbers from the data. Max 3-5 bullet points per section.

Use these EXACT section headers:

## 🔥 Неожиданные и растущие тренды
List 3-5 SURPRISING technologies with exact numbers.

## 📊 Ожидаемый стек (baseline)
List core expected technologies with numbers.

## 🚀 Что набирает популярность
3-5 growing technologies with numbers.

## ⚠️ Возможные blind spots
List missing technologies, one line each.

## 💡 Рекомендации
3-5 concrete actions with specific technology names.

CRITICAL: Every sentence MUST contain a specific number or technology name.
```

**Параметры:** `temperature: 0.3`, `max_tokens: 4000`

### 4. Сводный анализ: вакансии + веб-тренды

**System:**
```
You are a senior technology market analyst. RULES: Write ONLY in Russian.
Be EXTREMELY concise. No introductions, no filler, no generic statements.
Every sentence must cite specific data. Use bullet points. Max 3-5 points
per section.
```

**User:**
```
You are an expert technology market analyst. You have TWO data sources to
cross-reference.

=== SOURCE 1: JOB VACANCY ANALYSIS (HeadHunter, {N} vacancies) ===
Role: "{vacancy_name}", period: {period} days

Our AI trend analysis of vacancy data:
{trend_analysis_text}

Top technologies from vacancies:
1. Python — 189 mentions
2. SQL — 156 mentions
...

=== SOURCE 2: WEB SEARCH RESULTS (Tavily, {M} results) ===
--- Web Result 1: {title} ---
URL: ...
{content}
...

=== YOUR TASK ===
Write a CONCISE cross-source analysis in RUSSIAN.

Use these EXACT section headers:

## 🌐 Кратко по веб-источникам
Summarize ALL web sources in 3-5 bullet points TOTAL. Group by theme.

## 🔄 Совпадения: что подтверждается
Technologies confirmed by both sources with numbers.

## 🆕 Новое из веба: чего нет в вакансиях
Technologies from web NOT in vacancies = future trends.

## 🔥 Главные находки и неожиданности
3-5 most surprising facts with specific numbers.

## 📌 Итоговые рекомендации
3-5 concrete actions with specific technology names.

CRITICAL: Every sentence MUST contain a specific technology name or number.
```

**Параметры:** `temperature: 0.3`, `max_tokens: 4000`

---

## LLM-вызовы: сводка

| Вызов | Назначение | Батч | Формат ответа | temperature |
|-------|-----------|------|---------------|-------------|
| Генерация ключевых слов | Расширение поискового запроса | 1 | Текст (ключевые слова через OR) | 0.3 |
| Извлечение технологий (×N) | Стек из описаний вакансий | 50 вакансий | JSON `{results: [{id, technologies}]}` | 0 |
| Retry технологий (×M) | Повтор для пропущенных | 10 вакансий | JSON `{results: [{id, technologies}]}` | 0 |
| Анализ трендов | Текстовый анализ вакансий | 1 | Markdown (русский) | 0.3 |
| Сводный анализ | Кросс-анализ: вакансии + веб | 1 | Markdown (русский) | 0.3 |

Пример масштаба: 500 вакансий → 10 батчей по 50 → 10 запросов к LLM (вместо 500). С retry — ещё до ~50 запросов батчами по 10.

---

## Технологии

| Компонент | Технология |
|-----------|-----------|
| Оркестрация | [n8n](https://n8n.io) — low-code платформа для автоматизации |
| LLM | [Gemma 3 27B IT](https://huggingface.co/google/gemma-3-27b-it) — открытая модель Google |
| Инференс LLM | [vLLM](https://github.com/vllm-project/vllm) — высокопроизводительный сервер инференса |
| Источник данных | [HH API](https://api.hh.ru) — открытый API HeadHunter |
| Веб-поиск трендов | [Tavily](https://tavily.com) — AI-оптимизированный поисковый API |
| Доставка результата | Email (SMTP) или скачивание из формы |
| Формат отчёта | HTML с инлайн SVG-графиками + Excel XML (SpreadsheetML) |

---

## HH API

Проект использует открытый API HeadHunter без аутентификации:

```
GET https://api.hh.ru/vacancies?text=...&per_page=100&period=7&search_field=name&page=0
GET https://api.hh.ru/vacancies/{id}
```

Особенности:
- До **20 страниц** по 100 вакансий = до **2 000** вакансий за запуск
- Дедупликация по ID на этапе сбора
- Header `HH-User-Agent` для идентификации приложения
- Ограничения API: 30 запросов в секунду. Батчинг по 10 параллельных запросов укладывается в лимит
- Fallback: если детали вакансии недоступны, используются snippet из поисковой выдачи

Документация: [https://api.hh.ru](https://api.hh.ru)

## Tavily Search API

[Tavily](https://tavily.com) — AI-оптимизированный поисковый API для получения структурированных результатов. Используется для поиска актуальных веб-публикаций о технологических трендах.

Система формирует поисковый запрос из названия вакансии и топ-5 технологий:
```
{vacancy_name} technology trends 2026 skills demand {top5_technologies}
```

Результаты анализируются LLM в кросс-сопоставлении с данными из вакансий — это позволяет находить тренды, которые ещё не дошли до вакансий, но уже обсуждаются в профессиональном сообществе.

---

## Установка и запуск

1. Импортируйте `SkillTrendium.json` в n8n
2. Привяжите credential для LLM-сервера (Bearer Auth, имя: `hf`) — URL вашего vLLM-сервера
3. Привяжите credential для Tavily API (имя: `Tavily account`)
4. Настройте SMTP credential для отправки email (опционально)
5. Активируйте workflow
6. Откройте форму по URL, который выдаст n8n

---

## Структура проекта

```
SkillTrendium/
├── SkillTrendium.json        # n8n workflow (весь пайплайн)
├── README.md
├── backups/                  # Резервные копии workflow
│   ├── backup_SkillTrendium.json
│   └── backup_1_SkillTrendium.json
├── demo/                     # Примеры сгенерированных отчётов
│   ├── SkillTrendium_ML_Engineer.html
│   ├── SkillTrendium_ML.html
│   ├── SkillTrendium_AIархитектор.html
│   ├── SkillTrendium_Менеджер_проекта.html
│   ├── SkillTrendium_1с.html
│   ├── SkillTrendium_LLM_разработчик.html
│   └── SkillTrendium_LLM_разработчик.xml
└── presentation/
    ├── SkillTrendium.md      # Презентация (Marp → PDF)
    └── HOW_TO_PDF.md
```

---

## Лицензия

MIT
