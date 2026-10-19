# Übung 1 · Das Modell als Baustein

**Team:** Batuhan Bayraktar (313356), Alejandro Parra Gomez (313451)
**Modul:** Cloud-Native AI WS26/27 · Prof. Dr. Pascal Laube · HTWG Konstanz

---

## Start

```bash
cp .env.example .env    # Standardwerte (lokales Modell über Ollama), nichts ändern nötig
docker compose up --build app
```

Weitere Skripte: `docker compose run --rm app sampling.py` usw.
Das Modell wird nur über `LLM_MODEL` in `.env` gewählt.

---

## Teil 1

### 1. Welche Container laufen, und wie findet die Anwendung das Modell?

In `compose.yaml` sehen wir wie `app` ausgeführt wird. In dieser wird `/app` gebaut und anhand von `.env` (Environment-Variablen) festgelegt, welches Modell ausgeführt wird. Hier wird der Container `ollama` ausgeführt. Zusätzlich läuft `modell-laden`, der einmalig die beiden Modelle herunterlädt und sich dann beendet.

In `llm.py` wird der OpenAI-Client gebaut anhand von der `BASE_URL` (`http://ollama:11434/v1`, `ollama` ist der Servicename im Compose-Netzwerk), dem `API_KEY` und dem `MODEL`.

Dies wird auch durch den Aufruf bestimmt, z. B.: `docker compose run --rm app hallo_llm.py`

### 2. Drei Fragen mit `llama3.2:3b`

| Frage | TTFT (ms) | Gesamt (ms) |
| --- | ---: | ---: |
| „Wie melde ich mich zu einer Prüfung an?“ | 9225 (enthält das Laden des Modells) | 23259 |
| „Was kann ich in der Mensa essen heute“ | 420 | 19992 |
| „Was kannst du“ | 280 | 12929 |

---

## Teil 2

### Dieselben drei Fragen mit `qwen2.5:0.5b`

| Frage | TTFT (ms) | Gesamt (ms) |
| --- | ---: | ---: |
| „Wie melde ich mich zu einer Prüfung an?“ | 2289 | 3022 |
| „Was kann ich in der Mensa essen heute“ | 129 | 3205 |
| „Was kannst du“ | 210 | 1496 |

### Vergleich bei der HTWG-Frage

| Frage | Modell | TTFT (ms) | Gesamt (ms) | Antwort brauchbar? |
| --- | --- | ---: | ---: | :---: |
| „Ist die HTWG Konstanz eine private oder eine staatliche Hochschule?“ | llama3.2:3b | 190 | 524 | nein |
| „Ist die HTWG Konstanz eine private oder eine staatliche Hochschule?“ | qwen2.5:0.5b | 6963 | 9588 | ja |

**Auffällig:** Das größere Modell `llama3.2:3b` war hier deutlich schneller (524 ms statt 9588 ms), hat die Frage aber nicht brauchbar beantwortet. Das kleine `qwen2.5:0.5b` war brauchbar. Größer heißt also nicht automatisch richtiger bei Fakten, und Geschwindigkeit sagt nichts über die Qualität. Die hohe TTFT von qwen (6963 ms) kommt vom Laden des Modells direkt nach dem Wechsel. Die HTWG ist staatlich, man muss solche Faktenantworten also immer prüfen.

---

## Teil 3

### Sampling

**1. Verschiedene Antworten je Temperature**

| Temperature | Ergebnis |
| ---: | --- |
| 0 | 1 Antwort, 5× dieselbe |
| 0,7 | 4 verschiedene, eine doppelt |
| 1,2 | 5 verschiedene |

**2. Ist Temperature 0 wirklich deterministisch?**

Es liefert dieselbe Antwort. Hätte es dies nicht getan, dann könnte das am Prompt oder an der Hardware liegen (Rundungsunterschiede bei Gleitkommazahlen, Batching).

**3. Faktenfrage**

- **Temp 0:** Es gibt nicht direkt einen Fehler, sondern es wurde nur anders formuliert und es ist immer noch 1 Satz.
- **Temp 0,7:** Es wurde einmal umformuliert und einmal etwas komisch geantwortet, was aber eigentlich kein direkter Fehler ist („HTWG in Konstanz“ war die Antwort statt „Die HTWG Konstanz“).
- **Höhere Werte:**
  - eine abgehackte Antwort
  - falsche Information („Hochschule“ und nicht „UNI“, und es ist auch keine private Schule)
  - einmal „Konstanz“ zu viel geschrieben
  - HTWG-Name falsch aufgeschrieben

**Ursache:** Formulierungen, doppelte Wörter und falsche Namen kommen erst bei höheren Temperaturen, das ist Sampling. Falsche Fakten wie „UNI“ oder „private Schule“ sind falsches Wissen aus den Trainingsdaten.

### Tokens

**1.1** Text aus der Vorlage

| Sprache | Tokens | Wörter |
| --- | ---: | ---: |
| Deutsch | 63 | 21 |
| Englisch | 49 | 21 |

Deutsch braucht 14 Tokens mehr (**+29 %**).

**1.2** Eigene Frage

> Wie lange braucht die HTWG um meine Note ins Prüfungsportal zu schreiben
> How long does it take for the HTWG to enter my grade into the exam portal?

| Sprache | Tokens | Wörter |
| --- | ---: | ---: |
| Deutsch | 43 | 10 |
| Englisch | 43 | 14 |

Deutsch braucht 0 Tokens mehr (**+0 %**).





---

## Teil 4

**1. Trefferquote**

| Modell | Anliegen richtig |
| --- | ---: |
| llama3.2:3b | 7 von 8 |
| qwen2.5:0.5b | 3 von 8 |

**2. Fehler**

Anliegen und Frist sind beide falsch.

**3. Was das Schema durchlässt**

Der Datentyp wird nicht geprüft. Das Schema (`str | None`) lässt jeden String zu, also auch den Text `"null"` statt `None` oder eine Frist, die nicht in der Mail steht. Es prüft nur die Form, nicht den Inhalt.

**4. Letzte Mail (Umlenkungsversuch)**

Er lässt sich nicht umlenken und bleibt beim Thema.

**5. Ohne `response_format`**

Das `response_format` ist wichtig, um eine schön formatierte Ausgabe zu machen. Bei Temp 0 ist die Antwort gleich, deswegen kann man sich nicht darauf verlassen.

Ein Retry bei Temp 0 bringt nichts, weil das Modell wieder dieselbe Antwort liefert. Man könnte beim Retry die Temperature erhöhen oder den Fehler mitgeben.

---

## Reflexion

**1. Warum steht das Modell in `.env` und nicht im Code?**
Man muss nicht immer neu bauen, saubere Trennung von Code und Config. Man kann so zwischen lokalem Modell und Cloud wechseln, ohne den Code zu ändern, und der API-Key steht nicht im Code.

**2. Welche Temperature in `einordnen.py` und warum?**
0.0, weil man möglichst die selben Antworten will und bei 1,2 oder 0,7 wären das zu viele Antworten.

**3. Was garantiert `response_format` mit Schema, und was nicht?**
Es garantiert die Form (gültiges JSON mit den richtigen Feldern und Typen), also eine schöne Formatierung und Struktur. Es garantiert nicht den Inhalt: das Anliegen kann trotzdem falsch sein, eine Frist kann erfunden sein oder `"null"` als Text drinstehen.

**4. Lokal oder Cloud?**

| Lokal | Cloud |
| --- | --- |
| Datenschutz | Qualität |
| Kosten | Hardware ist kein Problem |