--- 
title: 'Transkribieren von Audio-Files mit whisper in Python'
description: ''
pubDate: 'Jul 08 2022'
heroImage: '/picography-laptop-notepad-desk-1.jpg'
---

<!--toc:start-->
- [Installation](#installation)
- [Erstes simples Script](#erstes-simples-script)
<!--toc:end-->

## Installation

Zunächst brauchen wir:
- [Python](https://www.python.org/downloads/)
- [FFmpeg](https://ffmpeg.org/download.html)

Anschließend installieren wir [whisper](https://github.com/openai/whisper) mit:
```bash
pip install -U openai-whisper
```

> *Mit [`pipenv`](https://realpython.com/pipenv-guide/) oder [`poetry`](https://python-poetry.org/) geht's noch eleganter ...*

Mehr findest du auf der [GitHub-Page von whisper](https://github.com/openai/whisper) ...


## Erstes simples Script

Zunächst laden wir mal ein Modell und transkribieren:
```python
import whisper

model = whisper.load_model("large")
result = model.transcribe('audio.mp3', language="de", fp16=False, verbose=True)
```
Hier haben wir die Wahl zwischen mehreren Modellgrößen:
|  Size  | Parameters | English-only model | Multilingual model | Required VRAM | Relative speed |
|:------:|:----------:|:------------------:|:------------------:|:-------------:|:--------------:|
|  tiny  |    39 M    |     `tiny.en`      |       `tiny`       |     ~1 GB     |      ~32x      |
|  base  |    74 M    |     `base.en`      |       `base`       |     ~1 GB     |      ~16x      |
| small  |   244 M    |     `small.en`     |      `small`       |     ~2 GB     |      ~6x       |
| medium |   769 M    |    `medium.en`     |      `medium`      |     ~5 GB     |      ~2x       |
| large  |   1550 M   |        N/A         |      `large`       |    ~10 GB     |       1x       |



