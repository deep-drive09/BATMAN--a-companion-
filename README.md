# 🦇 Batman A.V.C. — Autonomous Virtual Companion

A voice-controlled virtual companion built in **Java Swing**, backed by a local **Python AI server**. You talk to Batman through your microphone, and he replies in a deep AI-generated voice. His stats (stamina, batarang ammo and Gotham's threat level) are saved in a **SQLite** database.

> Java subject micro-project · Fan project, not affiliated with DC or Warner Bros.

---

## Features

- 🎙 **Voice conversation**: record your voice, which is transcribed offline and answered in character by an AI Batman
- 🔊 **Neural voice replies**: responses are spoken aloud in a deep cinematic voice
- 🎭 **Reactive avatar**: animated GIFs change with Batman's mood (for example, he gets angry if he mentions the Joker)
- ⚔ **Actions**: Patrol, Sleep and Eat change stamina, ammo and threat level
- 🌃 **Living world**: a background thread slowly raises Gotham's threat level and drains stamina over time
- 💾 **Persistent progress**: stats are saved to SQLite after every action and reloaded on startup
- 🖥 **Tactical terminal UI**: custom Swing dashboard with smooth bicubic-scaled GIFs

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | Java Swing, AWT (Graphics2D) |
| Client logic | Core Java: `HttpURLConnection`, `javax.sound.sampled` |
| Database | SQLite via JDBC (`sqlite-jdbc.jar`) |
| AI server | Python, Flask |
| Speech-to-text | `faster-whisper` (offline, `small.en` model) |
| AI chat | `g4f` client (`gpt-4o-mini`) |
| Text-to-speech | `edge-tts` (`en-US-ChristopherNeural`) + `playsound` |

---

## Architecture

Local client–server model on `127.0.0.1:5000`:

```
Java Swing app (client)                     Python Flask server (AI brain)
──────────────────────                      ──────────────────────────────
🎙 Record mic → audio.wav ──POST /transcribe──► faster-whisper → text
                          ◄──── JSON text ────┘
   Send text ─────────────POST /chat─────────► g4f (gpt-4o-mini, "You are Batman")
                          ◄── Batman's reply ─┘
   Send reply ────────────POST /tts──────────► edge-tts → MP3 → plays aloud
   Update avatar GIF

   Actions / game loop ──► DatabaseManager ──► batman_stats.db (SQLite)
```

---

## Database

File: `batman_stats.db` (created automatically)

| Column | Default |
|---|---|
| `id` | 1 |
| `stamina` | 100 |
| `batarang_ammo` | 100 |
| `threat_level` | 0 |

- **On startup:** `CREATE TABLE IF NOT EXISTS stats`, then inserts the default row if the table is empty, then `SELECT` to load the stats.
- **On every action:** `UPDATE stats ... WHERE id = 1` saves the current values.

---

## Project Structure

| File | Purpose |
|---|---|
| `BatmanPet.java` | Main entry point. Builds the Swing GUI, handles buttons and actions, picks GIFs by mood |
| `DatabaseManager.java` | All SQLite logic: `initDatabase()`, `loadStats()`, `saveStats()` |
| `HeroStats.java` | POJO holding stamina, ammo and threat, kept within 0–100 |
| `WhisperAudioRecorder.java` | Records the microphone to a WAV file (16 kHz, 16-bit, mono) and sends it for transcription |
| `LLMNetworkClient.java` | Sends prompts to `/chat`, parses the JSON reply, decodes Unicode escapes |
| `GameLoopThread.java` | Background thread that raises the threat level and drains stamina every 10–15 seconds |
| `local_whisper_server.py` | Flask server with `/transcribe`, `/chat` and `/tts` endpoints |

---

## How to Run

### 1. Start the AI server
```bash
pip install flask faster-whisper g4f edge-tts playsound
python local_whisper_server.py
```
Wait until the models load and the server is listening on port 5000. The first run downloads the Whisper model.

### 2. Run the Java app
Put `sqlite-jdbc.jar` in the project folder, then:
```bash
javac -cp ".;sqlite-jdbc.jar" *.java
java  -cp ".;sqlite-jdbc.jar" BatmanPet
```
(On macOS or Linux, use `:` instead of `;` in the classpath.)

**Requirements:** Java 8+, Python 3.9+, a microphone, and an internet connection for the chat and voice features.

---

## Limitations

- The chat feature relies on `g4f`, an unofficial free wrapper, so it can stop working without warning when providers change.
- The Python server must be running before you can use voice features.
- One save slot only (`id = 1`).

## Future Scope

- Switch to an official LLM API, or a local model such as Ollama, for reliability
- Multiple save profiles
- Mission or quest system tied to the threat level
- Packaged single-click launcher for the Java app and Python server
