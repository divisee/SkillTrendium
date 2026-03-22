# SkillTrendium

**Инструмент аналитики рынка труда на основе AI.**

Автоматический сбор вакансий с HeadHunter, извлечение технологического стека через LLM, поиск актуальных трендов через Tavily и формирование аналитического отчёта с визуализацией данных. Доставка результата — скачивание из формы или отправка на email.

---

## Целевая аудитория

| Аудитория | Задача |
|-----------|--------|
| **Соискатели** | Понять, какие технологии востребованы при смене направления |
| **HR и рекрутинг** | Быстрый обзор рынка: технологии, зарплаты, распределение по опыту |
| **Образовательные платформы** | Актуализация программ обучения на основе реальных требований |
| **Карьерные консультанты** | Статистика и тренды для обоснованных рекомендаций |

---

## Демо-отчёты

Примеры отчётов, сгенерированных системой:

| Запрос | Отчёт |
|--------|-------|
| ML Engineer | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_ML_Engineer.html) |
| ML | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_ML.html) |
| AI-архитектор | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_AIархитектор.html) |
| Менеджер проекта | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_Менеджер_проекта.html) |
| 1С | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_1с.html) |
| LLM-разработчик | [Открыть](https://divisee.github.io/SkillTrendium/demo/SkillTrendium_LLM_разработчик.html) |

---

## Архитектура пайплайна

Пайплайн реализован в **n8n** и включает 10 этапов:

```
Форма → Email-роутинг → Ключевые слова (LLM) → Поиск вакансий (HH API) →
→ Описания → Технологии (LLM, батчи) → Retry → AI-анализ трендов (LLM) →
→ Веб-анализ (Tavily + LLM) → Отчёт + Доставка
```

### Описание этапов

**1. Ввод запроса.**
Пользователь указывает название вакансии, период анализа (до 30 дней) и опционально email для доставки.

**2. Email-роутинг.**
При выборе email-доставки система мгновенно подтверждает приём запроса и продолжает обработку в фоне. Без email — пользователь ожидает завершения и скачивает отчёт из формы.

**3. Генерация ключевых слов (LLM).**
Модель генерирует до 20 поисковых вариаций на русском и английском (оператор `OR`) для расширения охвата поиска.

**4. Поиск вакансий (HH API).**
Запрос к API HeadHunter: `search_field=name`, `per_page=100`, до **20 страниц** (до **2 000 вакансий**). Дедупликация по ID.

**5. Получение описаний.**
Параллельный запрос `GET /vacancies/{id}` для каждой вакансии. Извлекаются: описание, ключевые навыки, зарплата, опыт, город. HTML очищается от тегов.

**6. Извлечение технологий (LLM, батчами по N).**
Вакансии группируются в батчи по N штук (размер настраивается; при тестировании N=50). Каждый батч — один запрос к LLM, который возвращает JSON с нормализованными названиями технологий для каждой вакансии.

**7. Retry.**
Вакансии без извлечённых технологий автоматически отправляются на повторную обработку меньшими батчами.

**8. AI-анализ трендов (LLM).**
На вход — топ-50 технологий с частотами, распределение по опыту и локациям. На выходе — структурированный анализ: неожиданные тренды, ожидаемый стек, растущие технологии, blind spots, рекомендации.

**9. Веб-анализ (Tavily + LLM).**
Поисковый запрос из названия вакансии и топ-5 технологий. Результаты кросс-сопоставляются с данными из вакансий: совпадения, новые тренды из веба, итоговые рекомендации.

**10. Формирование отчёта.**
HTML-отчёт (инлайн SVG-графики, тёмная тема, адаптивная вёрстка, поддержка печати) + Excel XML (6 листов). Доставка: скачивание или email.

---

## Состав отчёта

| Секция | Содержание |
|--------|-----------|
| Сводная статистика | Вакансий, технологий, городов, веб-источников |
| Топ-25 технологий | Горизонтальная столбчатая диаграмма (SVG) |
| Распределение по опыту | Кольцевая диаграмма (SVG) |
| Зарплатные диапазоны | Кольцевая диаграмма по бакетам |
| Топ-15 локаций | Горизонтальная столбчатая диаграмма (SVG) |
| AI-анализ трендов | Текстовый анализ по данным вакансий |
| Веб-источники Tavily | Список источников с описаниями |
| Сводный анализ | Кросс-анализ: вакансии + веб-тренды |
| Таблица технологий | Полный рейтинг с прогресс-барами |
| Список вакансий | До 200 вакансий с тегами технологий |

---

## LLM-промпты

В пайплайне 4 типа вызовов к LLM (+ retry).

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

## Неожиданные и растущие тренды
## Ожидаемый стек (baseline)
## Что набирает популярность
## Возможные blind spots
## Рекомендации

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

## Кратко по веб-источникам
## Совпадения: что подтверждается
## Новое из веба: чего нет в вакансиях
## Главные находки и неожиданности
## Итоговые рекомендации

CRITICAL: Every sentence MUST contain a specific technology name or number.
```

**Параметры:** `temperature: 0.3`, `max_tokens: 4000`

---

## Сводка LLM-вызовов

| Вызов | Назначение | Батч | Формат ответа | temperature |
|-------|-----------|------|---------------|-------------|
| Генерация ключевых слов | Расширение поискового запроса | 1 | Текст (ключевые слова через OR) | 0.3 |
| Извлечение технологий (×N) | Стек из описаний вакансий | N вакансий (настраивается) | JSON `{results: [{id, technologies}]}` | 0 |
| Retry технологий (×M) | Повтор для пропущенных | Меньший батч | JSON `{results: [{id, technologies}]}` | 0 |
| Анализ трендов | Текстовый анализ вакансий | 1 | Markdown (русский) | 0.3 |
| Сводный анализ | Кросс-анализ: вакансии + веб | 1 | Markdown (русский) | 0.3 |

Пример масштаба (при N=50): 500 вакансий → 10 батчей → 10 запросов к LLM вместо 500.

---

## Технологический стек

| Компонент | Технология |
|-----------|-----------|
| Оркестрация | [n8n](https://n8n.io) — low-code платформа автоматизации |
| LLM | [Gemma 3 27B IT](https://huggingface.co/google/gemma-3-27b-it) — открытая модель Google |
| Инференс | [vLLM](https://github.com/vllm-project/vllm) — высокопроизводительный сервер инференса |
| Источник данных | [HH API](https://api.hh.ru) — открытый API HeadHunter |
| Веб-поиск | [Tavily](https://tavily.com) — AI-оптимизированный поисковый API |
| Доставка | Email (SMTP) или скачивание из формы |
| Отчёт | HTML с инлайн SVG-графиками + Excel XML |

---

## Установка и запуск

1. Импортируйте `SkillTrendium.json` в n8n
2. Привяжите credential для LLM-сервера (Bearer Auth, имя: `hf`) — URL вашего vLLM-сервера
3. Привяжите credential для Tavily API (имя: `Tavily account`)
4. Настройте SMTP credential для отправки email (опционально)
5. Активируйте workflow
6. Откройте форму по URL, который выдаст n8n

---

## Лицензия

MIT
