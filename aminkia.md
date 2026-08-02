# AMINKIA — YouTube AI Toolkit Skill

## Trigger
Quando l'utente scrive `/aminkia` o `aminkia` avvia questa skill dall'inizio.

---

## STEP 1 — Banner di apertura

Stampa direttamente questo testo (NON usare bash, scrivilo tu come output):

```
 █████╗ ███╗   ███╗██╗███╗   ██╗██╗  ██╗██╗ █████╗ 
██╔══██╗████╗ ████║██║████╗  ██║██║ ██╔╝██║██╔══██╗
███████║██╔████╔██║██║██╔██╗ ██║█████╔╝ ██║███████║
██╔══██║██║╚██╔╝██║██║██║╚██╗██║██╔═██╗ ██║██╔══██║
██║  ██║██║ ╚═╝ ██║██║██║ ╚████║██║  ██╗██║██║  ██║
╚═╝  ╚═╝╚═╝     ╚═╝╚═╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝╚═╝  ╚═╝

  Turn any video into content, insights & AI prompts
```

---

## STEP 2 — Lingua output

Chiedi all'utente:

```
🌐 In which language do you want the output?
(e.g. English, Italiano, Deutsch, Français, Español...)
Press Enter to auto-detect from the video language.
```

Salva la lingua scelta come OUTPUT_LANGUAGE. Usala per TUTTI gli output delle funzioni F1-F10.
Se l'utente preme Invio o non risponde → usa la lingua del transcript (auto-detect).

---

## STEP 3 — Scegli sorgente

Chiedi all'utente:

```
What do you want to analyze?

  1. 🎬  YouTube URL
  2. 📁  Video on my computer

Type 1 or 2:
```

Attendi la risposta e vai al branch corretto.

---

## STEP 3A — Sorgente: URL YouTube

Chiedi: `Incolla l'URL del video YouTube:`

Una volta ricevuto l'URL:

### 3A.1 — Verifica dipendenze

```bash
python3 -c "from youtube_transcript_api import YouTubeTranscriptApi; print('OK')" 2>/dev/null || echo "MISSING"
```

Se output è `MISSING`:
```bash
pip install youtube-transcript-api -q
```

### 3A.2 — Estrai il video ID dall'URL

L'ID è la parte dopo `v=` o dopo `youtu.be/`. Esempio:
- `https://www.youtube.com/watch?v=dQw4w9WgXcQ` → ID = `dQw4w9WgXcQ`
- `https://youtu.be/dQw4w9WgXcQ` → ID = `dQw4w9WgXcQ`

### 3A.3 — Scarica il transcript

```bash
python3 << 'EOF'
from youtube_transcript_api import YouTubeTranscriptApi

video_id = "VIDEO_ID_QUI"

try:
    ytt_api = YouTubeTranscriptApi()
    transcript_list = ytt_api.list(video_id)
    transcript = transcript_list.find_transcript(['it', 'en', 'en-US'])
    data = transcript.fetch()

    with open('/tmp/aminkia_transcript.txt', 'w') as f:
        for entry in data:
            minutes = int(entry.start // 60)
            seconds = int(entry.start % 60)
            f.write(f"[{minutes:02d}:{seconds:02d}] {entry.text}\n")

    with open('/tmp/aminkia_text.txt', 'w') as f:
        f.write(' '.join([e.text for e in data]))

    words = len(' '.join([e.text for e in data]).split())
    print(f"OK:{words}")

except Exception as e:
    print(f"ERROR:{e}")
EOF
```

Sostituisci `VIDEO_ID_QUI` con l'ID estratto.

Se output inizia con `ERROR:` → mostra l'errore all'utente e chiedi un altro URL.

Se output inizia con `OK:N` → mostra:
```
✅ Transcript caricato — ~N parole
```

Leggi il transcript: `Read /tmp/aminkia_transcript.txt`

---

## STEP 3B — Sorgente: Video locale

Chiedi: `Incolla il percorso del video (es. C:\Users\Carmelo\Desktop\video.mp4):`

Converti il percorso Windows in WSL:
- `C:\Users\Carmelo\Desktop\video.mp4` → `/mnt/c/Users/Carmelo/Desktop/video.mp4`

### 3B.1 — Verifica Whisper

```bash
python3 -c "import whisper; print('OK')" 2>/dev/null || echo "MISSING"
```

Se `MISSING`, chiedi all'utente:
```
Whisper non è installato. È necessario per trascrivere video locali.
Vuoi installarlo ora? (richiede ~1GB di download la prima volta)

  1. Sì, installa
  2. No, usa un URL YouTube invece
```

Se sì:
```bash
pip install openai-whisper -q
```

### 3B.2 — Trascrivi il video

```bash
python3 << 'EOF'
import whisper, os

video_path = "PERCORSO_VIDEO_QUI"
model = whisper.load_model("base")

print("⏳ Trascrizione in corso (può richiedere qualche minuto)...")
result = model.transcribe(video_path, language=None, verbose=False)

# Salva con timestamp
with open('/tmp/aminkia_transcript.txt', 'w') as f:
    for seg in result['segments']:
        minutes = int(seg['start'] // 60)
        seconds = int(seg['start'] % 60)
        f.write(f"[{minutes:02d}:{seconds:02d}] {seg['text'].strip()}\n")

# Salva testo puro
with open('/tmp/aminkia_text.txt', 'w') as f:
    f.write(result['text'])

words = len(result['text'].split())
print(f"OK:{words}")
EOF
```

Sostituisci `PERCORSO_VIDEO_QUI` con il percorso WSL del video.

Se output `OK:N` → mostra:
```
✅ Trascrizione completata — ~N parole
```

---

## STEP 4 — Menu funzioni

Mostra il menu:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  COSA VUOI FARE CON QUESTO VIDEO?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  📘 LEARN
   1  ⚡  Sintetizza        TL;DR + punti chiave + capitoli
   2  📝  Appunti           Bullet points strutturati
   3  🎯  Quiz              8 domande con risposta

  ✍️  CREATE
   4  💼  LinkedIn Post     Hook + insight + CTA professionale
   5  𝕏   Thread X          7 tweet collegati
   6  📧  Newsletter        Blocco pronto da incollare
   7  🎙️  Script Replica    Riscrive il video come script tuo

  🎬 AI VIDEO
   8  🤖  Higgsfield Copy   Prompt per generare video AI
   9  🔥  Hook Generator    5 hook virali per TikTok/Reels
  10  ✂️  Clip Finder       Top 5 momenti clippabili + timestamp

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Scrivi il numero (1-10) o 'tutte' per eseguire tutto:
```

---

## STEP 5 — Esegui la funzione scelta

Leggi il contenuto: `Read /tmp/aminkia_text.txt` e `Read /tmp/aminkia_transcript.txt`

Usa SEMPRE il transcript con timestamp per le funzioni F10 (Clip Finder) e F8 (Higgsfield Copy).
Per le altre funzioni usa il testo puro.

### F1 — Sintetizza
Analizza il transcript e produci:

**TL;DR** (max 3 righe)
[sintesi essenziale]

**5 Punti Chiave**
1. ...
2. ...
3. ...
4. ...
5. ...

**Struttura del video**
- [00:00] Intro: ...
- [XX:XX] ...
(ricava i tempi dal transcript con timestamp)

---

### F2 — Appunti
Trasforma il transcript in appunti gerarchici con titoli in grassetto, bullet points per ogni concetto, sottobullet per i dettagli. Elimina ripetizioni e linguaggio verboso del parlato. Pronti da copiare e studiare.

---

### F3 — Quiz
Genera 8 domande con risposta. Varia: comprensione, applicazione, analisi. Formato:

**Q1: [domanda]**
R: [risposta]

---

### F4 — LinkedIn Post
- Riga 1: hook che ferma lo scroll
- Corpo: 3-5 insight con spazi bianchi tra i punti
- Chiusura: domanda o CTA
- 3-5 hashtag rilevanti in fondo
- Max 1300 caratteri, tono professionale ma umano

---

### F5 — Thread X
7 tweet numerati (1/7, 2/7...). Hook potente al primo, CTA al settimo. Max 280 caratteri per tweet.

---

### F6 — Newsletter
Titolo + intro 2 righe + 5 punti con mini-spiegazione + take principale + CTA "Guarda il video".

---

### F7 — Script Replica
Riscrivi come script originale con tono naturale. Includi note di regia [pausa] [enfasi]. Struttura: Hook → Corpo → Chiusura con CTA.

---

### F8 — Higgsfield Copy
Genera 4 prompt in inglese ottimizzati per Higgsfield AI. Per ognuno:

**Prompt N: [titolo]**
```
[prompt completo: soggetto, stile visivo, mood, camera movement, lighting]
```
*Modello consigliato: [Seedance 2.0 / Soul Cinema / Nano Banana]*

---

### F9 — Hook Generator
5 hook virali per TikTok/Reels. Max 2 righe ciascuno. Varia: domanda, affermazione shock, promessa, controintuizione, storytelling. Per ognuno spiega in 1 riga perché funziona.

---

### F10 — Clip Finder
Identifica i 5 momenti più clippabili usando i timestamp del transcript.

**Clip N — [titolo]**
- ⏱ [MM:SS → MM:SS]
- 🎯 Perché è virale: [1 riga]
- 📝 Caption: "[testo pronto]"
- #️⃣ [#tag1 #tag2 #tag3]

---

### Opzione 'tutte'
Esegui F1, F4, F9, F10 in sequenza (le più utili). Separa ogni output con `━━━━━━━━━━`.

---

## STEP 6 — Dopo l'output

Dopo ogni output mostra SEMPRE il menu completo e poi chiedi:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  COSA VUOI FARE CON QUESTO VIDEO?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  📘 LEARN
   1  ⚡  Sintetizza        TL;DR + punti chiave + capitoli
   2  📝  Appunti           Bullet points strutturati
   3  🎯  Quiz              8 domande con risposta

  ✍️  CREATE
   4  💼  LinkedIn Post     Hook + insight + CTA professionale
   5  𝕏   Thread X          7 tweet collegati
   6  📧  Newsletter        Blocco pronto da incollare
   7  🎙️  Script Replica    Riscrive il video come script tuo

  🎬 AI VIDEO
   8  🤖  Higgsfield Copy   Prompt per generare video AI
   9  🔥  Hook Generator    5 hook virali per TikTok/Reels
  10  ✂️  Clip Finder       Top 5 momenti clippabili + timestamp

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Scrivi il numero (1-10) o 'no' per uscire:
```

Se risponde con un numero → torna al STEP 5.
Se risponde `no` → mostra:

Se risponde qualsiasi altra cosa → rimosta il menu completo:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  COSA VUOI FARE CON QUESTO VIDEO?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  📘 LEARN
   1  ⚡  Sintetizza        TL;DR + punti chiave + capitoli
   2  📝  Appunti           Bullet points strutturati
   3  🎯  Quiz              8 domande con risposta

  ✍️  CREATE
   4  💼  LinkedIn Post     Hook + insight + CTA professionale
   5  𝕏   Thread X          7 tweet collegati
   6  📧  Newsletter        Blocco pronto da incollare
   7  🎙️  Script Replica    Riscrive il video come script tuo

  🎬 AI VIDEO
   8  🤖  Higgsfield Copy   Prompt per generare video AI
   9  🔥  Hook Generator    5 hook virali per TikTok/Reels
  10  ✂️  Clip Finder       Top 5 momenti clippabili + timestamp

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Scrivi il numero (1-10) o 'no' per uscire:
```

Dopo ogni output mostra SEMPRE questo menu compatto prima di chiedere:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Grazie per aver usato AMINKIA 🎬
  github.com/tuousername/aminkia
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Note operative

- Il transcript viene salvato in `/tmp/` e rimane disponibile per tutta la sessione
- Se il video è molto lungo (>1 ora), usa solo i primi 15.000 caratteri del testo per l'elaborazione
- OUTPUT_LANGUAGE viene chiesta all'inizio e applicata a tutti gli output F1-F10, indipendentemente dalla lingua del transcript
- Se l'utente non specifica la lingua → auto-detect dalla lingua del transcript
