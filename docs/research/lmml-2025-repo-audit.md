# LMML 2025: аудит публичного репозитория `lmml-solutions` (без флагов и готовых решений)

> **Статус документа:** исследовательский отчёт для подготовки к LMML 2.0.
> **Флаги, итоговые слова и закодированные строки ответов из исходников сюда намеренно не перенесены.**
> Ссылки на секции `Solution` исходного репозитория ведут прямо на спойлеры — открывайте их только после тренировки.

---

## 0. TL;DR — 10 главных наблюдений

| # | Наблюдение | Тип | Уверенность |
|---|---|---|---|
| 1 | В архиве **ровно 3 задания** (`australia-neural-cypher`, `morocco-he-never-understands-me`, `paraguay-you-cant-stop-me`). По внешним источникам в 2025 году заданий было около 23–25, то есть архив покрывает примерно **12–13%** и подобран специально: это «самые запрашиваемые/сложные» задания ([README L3](#ссылки)). | подтверждено (дерево) + вывод (доля) | высокая / средняя |
| 2 | **Ни одно** из трёх заданий не требует обучения модели с нуля или fine-tuning. Два — zero-training (анализ весов; диалог с LLM), одно — лёгкая подгонка линейной модели (Ridge на признаках) за секунды на CPU. | вывод из Task+Solution | высокая |
| 3 | Во всех трёх ответ — это **строка или флаг, спрятанные в артефакте**: биты весов, кодировка диалога, Morse-сигнал в данных. ML здесь — средство «увидеть» сигнал, а не цель с метрикой. | подтверждено | высокая |
| 4 | **Сюжет — это техническая подсказка.** Авторы прямо пишут, что фразы из storyline указывают на механизм (Australia [L105–107], Paraguay [L44–48]). | подтверждено | высокая |
| 5 | Повторяется трюк «**поступи наоборот**»: не переобучать (Australia: «Do not retrain»), переобучить нарочно (Paraguay), подстроиться под модель, а не модель под себя (Morocco). | подтверждено + интерпретация | средне-высокая |
| 6 | Scoring **разный в каждой задаче**: 100/50/25/0 только в Australia; «если слово верное — pass» в Paraguay; в Morocco scoring не указан вовсе. Общих правил оценки LMML в репозитории **нет**. | подтверждено | высокая |
| 7 | Авторский разбор **неполный**: в Australia объяснено извлечение только первой половины флага (T5). Как достать вторую часть (SimpleCNN), не написано. В тексте Australia есть внутреннее противоречие («ищи в конце `parameters()`» vs. скрипт берёт **первый** параметр). | подтверждено | высокая |
| 8 | Внешние артефакты хрупкие: статья по ссылке в Australia — заглушка PythonAnywhere; датасет Paraguay (Google Drive) при анонимном запросе отдал 404; T5-модель на HF **gated** (нужен логин и согласие). Для Morocco артефактов нет совсем: LLM-эндпоинт не опубликован. | подтверждено (13.11.2026) | высокая |
| 9 | Риски безопасности: `trust_remote_code=True` без `revision`-пина (SimpleCNN), скачивание 242 МБ весов, неконтролируемые зависимости. Сейчас код SimpleCNN на HF — безобидные 843 байта, веса в `safetensors` (pickle нет), но репозиторий может измениться. | подтверждено (статический просмотр) | высокая |
| 10 | Публичные анонсы LMML 2.0 (сайт и unde.io) **подтверждают** список доменов по континентам из Mentor Guideline и добавляют седьмой — «Antarctica / classified». География заданий 2025 года **не совпадает** с доменами 2026 (например, Paraguay = регрессия, а Южная Америка 2026 = AI Security). | подтверждено (внешние страницы) | высокая |

> ⏰ **Срочно:** по странице unde.io LMML 2.0 проходит **14–15 ноября 2026**, то есть **завтра** относительно даты отчёта (13.11.2026). Поэтому `lmml-2026-prep-plan.md` ставит на первое место план «на вечер/1 день».

---

## 1. Методология и границы

| Параметр | Значение |
|---|---|
| Исходный репозиторий | <https://github.com/SigmoidAI/lmml-solutions> (read-only) |
| Default branch | `main` (единственная ветка; тегов и releases нет — 0) |
| Проверенный commit | [`f92ba23df8706a7adc3d14770bb139e0c5633596`](https://github.com/SigmoidAI/lmml-solutions/commit/f92ba23df8706a7adc3d14770bb139e0c5633596) (2025-11-30, «docs: update formatting mistake») — HEAD `main` на момент проверки |
| Рабочая копия | форк `barbalatv/lmml-solutions`, ветка `arena/f3c18e36-lmml-solutions`, ответвлена от того же SHA `f92ba23` (содержимое идентично upstream) |
| Дата анализа | 2026-11-13 |
| Issues / PR в upstream | 0 / 0 (проверено через GitHub API) |
| Лицензия | собственная «Sigmoid Open-Source License (SOSL-1.0)»: non-commercial + share-alike ([LICENSE](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/LICENSE)). GitHub определяет её как `NOASSERTION` |

**Что сделано:**

1. Прочитаны все 11 файлов дерева: 4 текстовых и 5 PNG; изображения `raw-data.png` и `simple-ridge-low-degree.png` просмотрены визуально.
2. Через GitHub API просмотрена история всех 9 коммитов: какие файлы и как менялись.
3. Внешние ссылки проверены **только метаданными**: HF Hub API (`/api/models/...`), сырой текст `config.json`, `configuration_simplecnn.py`, `modeling_simplecnn.py` (как текст), HTTP-просмотр статьи и ссылки на Google Drive.
4. В Google Drive пользователя искался `LMML_2026_Mentors_Guideline.pdf` — **не найден** (найден только `Mentors.m4a`, его не открывали). Поэтому все сведения из Mentor Guideline ниже — **предоставленный контекст, мною не проверенный**. Часть из них подтверждается публичными страницами (раздел 7).

**Чего сознательно НЕ делали:** не запускали `extract_flag.py` и сниппеты из README; не скачивали `model.safetensors` (242 МБ) и `train.csv`; не вызывали `from_pretrained`; не использовали `trust_remote_code`; не декодировали base64-строки из Morocco; не восстанавливали флаги. Поэтому всё, что касается **фактического** поведения артефактов, помечено как «не проверено запуском».

**Шкала уверенности:** *высокая* — прямо видно в файле или метаданных; *средняя* — логический вывод из нескольких фактов; *низкая* — гипотеза.

---

## 2. Инвентаризация файлов

| Путь | Размер (байт) | Тип | Добавлен в коммите | Содержимое |
|---|---:|---|---|---|
| `README.md` | 1009 | md | `89da14e` → `2b14cdc` → `4ffe060` | Назначение репозитория; просьба создавать issues для новых разборов |
| `LICENSE` | 1161 | text | `89da14e` → `2b14cdc` | SOSL-1.0 |
| `challenges/australia-neural-cypher/README.md` | 11291 | md | `6dfcf46` (как `neural-cypher/`), `7b2edbd` (1 строка), `4fcb791` (rename) | Task + Solution (T5) |
| `challenges/australia-neural-cypher/extract_flag.py` | 1596 | py | `6dfcf46` | Копия «полного скрипта» из README (только T5) |
| `challenges/morocco-he-never-understands-me/README.md` | 1547 | md | `c4d025d`, `f92ba23` (только уровни заголовков) | Короткий Task + Solution |
| `challenges/paraguay-you-cant-stop-me/README.md` | 10073 | md | `44d1c95` | Task + подробный Solution |
| `challenges/paraguay-you-cant-stop-me/assets/raw-data.png` | 170398 | png | `44d1c95` | Scatter сырых данных |
| `…/assets/simple-ridge-low-degree.png` | 34765 | png | `44d1c95` | Кривая Method 1 |
| `…/assets/ridge-with-fourier-features.png` | 85860 | png | `44d1c95` | Кривая Method 2 |
| `…/assets/ridge-polynomial-fourier-features.png` | 76344 | png | `44d1c95` | Кривая Method 3 (**спойлер**) |
| `…/assets/overfit-prediction-curve.png` | 82254 | png | `44d1c95` | Кривая с разметкой (**спойлер**) |

**В репозитории нет:** `requirements.txt`, `pyproject`, notebook'ов, CI, датасетов, весов, evaluator-кода, кода платформы и правил участников. Единственное указание версий — строка `pip install torch==2.9.0 transformers==4.57.1 numpy==2.1.3` в Australia ([L351–354](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L351-L354)).

**Хронология (UTC):** репозиторий создан 2025-11-18, последний push — 2025-11-30. Сам марафон прошёл 1–2 ноября 2025 (unde.io, FCIM UTM), то есть разборы опубликованы через 2–4 недели после события. HF-модели Australia созданы раньше: T5 — 2025-03-09 (последнее изменение 2025-04-12), SimpleCNN — 2025-04-22. *Вывод (средняя уверенность):* задание, вероятно, готовилось заранее или переиспользовано из более ранней активности.

---

## 3. Сводная таблица заданий

| Поле | Australia — Neural Cypher | Morocco — He never understands me | Paraguay — You can't stop me |
|---|---|---|---|
| Путь | `challenges/australia-neural-cypher/` | `challenges/morocco-he-never-understands-me/` | `challenges/paraguay-you-cant-stop-me/` |
| Страна / континент (2025) | Австралия / Океания | Марокко / Африка | Парагвай / Юж. Америка |
| Домен по сути (моя классификация) | Model forensics / стеганография в весах (AI security) | Взаимодействие с LLM, кодировки (GenAI / prompt-игра) | Регрессия и обработка сигнала + декодирование Morse (classic ML / DS) |
| Цель | Достать 2 частичных флага из весов двух моделей и склеить (T5 → CNN) | Получить флаг от «Flag Keeper AI» | Найти скрытое слово в шумном наборе (x, y) |
| Вход | 2 модели на HF Hub | Чат с LLM (эндпоинт не опубликован) | `train.csv`, ~15 000 точек (Google Drive) |
| Выход | Строка-флаг (конкатенация) | Флаг в ответе LLM | Слово |
| Scoring (из README **этой** задачи) | 100% / 50% / 25% / 0 | **не указан** | «If the word is correct, you pass» |
| Обучение | **Zero-training** (только инспекция весов) | **Zero-training** (только inference через чужую LLM) | **Лёгкое обучение**: Ridge на ~216 признаках, секунды на CPU; без обучения тоже возможно (сглаживание) |
| Вычисления | CPU, ~1–2 ГБ RAM (242 МБ весов fp32 + numpy-копии); GPU не нужен | Почти ноль | CPU, < 1 ГБ RAM |
| Сколько ML на деле | Низко: нужно знание PyTorch/HF и float32 | Низко: нужна гипотеза о кодировке | Средне: базисные функции, регуляризация |
| Ручной этап | Подбор тензора, порядка битов, кодировки | Распознать кодировку, переписываться в ней | Визуальное чтение пиков и перевод Morse |
| Внешние зависимости | HF Hub (T5 gated), `trust_remote_code` | Хостинг LLM организаторов | Google Drive |
| Воспроизводимость сейчас | Частично: T5 требует логина; SimpleCNN открыт; статья мертва | **Нет** | Не подтверждена (Drive → 404 анонимно) |
| Сложность (моя оценка) | средняя (2 части, вторая не разобрана) | низкая–средняя (зависит от «щелчка») | средняя (визуальная неоднозначность) |
| Уверенность в анализе | высокая для T5, низкая для CNN-части | средняя (мало текста) | высокая |

---

## 4. Углублённый разбор по задачам

### 4.1. Australia — Neural Cypher

**Источники:** [README](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md), [extract_flag.py](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/extract_flag.py).

#### Task — что написали авторы · Australia

- Storyline ([L10–45](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L10-L45)): «Do not retrain. Keep hidden layers locked», «payload vector», «trigger prompt», «bitmask embedding pass», «maintain internal state across sessions», «left their mark in **both models**».
- Objective ([L47–53](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L47-L53)): прямо сказано, что флаги спрятаны «within their weight files using steganography techniques». То есть **тип задачи раскрыт в условии**.
- Модели ([L55–75](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L55-L75)): `CezarCalin/hidden-flag-model` (T5) и `CezarCalin/simple-cnn` — последняя загружается через `trust_remote_code=True` ([L72](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L72)).
- Формат флага ([L77–87](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L77-L87)): конкатенация, порядок T5 → CNN.
- Scoring ([L96–101](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L96-L101)): 100% — обе части; 50% — одна; 25% — «partial bits»; 0. **Как платформа оценивала «partial bits», в репозитории не описано.**
- Ссылка на «cool article» ([L45](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L45)) сейчас открывает заглушку «Coming Soon» PythonAnywhere. Поддомен называется `sharedweights` — это сам по себе намёк.

#### Solution — механизм (без ответа) · Australia

- Авторы сводят сюжет к технике ([L105–107](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L105-L107)): «maintain internal state» → shared weights; «payload vector» → LSB-встраивание.
- Механизм ([L173–229](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L173-L229)): младший бит мантиссы float32 в общем (shared) тензоре эмбеддингов, по 1 биту на вес, UTF-8, порядок битов big-endian.
- Разобрана **только T5-часть**. Для SimpleCNN Solution не содержит ничего, хотя Task требует обе части. `extract_flag.py` тоже работает только с T5.

#### Обнаруженные несоответствия в авторском тексте

1. **«Ищи в конце» vs. «берём первый параметр».** README [L145–148](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L145-L148) и [L168–170](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L168-L170) утверждают, что shared-тензор появляется в `model.parameters()` «usually at its last occurrence» и что «flag hides at the end». При этом и сниппет ([L266](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L266)), и скрипт ([L309](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L309), [extract_flag.py L12](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/extract_flag.py#L12)) берут `next(model.parameters())`, то есть **первый** параметр. *Мой вывод (средне-высокая уверенность, запуском не проверено):* PyTorch при дедупликации общих тензоров отдаёт их при **первой** встрече. В `T5ForConditionalGeneration` модуль `shared` регистрируется первым, поэтому скрипт верен, а текст про «конец» вводит в заблуждение. **Урок:** не верить описанию порядка на слово, а перечислять тензоры по именам.
2. **Строгое UTF-8 декодирование всего тензора** ([L331–334](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L331-L334)). Скрипт читает LSB **всех** весов тензора: для T5-small это порядка 16 млн значений, около 2 МБ «байтов». Затем он декодирует их строго. Со случайными LSB после полезной нагрузки такое декодирование обычно падает. Авторы утверждают, что флаг печатается. Значит, хвост, вероятно, обнулён или как-то подготовлен, но это **не проверено**. На практике нужен устойчивый декодер: искать известный префикс или читать до первого невалидного байта.
3. В заголовках сниппетов ` ```bash ` стоит у Python-кода ([L57](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L57), [L68](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L68)). Мелочь, но показывает, что разбор писали быстро.

#### Внешние артефакты (метаданные HF, 13.11.2026)

| Модель | HF sha | Файлы | Формат весов | Параметры | Доступ |
|---|---|---|---|---|---|
| `CezarCalin/hidden-flag-model` | `c0e57e5c…` | config, tokenizer, `spiece.model`, `model.safetensors` (242 041 896 Б) | safetensors, F32 | 60 506 624 (≈ T5-small) | **gated: auto** — файлы и история коммитов только после логина и принятия условий |
| `CezarCalin/simple-cnn` | `5944e8d1…` | `config.json`, `configuration_simplecnn.py` (309 Б), `modeling_simplecnn.py` (843 Б), `model.safetensors` (427 784 Б) | safetensors, F32 | 106 776 | открыт; 11 коммитов 2025-04-22 |

- Код SimpleCNN при статическом чтении: `Conv2d(1→16)` → `MaxPool` → `Conv2d(16→32)` → `MaxPool` → `Linear(32·7·7→64)` → `Linear(64→24)`. Импортов `os`, `subprocess` и сетевых вызовов нет. Пересчёт параметров (160 + 4 640 + 100 416 + 1 560 = 106 776) **совпадает** с метаданными safetensors. Значит, в файле весов нет лишних тензоров сверх архитектуры, и полезная нагрузка, если она есть, лежит **внутри** обычных весов. *Вывод (средняя уверенность):* вход 1×28×28 и 24 класса соответствуют формату Sign Language MNIST.
- `auto_map.AutoModel` = `CezarCalin/simple-cnn--modeling_simplecnn.SimpleCNN` — ссылка на код в репозитории HF. Без `revision=` загрузится **текущая** версия кода, и автор может поменять её в любой момент. Это и есть главный риск `trust_remote_code`.

#### Интерпретация · Australia

- **Проверяемый навык:** понимание того, как веса хранятся (float32, safetensors, tied/shared weights, `state_dict` vs `parameters`), и базовая стеганография.
- **Скрытая идея:** «модель» тут — просто контейнер байтов. Решение не требует ни одного forward pass.
- **Почему наивные подходы проваливаются:** генерировать текст T5 «trigger prompt'ами» (сюжетная приманка); дообучать (стирает LSB — «Do not retrain»); загружать в fp16/bf16 или квантовать (младшие биты float32 теряются); смотреть только первый/последний тензор, не перебирая все; ждать ASCII в «целых» значениях вместо битов.
- **Минимальный стек:** `safetensors` + `numpy`. Ни `transformers`, ни `torch`, ни `trust_remote_code` для чтения весов **не нужны**: `safe_open(..., framework="np")` читает тензоры без выполнения кода.
- **Без ML:** фактически всё решается без ML.
- **Обучение:** zero-training.
- **Польза LLM и агентов:** высокая для генерации шаблонного кода разбора битов и перебора гипотез (порядок битов, номер бита, тензор). Низкая для догадки «где искать» — тут нужен человек, который читает сюжет.
- **Ресурсы:** CPU; несколько сотен МБ – 2 ГБ RAM; время упирается в скачивание (gated!).
- **Частые ошибки:** забыть принять gated-условия заранее; потерять точность при загрузке; перепутать порядок частей флага; декодировать весь тензор вместо поиска полезной нагрузки.
- **Частичный успех:** по README — 50% за одну часть. *Вывод:* есть смысл сдавать T5-часть сразу, если платформа принимает частичные ответы. В репозитории это **не подтверждено**.

---

### 4.2. Morocco — He never understands me

**Источник:** [README](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md) (39 строк — самый короткий).

#### Task — что написали авторы · Morocco

- [L11–22](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L11-L22): «secure communication system that uses **encoded messages**», «follow the proper protocol», «**encryption protocol**», «only communicates in thought messages», «only responds to properly thought messages». Вход — промпт к LLM, выход — флаг в ответе.
- **Нет:** эндпоинта или модели, системного промпта, ограничений по числу запросов, scoring, формата флага. Последний коммит `f92ba23` поменял только уровни заголовков, содержание не трогал.

#### Solution — механизм (без ответа) · Morocco

- [L26–39](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L26-L39): модель отвечала строками, похожими на «тарабарщину». Нужно было распознать в них стандартную текстовую кодировку (авторы называют base64, [L34](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L34)) и задать вопрос **в той же кодировке**. Ключевая формулировка: «adapt to the model instead of the model to adapt to you» ([L33](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L33)).
- В README приведены две закодированные строки ([L31](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L31), [L39](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L39)). В отчёт они **не переносились и не декодировались**.

#### Интерпретация · Morocco

- **Проверяемый навык:** наблюдательность при работе с LLM; знание распространённых кодировок (base64, hex, rot13, URL-encoding, Morse, binary); готовность сменить «язык» общения вместо jailbreak-атак.
- **Скрытая идея:** «защита» — это не отказ, а протокол. В слове «thought» в Task, *возможно*, игра слов (low confidence). Главная подсказка — «encoded messages / encryption protocol».
- **Почему наивные подходы проваливаются:** классические prompt injection и jailbreak на английском («ignore previous instructions…»). Модель по сюжету «не понимает» обычный язык, и попытки сломать её тратят время.
- **Стек:** браузер + CyberChef или Python `base64`. Ни ML, ни GPU.
- **Обучение:** zero-training (inference чужой модели).
- **Польза внешних LLM:** высокая. Современная LLM почти мгновенно распознаёт base64 и умеет кодировать ответ. **Можно ли передавать текст задания во внешние LLM — вопрос к регламенту.**
- **Риски и ошибки:** не попробовать декодировать «мусор»; не сохранять историю диалога (на случай, если флаг появится частично); тратить лимит запросов (если он есть — неизвестно).
- **Частичный успех:** в README не описан.
- **Воспроизводимость:** **нулевая** — без эндпоинта задачу не повторить; тренироваться можно только на аналогах (см. план).

---

### 4.3. Paraguay — You can't stop me

**Источник:** [README](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md) и 5 PNG в `assets/`.

#### Task — что написали авторы · Paraguay

- Storyline ([L12–18](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L12-L18)) **почти полностью раскрывает метод**: «fit the chaos perfectly», «peaks and valleys», «gaps of one unit… gaps of three» (тайминги Morse), «mountains are of different heights», «polynomials of high degree, or sinusoidal waves», «linear in its parameters, but nonlinear in its representation».
- Данные ([L23–27](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L23-L27)): `train.csv`, ~15 000 точек (x, y). Ссылка на Google Drive. Анонимный запрос 13.11.2026 вернул 404, через Drive-коннектор пользователя файл тоже не найден — статус **«не подтверждён»**.
- Scoring ([L32–33](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L32-L33)): бинарный — «If the word is correct, you pass».

#### Solution — механизм (без ответа) · Paraguay

- Три метода по нарастающей ([L82–217](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L82-L217)): Ridge на полиномах степени 10 → Ridge на Fourier-признаках (K = 150) → Ridge на полиномах степени 15 + Fourier K = 100 при `alpha≈1e-9` (рекомендуемый).
- Затем **ручное** чтение Morse по ширине пиков (точка/тире) и по смене **уровня высоты** пиков (граница букв) ([L219–239](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L219-L239)).
- Авторы честно предупреждают: ответ выглядит как бессмысленный набор букв, а декодирование «will probably not fit straight like a glove» ([L235](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L235), [L239](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L239)).
- ⚠️ [L240–253](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L240-L253) и PNG `ridge-polynomial-fourier-features.png` / `overfit-prediction-curve.png` содержат ответ (спойлер).

#### Что видно на изображениях (проверено визуально, без спойлеров)

- `raw-data.png`: x ∈ [0, 1], y примерно от −15 до +15. Шум большой, но **локальные сгущения и сдвиги среднего видны уже на сыром scatter** — данные не выглядят «чистым шумом».
- `simple-ridge-low-degree.png`: полином степени 10 даёт лишь плавный тренд с 2–3 горбами, и сигнал по нему не читается. Это иллюстрирует, что ёмкости низкостепенного базиса не хватает.

#### Интерпретация (моя, не авторская) · Paraguay

- **«Переобучение» здесь — во многом риторика.** 216 признаков на 15 000 точек — это **не** переобученная (overparameterized) модель, а регрессия на усечённом Fourier-базисе, то есть по сути **низкочастотный фильтр**, восстанавливающий E[y|x]. Важна не «память», а достаточная частотная ёмкость при малой регуляризации. *Следствие (средне-высокая уверенность):* тот же сигнал, скорее всего, видно и без обучения — через binned mean / rolling median по отсортированному x, фильтр Савицкого–Голея или FFT low-pass. Без датасета это **не проверено**.
- **Проверяемый навык:** feature engineering (базисные функции), понимание bias/variance, обработка сигналов, умение читать сюжет и закрыть задачу вручную.
- **Почему наивные подходы проваливаются:** «правильный» ML с CV/регуляризацией выбирает гладкую модель и уничтожает сигнал; градиентный бустинг или kNN при неудачных гиперпараметрах даёт рваную кривую; поиск «слова» в табличных значениях (ASCII) — ложный путь. Кроме того, есть риск «исправить» бессмысленное на вид слово в осмысленное, и авторы прямо предупреждают об этом.
- **Стек:** `pandas`, `numpy`, `scikit-learn`, `matplotlib`; опционально `scipy.signal` для поиска пиков.
- **Обучение:** **лёгкое** (линейная модель, секунды на CPU) **или zero-training** (сглаживание).
- **Польза LLM:** высокая для кода признаков и графиков; средняя для чтения Morse по графику (мультимодальные модели ошибаются на уровнях и ширинах — нужна проверка человеком).
- **Частые ошибки:** не отсортировать по x перед сглаживанием; переусердствовать с регуляризацией; путать границы букв (уровни) с паузами; полагаться на одну визуализацию без численного поиска пиков (`find_peaks` с шириной и высотой).
- **Частичный успех:** нет (бинарный pass).

---

## 5. Реальный соревновательный процесс — что следует из исходников

| Аспект | Что есть в репозитории | Чего нет |
|---|---|---|
| Тип ответа | Строка: флаг (Australia, Morocco) или слово (Paraguay) | Метрик-лидербордов (accuracy/F1) нет ни в одном задании |
| Формат флага | Australia: «`{flag1_content + flag2_content}`» и пример конкатенации ([L83–87](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L83-L87)); в описании кодирования упоминается вид `"flag{...}"` ([L200](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L200)) | Единого формата (`LMML{...}` и т. п.) — не указан; нормализация регистра и пробелов — не указана |
| Частичные очки | Только Australia: 100/50/25/0 | Как начислялось на платформе; есть ли частичные очки в других задачах; штрафы за неверные сдачи |
| Evaluator | — | Кода evaluator'а, описания автоматической проверки, лимитов сдач нет |
| Интернет / хостинг | Нужен: HF Hub (Australia), Google Drive (Paraguay), хостинг LLM (Morocco) | Офлайн-зеркал нет |
| Ручные шаги | Чтение сюжета, визуальное декодирование Morse, распознавание кодировки | — |
| Подсказки за очки | — | В репозитории не упоминаются (они есть только в предоставленном Mentor Guideline) |

> **Важно:** 100/50/25% — правило **одной задачи** (Australia), а не LMML в целом. Paraguay — бинарное, Morocco — не описано. Общих правил платформы в репозитории **нет**.

---

## 6. Повторяющиеся паттерны и ловушки (по 3 заданиям)

| Паттерн | Где подтверждено |
|---|---|
| **Сообщение спрятано в артефакте**, а не предсказывается: веса, ответы LLM, форма функции | Australia [L49](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L49); Morocco [L16](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L16); Paraguay [L21](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L21) |
| **Классические кодировки** как последний шаг: UTF-8-биты, base64, Morse | Australia [L196–255](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L196-L255); Morocco [L34](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L34); Paraguay [L219–231](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L219-L231) |
| **Сюжет = спецификация.** Каждая «художественная» фраза — техническая подсказка | Australia [L105–107](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L105-L107); Paraguay [L44–48](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L44-L48) |
| **Инверсия ML-интуиции**: не обучать, нарочно переобучить, подстроиться под модель | Australia [L17](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L17); Paraguay [L38](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L38); Morocco [L33](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L33) |
| **Приманки в сюжете** («trigger prompt», «payload vector», упоминание fine-tuning) уводят в дорогие действия | Australia [L19](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L19), [L25–26](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L25-L26) — *интерпретация, средняя уверенность* |
| **Перебор гипотез** (какой тензор, какой метод, какой уровень) вместо одного «правильного» пути | Australia [L306](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L306) («You can change which layer»); Paraguay [L80](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L80); Morocco [L34](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md#L34) |
| **Ответ может выглядеть бессмысленно** — не «исправлять» его | Paraguay [L235](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md#L235) |
| **Составные флаги** (несколько частей, важен порядок) | Australia [L79–87](https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md#L79-L87) |
| **Хрупкие внешние зависимости** (HF gated, Drive, мёртвые ссылки) | разделы 4.1, 4.3 |

**Ловушки:** долгое обучение «на всякий случай»; потеря точности при загрузке весов; доверие к описанию автора вместо проверки (см. противоречие в 4.1); jailbreak вместо протокола; недоверие к странному на вид ответу.

---

## 7. Сопоставление с направлениями LMML 2.0

**Источники о 2026 году:**

- (a) `LMML_2026_Mentors_Guideline.pdf` — **мне недоступен**, сведения взяты из постановки задачи как *предоставленный контекст*;
- (b) публичные страницы, проверенные 13.11.2026: [lmml.sigmoidai.org](https://lmml.sigmoidai.org/) и [unde.io — LMML 2.0](https://unde.io/events/1908-learning-marathon-of-machine-learning-2-0).

Страницы (b) **совпадают** с (a) по шести доменам и дополнительно называют «Antarctica» — особую/засекреченную зону «для тех, кто дойдёт достаточно далеко». На unde.io также указано: 24 часа, 14–15 ноября 2026, Clasa Viitorului, гибридный формат, **команды 1–5 человек**, «indicii» (подсказки) и очки. Это маркетинговые анонсы, а не регламент.

| Континент 2026 (a, b) | Домен 2026 | Что из архива 2025 релевантно по **сути** (не по географии) | Насколько релевантно |
|---|---|---|---|
| Европа | Classic ML & Data Science (EDA) | Paraguay: регрессия, базисные функции, обработка сигнала, EDA шумных данных | высокая по навыкам |
| Африка | GenAI, Audio & Agents | Morocco: работа с LLM как с «чёрным ящиком», протокол/кодировка | средняя |
| Азия | Computer Vision | SimpleCNN в Australia — CV-модель, но задача была про веса, а не про зрение | низкая |
| Южная Америка | Adversarial Attacks & Security in AI | Australia: стеганография в весах, supply-chain риски (`trust_remote_code`); Morocco: обход «защиты» LLM | высокая по навыкам |
| Северная Америка | NLP | T5 (только как контейнер), Morocco (текстовые кодировки) | низкая–средняя |
| Океания | Reinforcement Learning | **Нет ни одного RL-задания в архиве**, хотя Australia (Океания) в 2025 году была про стеганографию | нет данных |
| Антарктида | засекречено (b) | — | нет данных |

**Выводы о соответствии:**

- География 2025 года **не совпадает** с доменами 2026: Австралия-2025 — это стеганография, а не RL; Парагвай-2025 — регрессия, а не AI Security. Значит, по стране задания 2025 года **нельзя** судить о его домене (подтверждено).
- Внешний источник [FCIM UTM, 10.11.2025](https://fcim.utm.md/en/news/remarkable-performance-of-the-fcim-team-falcons-at-the-machine-learning-marathon-organized-by-sigmoid/) перечисляет для 2025 года NLP, CV, RL, GenAI, Data Science, AI Security и **25 заданий** (команда решила 21). Страница архива [lmml.sigmoidai.org/2025.html](https://lmml.sigmoidai.org/2025.html) показывает «5 thematic continents», «23 tasks», «20 hours». Счётчики анимированные, а на главной странице они дали явно неверные значения (например, «38 continents»). **Цифры противоречивы — низкая уверенность.**
- Для RL, CV, Audio и Agents архив **не даёт никакой** исторической опоры. Эти домены нужно готовить «с нуля» по общим принципам.

---

## 8. Подтверждено / вывод / неизвестно

| Подтверждено (файлы или метаданные) | Вывод (интерпретация) | Неизвестно |
|---|---|---|
| 3 задания, 11 файлов, SHA `f92ba23`, 9 коммитов, 0 issues/PR/releases | Архив — нерепрезентативная выборка «самых трудных» | Полный список заданий 2025 и их домены |
| Ни одно задание не требует обучения с нуля или fine-tuning | В LMML-стиле важнее распознать трюк, чем обучить модель | Были ли в 2025 «честные» ML-задания с метрикой |
| Scoring 100/50/25 только в Australia; Paraguay — pass/fail; Morocco — не указан | Частичные очки — свойство отдельных задач | Правила платформы, штрафы, лимиты сдач |
| T5 gated, SimpleCNN `trust_remote_code`, оба в safetensors; код CNN безобиден на 13.11.2026 | Для анализа весов достаточно `safetensors` + `numpy` | Как спрятана CNN-часть флага (в разборе нет) |
| Противоречие «last parameter» vs `next(parameters())` | Скрипт верен, текст ошибочен | — (не проверено запуском) |
| Статья Australia мертва; Drive Paraguay → 404 анонимно | Артефакты 2025 частично невоспроизводимы | Будут ли задания 2026 раздаваться офлайн или через платформу |
| Публичные анонсы 2026: 6 доменов + Antarctica, 1–5 человек, гибрид, 14–15.11.2026 | Mentor Guideline в части доменов согласуется с публичными данными | Правила LLM/агентов, облака, обмена кодом, параллельный доступ к континентам |

---

## 9. Рекомендации к LMML 2.0 (с поправкой на выборку из 3 заданий)

1. **Триаж за 5 минут на задачу:** прочитать сюжет и выписать каждую «странную» фразу как гипотезу; понять, что сдаётся (флаг/слово/метрика), и есть ли частичные очки. **Сначала** проверить zero-training-гипотезы (артефакт, кодировка, утечка, метаданные), и только потом обучать.
2. **Бюджет обучения:** не начинать обучение дольше ~15–20 минут, пока нет подтверждения, что ответ от него зависит. В трёх заданиях 2025 ни одно не требовало GPU.
3. **Арсенал «распознать и декодировать»:** base64/32/85, hex, binary, rot-N, XOR с коротким ключом, Morse, ASCII-коды в числах, LSB в float/uint8, порядок битов (big/little), UTF-8 vs Latin-1.
4. **Инспекция моделей без выполнения кода:** `safetensors.safe_open` → имена, dtype, shape, статистики, доля единиц в LSB по тензорам; tied weights; аномальные тензоры и метаданные. Для `.bin`/`.pt` — `torch.load(weights_only=True)` или статический сканер pickle; никакого `trust_remote_code` без чтения кода и пина `revision`.
5. **Классический DS-набор на готовом шаблоне:** сортировка, сглаживание, FFT, find_peaks, базисные регрессии, быстрый бустинг-бейзлайн — для Европы и «сигнальных» задач.
6. **LLM-задачи (Африка/NLP/Security):** вести лог диалога; пробовать смену языка/кодировки/роли/формата; распознавать кодировки в ответах; помнить, что «защита» может быть протоколом, а не отказом.
7. **Безопасность и изоляция:** отдельный venv или контейнер; без личных токенов в окружении; HF-токен только read-only; заранее принять gated-условия, если модели будут объявлены.
8. **Командная логистика:** один человек ведёт таблицу «задача → гипотеза → статус → время → очки» и сдаёт частичные ответы, если правила это позволяют.

Конкретный план — в [`lmml-2026-prep-plan.md`](./lmml-2026-prep-plan.md).

---

## 10. Критические вопросы организаторам

1. Доступны ли все континенты **параллельно** с начала или открываются волнами/по прогрессу («CTF-style tasks in waves» на сайте)? Что нужно для доступа к «Antarctica»?
2. Разрешены ли внешние **LLM/API** (ChatGPT, Claude и т. п.) и **AI coding agents**? Можно ли передавать им тексты заданий и артефакты?
3. Разрешены ли **облачные GPU** (Colab, Kaggle, собственные)? Предоставляются ли вычисления организаторами?
4. Разрешены ли **предобученные модели** и внешние датасеты? Есть ли ограничения по лицензиям?
5. **Scoring:** фиксированные очки или динамические (зависят от числа решивших)? Есть ли частичные баллы и где? Штрафы за неверные сдачи? Лимит попыток? Влияет ли время сдачи на очки или тай-брейк?
6. **Подсказки за очки:** сколько стоят, как влияют на итог, видны ли другим?
7. **Формат флага** и нормализация (регистр, пробелы, обёртка вида `XXX{...}`)? Можно ли сдавать части составного флага?
8. **Размер команды** (на unde.io — 1–5) и правила для онлайн- и офлайн-участников: одинаковые ли задания и зачёт?
9. **Интернет** на площадке, ограничения по скорости; будут ли артефакты локально (USB/зеркало) на случай проблем с HF/Drive?
10. **Обмен кодом** между командами, публикация решений во время соревнования, общение с менторами — что допустимо?
11. **Evaluators:** публичная или скрытая проверка; есть ли задания с метриками и скрытым тестом; можно ли вызывать evaluator многократно?
12. Есть ли rate-limit на LLM-эндпоинтах заданий и логируются ли запросы?
13. Как фиксируются результаты при сбое платформы и действует ли апелляция?

---

## Ссылки

Все ссылки на файлы закреплены за SHA `f92ba23df8706a7adc3d14770bb139e0c5633596`:

- README репозитория: <https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/README.md#L3>
- Australia README: <https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/README.md>
- Australia script: <https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/australia-neural-cypher/extract_flag.py>
- Morocco README: <https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/morocco-he-never-understands-me/README.md>
- Paraguay README: <https://github.com/SigmoidAI/lmml-solutions/blob/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/README.md>
- Paraguay assets: <https://github.com/SigmoidAI/lmml-solutions/tree/f92ba23df8706a7adc3d14770bb139e0c5633596/challenges/paraguay-you-cant-stop-me/assets>
- HF: <https://huggingface.co/CezarCalin/hidden-flag-model> (gated, HF sha `c0e57e5c361527e510dbe0804b6be885e082fbd8`), <https://huggingface.co/CezarCalin/simple-cnn> (HF sha `5944e8d1b95022d7c9470d774bf2d106fc0b261c`)
- Внешний контекст: <https://lmml.sigmoidai.org/> , <https://lmml.sigmoidai.org/2025.html> , <https://unde.io/events/1908-learning-marathon-of-machine-learning-2-0> , <https://unde.io/events/1397-learning-marathon-of-machine-learning> , <https://fcim.utm.md/en/news/remarkable-performance-of-the-fcim-team-falcons-at-the-machine-learning-marathon-organized-by-sigmoid/>
