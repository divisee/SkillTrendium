# Как сгенерировать PDF из презентации

Презентация написана в формате [Marp](https://marp.app/) — Markdown Presentation Ecosystem.

## Способ 1: VS Code / Cursor (рекомендуется)

1. Установите расширение **Marp for VS Code** (id: `marp-team.marp-vscode`)
2. Откройте `SkillTrendium.md`
3. Нажмите иконку Marp в правом верхнем углу → **Export Slide Deck**
4. Выберите формат PDF

## Способ 2: CLI (npx, без установки)

```bash
cd presentation
npx @marp-team/marp-cli SkillTrendium.md --pdf --allow-local-files --no-stdin --html
```

Требуется Node.js. Файл `SkillTrendium.pdf` появится в той же папке.

## Добавление скриншотов

Положите изображения в папку `source/` и ссылайтесь на них в презентации:

```markdown
![Скриншот формы](source/form_screenshot.png)
```
