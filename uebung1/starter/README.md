**Team:** Batuhan Bayraktar (313356), Alejandro Parra Gomez (313451)
**Modul:** Cloud-Native AI WS26/27 · Prof. Dr. Pascal Laube · HTWG Konstanz

## Teil 1
1. Welche Container laufen, und wie findet die Anwendung das Modell?

in compose.yaml sehen wir wie "app" ausgeführt wird in dieser wird /app gebaut und anhand von .env (enviromentvariables) festgelegt welches modell ausgeführt. Hier wird der container ollama ausgeführt.
In llm.py wird der openAI client gebaut anhand von der BASE_URL, dem API_KEY und dem MODEL
 Dies wird auch durch den aufuf wie bei zb: docker compose run --rm app hallo.llm.py bestimmt.

2.-"Wie melde ich mich zu einer Prüfung an?"
Time to First Token: 9225 ms
Gesamtdauer:         23259 ms

-"Was kann ich in der Mensa essen heute"
Time to First Token: 420 ms
Gesamtdauer:         19992 ms

-"Was kannst du"
Time to First Token: 280 ms
Gesamtdauer:         12929 ms

---

## Teil 2.

-"Wie melde ich mich zu einer Prüfung an?"
Time to First Token: 2289 ms
Gesamtdauer:         3022 ms

-"Was kann ich in der Mensa essen heute"
Time to First Token: 129 ms
Gesamtdauer:         3205 ms

-"Was kannst du"
Time to First Token: 210 ms
Gesamtdauer:         1496 ms

| Frage | Modell | TTFT(ms) | Gesamt(ms)| Antwort brauchen|
| --- | --- | --- | --- | --- |
| „Ist die HTWG Konstanz eine private oder eine staatliche Hochschule?“ | llama3.2:3b | 190 | 524 | nein |
| „Ist die HTWG Konstanz eine private oder eine staatliche Hochschule?“ | qwen2.5:0.5b | 6963 | 9588 | ja |

---

## Teil 3.

1. -Temp 0: 1 Antwort 5x
-Temp 0,7: 4 verschiedene, eine doppelt
-Temp 1,2: 5 verschiedene

2. es liefert dieselbe, hätte es dies nicht getan dann könnte das an dem Prompt, der Hardware oder nicht genug caching liegen.

3. -bei Temp 0 gibt es nicht direkt einen Fehler sondern es wurde nur anders formuliert und es ist immeroch 1 Satz

-bei Temp 0,7 wurde es einmal umformuliert und einmal
 etwas komisch geantwortet was aber eigentlich kein direkter Fehler ist (HTWG in Konstanz war die Antwort statt Die HTWG Konstanz)

-eine abgehackte antwort, falsche information (hochschule und nicht UNI und es ist auch keine private Schule)
-einmal konstanz zu viel geschrieben
-HTWG namen flasch aufgeschrieben

1.2
Deutsch    63 Tokens für 21 Wörter
Englisch   49 Tokens für 21 Wörter
Deutsch braucht 14 Tokens mehr (+29%).

2.2
Wie lange braucht die HTWG um meine Note ins Prüfungsportal zu schreiben
How long does it take for the HTWG to enter my grade into the exam portal?

Deutsch    43 Tokens für 10 Wörter
Englisch   43 Tokens für 14 Wörter

Deutsch braucht 0 Tokens mehr (+0%).

---

## Teil 4

1. -llama hat 7 von 8 richtig
   -qwen hat 3 von 8 richtig
2. Anliegen und Frist sind beide falsch
3. der datentyp wird nicht geprüft
4. er lässt sich nicht umleneken und bleibt beim thema
5. das response format ist wichtig um eine schöne formatierte ausgabe zu machen,bei temp 0 ist die antwort gleich, deswegen kann man sich nicht darauf verlassen

---

## Reflexion

1. man muss nicht immer neu bauen, saubere trennung von code und config
2.0.0, weil man möglichst die selben antworten will und bei 1.3 oder 0,7 wären das zu viele antworten
3. es bietet eine schöne formatierung, und Struktur
4. lokal:   - Datenschutz
            - Kosten
  Cloud:    - Qualität
            - hardware ist kein Problem