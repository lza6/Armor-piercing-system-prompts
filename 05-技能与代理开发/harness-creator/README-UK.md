# harness-creator

Компактний скіл для побудови та аудиту harness навколо агентів AI-кодування.

Він допомагає репозиторію забезпечити агентам п'ять необхідних речей: інструкції, стан, верифікацію, межі обсягу та передачу життєвого циклу.

## Встановлення

```bash
npx skills add walkinglabs/learn-harness-engineering --skill harness-creator
```

Або скопіюйте `skills/harness-creator/` до вашого шляху скілів.

## Використання

```bash
node skills/harness-creator/scripts/create-harness.mjs --target /path/to/project
node skills/harness-creator/scripts/validate-harness.mjs --target /path/to/project
node skills/harness-creator/scripts/run-benchmark.mjs --target /path/to/project --html /path/to/report.html
```

Скрипти використовують лише вбудовані модулі Node.js. Їх можна запускати після копіювання директорії скіла до іншого репозиторію.

## Що створюється

- `AGENTS.md` або `CLAUDE.md`
- `feature_list.json`
- `progress.md`
- `init.sh`
- `session-handoff.md`

`create-harness.mjs` визначає поширені типи проєктів і пакетні менеджери. Підтримуються Node/npm/pnpm/yarn/bun, Python, Go, Rust, Maven, Gradle і .NET на базовому рівні команд верифікації.

## Що перевіряється

`validate-harness.mjs` оцінює п'ять підсистем harness:

1. Інструкції
2. Стан
3. Верифікація
4. Обсяг
5. Життєвий цикл

Оцінка є структурною. Вона показує, чи наявний harness і чи є він узгодженим; вона не замінює реального тестування сесій агента до і після змін.

## Стан

- [x] Мінімальне scaffolding harness
- [x] Валідація п'яти підсистем
- [x] HTML-звіт оцінки
- [x] Структурний benchmark-звіт
- [x] 10 eval-кейсів
- [x] Загальне визначення верифікації для поширених стеків
- [ ] Опціональне відтворення сесій агента до/після (реальне)

## Файли

```text
harness-creator/
├── SKILL.md
├── metadata.json
├── agents/openai.yaml
├── scripts/
│   ├── create-harness.mjs
│   ├── validate-harness.mjs
│   ├── render-assessment-html.mjs
│   ├── run-benchmark.mjs
│   └── lib/harness-utils.mjs
├── templates/
│   ├── agents.md
│   ├── feature-list.json
│   ├── feature-list.schema.json
│   ├── init.sh
│   ├── progress.md
│   └── session-handoff.md
├── references/
└── evals/evals.json
```

## Межі

Цей скіл призначений для інженерії harness, а не для вибору моделей, виключно для налаштування промптів або архітектури застосунку. Зберігайте специфічні факти проєкту в цільовому репозиторії.
