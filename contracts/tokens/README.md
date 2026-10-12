# Контракт `tokens` 1.0.0: токены стиля

**Владелец:** верстка (`cm-design/tokens/style.yaml`, `map.yaml`). **Читает:** картограф (стили карт), генератор схем, позже расчетчик (оформление таблиц расчетов).

Токены — единственный источник цветов, толщин линий, шрифтов и кеглей для всего, что попадает на лист: слайдов, карт, схем. Картограф читает файлы верстки из ее репозитория и не копирует значения к себе. Смена токена меняет слайды и карты одновременно без правок кода.

## Что верстка гарантирует

Ключи, которые картограф читает; имена — по факту в файлах верстки (подтверждено верстальщиком 11.10.2026). `schema.json` проверяет пару файлов `{style: <style.yaml>, map: <map.yaml>}`, прочитанных как YAML и приведенных к модели JSON (ключи — строки). Длины — мм на листе А3, кегли — pt.

| Ключ | Что | Пример использования картографом |
|---|---|---|
| `style.yaml: colors.{text, accent, rule, background, secondary, placeholder}` | роли цвета слайда, RGB без решетки | данные и подписи (`text`), граница территории (`accent`), плашки (`background`), строка источника (`secondary`) |
| `map.yaml: palette.*` | палитра карт: `block`, `green`, `water`, `building`, `road`, зоны ограничений и ПЗЗ `zone_*` | заливки и обводки слоев; цвет зоны ПЗЗ — `map.yaml: zoning.categories.*.fill` |
| `style.yaml: font.faces.{light, regular, medium, semibold}`, `font.files` | имя семейства по начертанию (для PowerPoint и QGIS) и файлы TTF | подписи объектов, легенда, масштаб |
| `style.yaml: sizes` | шкала кеглей, pt; других нет | кегли на карте — `map.yaml: text.styles.*.{size, weight, color}` (7, 8, 10 из шкалы); кегли слайда — `text.yaml: roles` |
| `map.yaml: line_types`, `min_line_width`, `layers.<ключ>.stroke.width`, `layers.<ключ>.hatch.width` | типы линий (штрих и пробел, мм), самая тонкая линия, толщины линий слоя, мм; у схем — `diagram.yaml: lines.*.width_pt` | границы, дороги, зоны ограничений — уже внутри стилей `design/qgis/<ключ>.qml` |
| `map.yaml: labels.defaults.<роль>.{text, style, field, auto}` | правило подписи по умолчанию: кварталы `blocks` — номер, `label_major`, только по запросу (`auto: false`); участки `parcels` — номер участка, `label_major`; зоны `zoning` — индекс зоны, `label_minor` (cm-design Д-065) | картограф отдает `label` в overlay по правилу; `label` и `label_style` описания карты — переопределение поля и стиля; справочная реализация — `cm_deck/qgis.py: label_rule` |
| `map.yaml: layers.<ключ>.{title, level, geometry}` | слои стандарта: ключ — имя стиля `.qml`, строка легенды, уровень, геометрия | слой геопакета → стиль (см. «Расхождения» ниже) |
| `style.yaml: unit.{pt, mm}`, `margins.{left_u, top_u}`, `grid.{columns, column_u, gutter_u}` | единица сетки (14 pt = 4,94 мм), поля и колонки в единицах сетки | отступы внутри карты уже в мм: `map.yaml: legend.inside`, `scale_bar.inset` |
| `style.yaml: number_format.{thousands, decimal}` | пробел в разрядах, запятая | цифры масштабной линейки и подписей |
| `map.yaml: places.<блок>.image` | ширина и высота карты, мм, по блоку места (`landscape`, `square_large`…); те же блоки и размеры, что `image.yaml: places.<блок>.size` | рендер карты в размер места; `size_mm` контракта `image` |
| `map.yaml: export.{format, dpi}` | PNG, 300 dpi; легенда — `export.legend`: SVG, текст кривыми | экспорт карты и легенды |

Стили и компоновки QGIS (`design/qgis/<слой>.qml`, `map_<блок>.qpt`, `legend_<ширина>.qpt`, `album_a3.qpt`) генерирует верстка из токенов: `python -m cm_deck.qgis`. Правила словами — `cm-design/MAPS.md`.

## Что запрещено

- Любой цвет, толщина или кегль в коде картографа, не взятый из токенов.
- Собственные файлы стилей с числами: `.qml` и `.qpt` генерирует верстка из токенов (этап 6 сделан).
- Правка токенов картографом. Предложение изменить токен — вопрос верстальщику через доклад.

## confirm — ответ верстки, 11.10.2026

1. Ключи — таблица выше. Чего нет и чем заменено: `colors.lines` → `colors.rule` (единственная линия слайда — под строкой таблицы); `colors.greys[]` → `colors.secondary` и `colors.placeholder`; `colors.map.*` → `map.yaml: palette`; `fonts.family`, `fonts.weights` → `font.faces`, `font.files`; `type.scale` → `sizes` (назначение — `text.yaml: roles`, на карте — `map.yaml: text.styles`); `lines.widths` → в `style.yaml` толщин нет: `map.yaml: line_types`, `min_line_width`, толщины в `layers`, у схем `diagram.yaml: lines`; `grid.unit_mm`, `grid.margin_mm` → `unit.mm`, `margins.left_u`, `margins.top_u` (в единицах сетки).
2. `layout-matrix.yaml` адресует места в колонках и единицах (`grid: {col, span, row_u, rows_u}`), не в мм. Размер в мм — по имени блока: `image.yaml: places.<блок>.size`, `map.yaml: places.<блок>.image`; блок картограф берет из описания карты (`place`).
3. `colors.map.*`: сделано — `map.yaml: palette`, этап 6, 11.10.2026; `MAPS.md` готов.

Расхождение с `geopackage` (картографу): `map.yaml` называет слой геопакета ключом слоя стандарта (`territory`, `road_main`, `point_1`, `reach_walk`, `zoning`), контракт `geopackage` — своим перечнем (`boundary_project`, `roads`, `poi_*`, `isochrones`, `zones_pzz`). Нужна таблица «слой геопакета → ключ стиля» в одном из двух контрактов.

## Изменения

1.0.0 — 11.10.2026 — первый черновик.
1.0.0 — 11.10.2026 — ключи подтверждены верстальщиком (имена по факту), `schema.json` и `example.yaml`.
