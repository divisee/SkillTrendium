---
marp: true
theme: default
paginate: true
style: |
  @import url('https://fonts.googleapis.com/css2?family=Raleway:wght@400;500;600;700;800;900&family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600;700&display=swap');

  section {
    font-family: 'Space Grotesk', 'Inter', system-ui, sans-serif;
    padding: 40px 60px;
    background: linear-gradient(160deg, #f8fafc 0%, #eef2ff 40%, #e0e7ff 100%);
    color: #1e293b;
    letter-spacing: 0.01em;
  }

  /* ===== TITLE SLIDES ===== */
  section.title {
    display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center;
    background: linear-gradient(135deg, #1e1b4b 0%, #312e81 30%, #4338ca 70%, #6366f1 100%);
    color: #e0e7ff;
    position: relative; overflow: hidden;
  }
  section.title::before {
    content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%;
    background: radial-gradient(ellipse at 30% 50%, rgba(99,102,241,0.3) 0%, transparent 50%),
                radial-gradient(ellipse at 70% 30%, rgba(6,182,212,0.2) 0%, transparent 40%);
    pointer-events: none;
  }
  section.title h1 { font-family: 'Raleway', sans-serif; font-size: 3.4em; font-weight: 900; color: #fff; text-shadow: 0 2px 20px rgba(99,102,241,0.5); position: relative; z-index: 1; letter-spacing: -0.02em; }
  section.title p { font-size: 1.2em; color: #c7d2fe; position: relative; z-index: 1; }
  section.title strong { color: #a5b4fc; }
  section.title em { color: #94a3b8; font-style: normal; }

  /* ===== COLORED ACCENTS (same blue bg, colored left border) ===== */
  section.problem { border-left: 6px solid #ef4444; }
  section.problem h2 { color: #dc2626; border-bottom-color: #ef4444; }

  section.solution { border-left: 6px solid #22c55e; }
  section.solution h2 { color: #16a34a; border-bottom-color: #22c55e; }

  section.tech { border-left: 6px solid #3b82f6; }
  section.tech h2 { color: #2563eb; border-bottom-color: #3b82f6; }

  section.money { border-left: 6px solid #f59e0b; }
  section.money h2 { color: #b45309; border-bottom-color: #f59e0b; }

  /* ===== COMMON ELEMENTS ===== */
  h1 { font-family: 'Raleway', sans-serif; color: #312e81; font-size: 2.2em; font-weight: 800; margin-bottom: 0.3em; letter-spacing: -0.02em; }
  h2 { font-family: 'Raleway', sans-serif; color: #4338ca; font-size: 1.55em; font-weight: 700; border-bottom: 3px solid #6366f1; padding-bottom: 8px; margin-bottom: 16px; letter-spacing: -0.01em; }
  h3 { font-family: 'Raleway', sans-serif; color: #5b21b6; font-size: 1.15em; font-weight: 700; }
  strong { color: #3730a3; }
  em { color: #64748b; }
  a { color: #2563eb; }

  table { font-size: 0.82em; border-radius: 8px; overflow: hidden; }
  th { background: linear-gradient(135deg, #4338ca, #6366f1); color: white; padding: 10px 14px; font-weight: 600; font-family: 'Raleway', sans-serif; letter-spacing: 0.02em; }
  td { background: rgba(255,255,255,0.7); padding: 9px 14px; border: 1px solid #c7d2fe; color: #1e293b; }

  code { background: rgba(99,102,241,0.1); color: #4338ca; padding: 2px 8px; border-radius: 6px; font-size: 0.9em; }

  ul, ol { line-height: 1.8; }
  li { margin-bottom: 4px; }

  img { border-radius: 12px; }

  /* ===== ACCENT BOX ===== */
  blockquote {
    background: linear-gradient(135deg, rgba(99,102,241,0.08), rgba(6,182,212,0.08));
    border-left: 4px solid #6366f1;
    border-radius: 0 12px 12px 0;
    padding: 14px 20px;
    margin: 12px 0;
    font-size: 0.95em;
  }
  blockquote p { margin: 0; }

  /* ===== SCREENSHOT SLIDES ===== */
  section.screenshot {
    padding: 0;
    display: flex;
    align-items: flex-end;
    justify-content: flex-start;
  }
  section.screenshot .badge {
    background: rgba(30, 27, 75, 0.82);
    backdrop-filter: blur(8px);
    color: #e0e7ff;
    font-family: 'Raleway', sans-serif;
    font-weight: 700;
    font-size: 0.85em;
    padding: 8px 22px;
    border-radius: 0 14px 0 0;
    letter-spacing: 0.02em;
  }
---

<!-- _class: title -->

# SkillTrendium

**AI-помощник для анализа трендов навыков на рынке труда**

*Вакансии HeadHunter + Tavily + LLM = аналитический отчёт за минуты*

![bg opacity:0.25](source/title_hero.png)

---

## О проекте

**Тип:** Инструмент аналитики рынка труда

**Статус:** MVP

> **Что делает:** Собирает вакансии с HH, извлекает навыки через LLM, ищет тренды через Tavily, генерирует HTML-отчёт с графиками и отправляет на почту.

**Для кого:**
- **Соискатели** -- понять, что учить для смены направления
- **HR-команды** -- анализ рынка за минуты вместо часов
- **Образовательные платформы** -- актуализация программ
- **Карьерные консультанты** -- данные для рекомендаций

---

## Команда

| Имя | Роль |
|-----|------|
| Распутина Анастасия | Разработка, архитектура, ML |

**Репозиторий:** GitHub -- SkillTrendium

---

<!-- _class: problem -->

## Проблема: соискатель

**Что происходит сейчас:**
- **4-6 часов** на ручной анализ рынка при смене направления
- Повторяется **1-2 раза в месяц** в активном поиске
- Нерелевантные вакансии --> неэффективное обучение --> редкие офферы

**Стоимость ручного анализа:**
**4 000 -- 12 000 rub** на одного пользователя в месяц

> **С SkillTrendium:** Сокращение до **20-30 минут** --> экономия значимой части времени и денег

---

<!-- _class: problem -->

## Проблема: B2B

**HR-команды, рекрутинговые агентства, образовательные платформы:**

- **50-200 вакансий** анализируются для одного обзора рынка
- **6-12 часов** работы аналитика на один отчёт
- Задача возникает **2-4 раза в месяц**

**Стоимость ручной аналитики:**
**18 000 -- 144 000 rub** в месяц на одну команду

> **Результат без автоматизации:** Решения на основе неполных или устаревших данных

---

<!-- _class: solution -->

## Решение

![bg right:30% fit](source/pipeline_flow.png)

**SkillTrendium** -- полный цикл аналитики:

1. Пользователь вводит вакансию и период
2. LLM генерирует ключевые слова
3. HH API собирает вакансии
4. LLM извлекает технологии (батчами)
5. LLM анализирует тренды
6. **Tavily** ищет веб-тренды
7. LLM формирует сводный анализ
8. HTML-отчёт с графиками
9. **Отправка отчёта на e-mail**

---

<!-- _class: tech -->

## Технический подход

| Компонент | Технология |
|-----------|-----------|
| Оркестрация | **n8n** -- low-code автоматизация |
| LLM | **Gemma 3 27B IT** -- открытая модель Google |
| Инференс | **vLLM** -- высокопроизводительный сервер |
| Вакансии | **HH API** -- открытый API HeadHunter |
| Веб-тренды | **Tavily** -- AI-поисковый API |
| Доставка | **E-mail** -- отправка отчёта на почту |
| Обработка | **Батчи по N** -- оптимизация LLM-вызовов |
| Отчёт | **HTML + SVG-графики** + Excel XML |

---

<!-- _class: tech -->

## Схема: ключевые технологии

```
┌─────────────────────────────────────────────────────────────────┐
│                        ПОЛЬЗОВАТЕЛЬ                             │
│                    (форма ввода запроса)                         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     n8n  ·  Оркестрация                         │
│  ┌──────────┐   ┌──────────────┐   ┌─────────────────────────┐  │
│  │  HH API  │──▶│  Gemma 3 27B │──▶│  Tavily Search API      │  │
│  │ вакансии │   │   (vLLM)     │   │  веб-тренды             │  │
│  └──────────┘   │  извлечение  │   └─────────────────────────┘  │
│                 │  технологий  │                                 │
│                 │  + анализ    │                                 │
│                 └──────────────┘                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              HTML-отчёт (SVG-графики) + Excel XML               │
│                  скачивание / e-mail                             │
└─────────────────────────────────────────────────────────────────┘
```

---

<!-- _class: tech -->

## Пользовательский сценарий

```
Пользователь                           Система
     |                                    |
     |  "Data Engineer", 14 дней          |
     |----------------------------------->|
     |                                    |  LLM --> 20 ключевых слов
     |                                    |  HH API --> 250 вакансий
     |                                    |  LLM --> технологии (25 батчей)
     |                                    |  LLM --> анализ трендов
     |                                    |  Tavily --> веб-тренды
     |                                    |  LLM --> сводный анализ
     |                                    |  SVG-графики + HTML-отчёт
     |        Скачать + письмо на почту   |
     |<-----------------------------------|
```

---

<!-- _class: money -->

## Финансовая оценка

### Для соискателя

| Параметр | Ручной анализ | С SkillTrendium |
|----------|:---:|:---:|
| Время | 4-6 часов | 20-30 минут |
| Стоимость/мес | 4 000 -- 12 000 rub | ~0 rub |

### Для B2B (одна HR-команда)

| Параметр | Ручной анализ | С SkillTrendium |
|----------|:---:|:---:|
| Время на отчёт | 6-12 часов | ~30 минут |
| Стоимость/мес | 18 000 -- 144 000 rub | Инфраструктура |
| **Экономия/мес** | | **50 000 -- 80 000 rub** |

---

<!-- _class: money -->

## Стоимость и окупаемость

| Раздел | Сумма |
|--------|-------|
| Разработка (разово) | ~0 rub (open-source стек) |
| Cloud GPU A100 40/80GB (в месяц) | 100 000 -- 150 000 rub |
| n8n хостинг (в месяц) | 0 rub (self-hosted) |
| Tavily API (в месяц) | 0 rub (1 000 запросов бесплатно) |
| Поддержка (в месяц) | Минимальная |

> GPU-инфраструктура тиражируется на другие AI-решения компании -- затраты на каждый проект **существенно ниже**.

### ROI (на примере одной HR-команды)

- **Экономия:** 50 000 -- 80 000 rub/мес (замена ручного анализа)
- **Затраты GPU (доля проекта при 5 решениях):** ~25 000 rub/мес
- **ROI = (экономия - затраты) / затраты = 100--220%**
- **Срок окупаемости:** 1--2 месяца

**Альтернатива:** Проприетарные LLM (GPT, Perplexity) -- без GPU, оплата за токены

---

## Безопасность и ограничения

**Конфиденциальность:**
Анализируются только открытые данные -- публичные вакансии с hh.ru. Персональные данные не обрабатываются.

**Основные риски:**

| Риск | Митигация |
|------|-----------|
| Изменение HH API | Мониторинг, адаптация запросов |
| LLM галлюцинации | Structured JSON output, валидация |
| Rate-limit HH API (30 req/s) | Последовательная обработка, лимит 300 |

> **LLM:** Локальная (Gemma, Qwen на vLLM) или проприетарная (GPT) -- конфиденциальная информация отсутствует

---

<!-- _class: solution -->

## Текущий статус и демо

**Что готово:**
- n8n пайплайн -- полный цикл из **9 этапов** (включая e-mail)
- Форма ввода с кастомным дизайном
- Интеграция с HH API, LLM, Tavily и e-mail
- HTML-отчёт с SVG-графиками и AI-аналитикой
- **Автоматическая отправка отчёта на почту**

**Демо:**
- Работа пайплайна в n8n
- Пример отчёта по реальному запросу
- Форма ввода и отправка на почту

---

<!-- _class: screenshot -->

<div class="badge">Форма ввода</div>

![bg contain](source/main_form.jpg)

---

<!-- _class: screenshot -->

<div class="badge">Пайплайн n8n</div>

![bg contain](source/n8n_pipe.jpg)

---

<!-- _class: screenshot -->

<div class="badge">Отчёт: графики технологий</div>

![bg contain](source/report1.jpg)

---

<!-- _class: screenshot -->

<div class="badge">Отчёт: аналитика и тренды</div>

![bg contain](source/report2.jpg)

---

<!-- _class: screenshot -->

<div class="badge">Отчёт: детальная статистика</div>

![bg contain](source/report3.jpg)

---

<!-- _class: screenshot -->

<div class="badge">Отправка на e-mail</div>

![bg contain](source/example_email.jpg)

---

<!-- _class: title -->

# Спасибо!

**SkillTrendium** -- данные вместо догадок

*Вопросы?*

![bg opacity:0.3](source/thankyou_bg.png)
