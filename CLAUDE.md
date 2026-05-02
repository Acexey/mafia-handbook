# Mafia Handbook — guidance for Claude

GitBook-published handbook on classic mafia (sport mafia) — strategy reference for live games.

## Structure

Top-level navigation lives in `SUMMARY.md`. Pages are nested under their parent README:

```
README.md                      главная: 2 карточки → voting, shooting
voting/README.md               2 карточки → divisions, free-voting
voting/divisions/README.md     7 карточек → 10/9/8/7/6/5/4-players
voting/divisions/N-players/    список статей в README + сами .md файлы
voting/free-voting/            пусто пока
shooting/                      пусто пока
assets/                        все картинки и пиктограммы
```

When adding a new article, update both:
- the parent `README.md` (add it to the article list)
- `SUMMARY.md` (so it appears in left nav)

## Card grid

GitBook карточки — это `<table data-view="cards">` с тремя колонками: заголовок, обложка (`data-card-cover`), таргет (`data-card-target`). **Без колонки описания** — пользователь явно попросил убрать «мелкие неинформативные подписки». На пустых страницах не оставлять placeholder-текст («Пока пусто», hint-блоки с пересказом названия) — только заголовок.

Пример из `voting/README.md`:

```markdown
<table data-view="cards">
  <thead><tr><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead>
  <tbody>
    <tr><td><strong>Деления</strong></td><td><a href="../assets/axe.gif">axe.gif</a></td><td><a href="divisions/README.md">divisions</a></td></tr>
  </tbody>
</table>
```

## Card cover spec (всегда применять)

Любая обложка карточки (PNG или анимированная GIF) проходит обработку Pillow и приводится к канону:

- Холст: **1200×675** (16:9)
- Фон: **#F0F0F0** (светло-серый, RGB `(240, 240, 240)`)
- Иконка: вписать в **220×220** бокс по центру, прозрачные края
- Источник обычно 640×640 outline-стиль с белым фоном — chroma-key с порогом **230** превращает белое в прозрачное

GitBook free tier не даёт менять CSS body карточки, поэтому серая «подложка» достигается через сам файл-обложку.

### Pipeline (Pillow)

```python
from PIL import Image, ImageSequence
import os

SRC = '/path/to/asset.gif'  # или .png
src = Image.open(SRC)

CANVAS_W, CANVAS_H = 1200, 675
ICON_BOX = 220
BG = (240, 240, 240)
THRESHOLD = 230

frames, durations = [], []
for frame in ImageSequence.Iterator(src):
    rgba = frame.convert('RGBA')
    w, h = rgba.size
    scale = ICON_BOX / max(w, h)
    nw, nh = int(w*scale), int(h*scale)
    icon = rgba.resize((nw, nh), Image.LANCZOS)
    pixels = icon.load()
    for x in range(nw):
        for y in range(nh):
            r, g, b, _ = pixels[x, y]
            if r >= THRESHOLD and g >= THRESHOLD and b >= THRESHOLD:
                pixels[x, y] = (0, 0, 0, 0)
    canvas = Image.new('RGB', (CANVAS_W, CANVAS_H), BG)
    canvas.paste(icon, ((CANVAS_W-nw)//2, (CANVAS_H-nh)//2), icon)
    frames.append(canvas.convert('P', palette=Image.ADAPTIVE, colors=64))
    durations.append(frame.info.get('duration', 100))

if len(frames) > 1:
    frames[0].save(SRC, save_all=True, append_images=frames[1:],
                   duration=durations, loop=0, optimize=True, disposal=2)
else:
    frames[0].save(SRC, optimize=True)
```

Для статичных PNG — то же самое, но без `ImageSequence` и без save_all (один кадр, без чрома-кей если источник уже прозрачный).

**Важно:** обработка перезаписывает файл-источник в `assets/`. Если боишься потерять оригинал — сохранить копию заранее.

## Стилистика иконок

Текущий набор (в `assets/`) — outline-style с бирюзовыми акцентами, источник gif.ski / Flaticon-подобный. При добавлении новой пиктограммы ищи в том же стиле, иначе будет визуальный рассинхрон.

## Существующие статьи

- `voting/divisions/9-players/checked-red-killed-sheriff.md` — разбор «полная власть на 9, деление в сторону», использует Mermaid и SVG.
