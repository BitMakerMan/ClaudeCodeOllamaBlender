# 🚀 Blender + Ollama + Claude Code: Assistente 3D Locale e Gratuito

> Collega **Blender** a un modello AI **100% locale** usando Claude Code e il protocollo MCP.  
> Nessun costo. Nessun dato inviato al cloud. Tutto gira sul tuo hardware.

[![Repo](https://img.shields.io/badge/GitHub-ClaudeCodeOllamaBlender-181717?style=flat-square&logo=github)](https://github.com/BitMakerMan/ClaudeCodeOllamaBlender)
[![Launcher](https://img.shields.io/badge/Launcher-Craicek_CCOL-blueviolet?style=flat-square&logo=github)](https://github.com/BitMakerMan/Craicek-ClaudeCodeLauncherOllama)
![Blender](https://img.shields.io/badge/Blender-3.0%2B-orange?style=flat-square&logo=blender)
![Ollama](https://img.shields.io/badge/Ollama-local-blue?style=flat-square)
![Claude Code](https://img.shields.io/badge/Claude_Code-MCP-purple?style=flat-square)
![Testato su](https://img.shields.io/badge/Testato_su-Windows_11-0078D4?style=flat-square&logo=windows)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

---

## 📖 Cosa fa questa guida?

Questa configurazione unisce tre tecnologie potenti per creare un assistente 3D locale, privato e completamente gratuito:

| Tecnologia | Ruolo |
|---|---|
| 🎨 **Blender** | Software 3D che esegue i comandi ricevuti via socket su `127.0.0.1:9876` |
| 🧠 **Ollama** | Serve il modello AI in locale — testato con `gemma4:26b` |
| 🤖 **Claude Code** | CLI agente che orchestra i tool MCP |
| 🔌 **BlenderMCP** | Bridge tra Claude Code e Blender via Model Context Protocol |
| ⚡ **[Craicek CCOL](https://github.com/BitMakerMan/Craicek-ClaudeCodeLauncherOllama)** | Launcher che reindirizza Claude Code verso Ollama invece dei server Anthropic |

Il risultato: un assistente AI che **legge la tua scena 3D**, crea oggetti, applica materiali, scarica asset da Poly Haven ed esegue script Python (`bpy`), tutto localmente.

---

## 🛠️ Requisiti

- **[Blender](https://www.blender.org/download/)** — versione 3.0 o superiore
- **[Ollama](https://ollama.com/)** — con almeno un modello scaricato
- **[Node.js](https://nodejs.org/)** — LTS, necessario per Claude Code
- **[Claude Code](https://docs.anthropic.com/claude-code)** — CLI di Anthropic
- **[uv](https://github.com/astral-sh/uv)** — gestore pacchetti Python (necessario per `uvx blender-mcp`)

### Installazioni rapide

```bash
# Claude Code
npm install -g @anthropic-ai/claude-code

# uv su Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# uv su macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Modello AI consigliato

```bash
# Testato e consigliato (richiede ~20GB VRAM)
ollama pull gemma4:26b

# Alternative per hardware meno potente
ollama pull llama3.1:8b        # ~8GB VRAM
ollama pull qwen2.5-coder:7b   # ~8GB VRAM — ottimizzato per codice bpy
```

> 💡 `gemma4:26b` è il modello con cui questa configurazione è stata testata in produzione.

---

## ⚙️ Step 1 — Installa l'Add-on BlenderMCP

L'add-on apre un server socket in Blender che riceve i comandi dall'AI.

1. Scarica `addon.py` da: **[https://github.com/ahujasid/blender-mcp](https://github.com/ahujasid/blender-mcp)**
2. Apri Blender → **Edit → Preferences → Add-ons**
3. Clicca **Install...** → seleziona `addon.py` → attiva **"Interface: Blender MCP"**
4. Nella vista 3D, premi `N` → tab **BlenderMCP** → **Connect to Claude**

> ✅ Il server è attivo quando vedi `Listening on 127.0.0.1:9876` nel terminale di Blender.

---

## 🔌 Step 2 — Configura il server MCP

Hai due opzioni. Scegli in base al tuo flusso di lavoro:

---

### ✅ Opzione A — Configurazione Globale (consigliata per iniziare)

Esegui questo comando **una volta sola** da qualsiasi terminale:

```bash
claude mcp add blender uvx blender-mcp
```

**Cosa fa:** aggiunge la configurazione direttamente nel file globale `~/.claude.json` (su Windows: `C:\Users\<nome>\.claude.json`), nella sezione del tuo profilo utente. Il server MCP blender sarà disponibile **in tutti i tuoi progetti** Claude Code.

Verifica con:

```bash
claude mcp list
# Output atteso:
# blender: uvx blender-mcp
```

---

### 📁 Opzione B — Configurazione Locale al Progetto (consigliata per setup condivisi)

Se vuoi che la config MCP sia contenuta **solo nella cartella del progetto** (es. per includerla nel repo GitHub), crea manualmente un file `.mcp.json` nella root del progetto:

```json
{
  "mcpServers": {
    "blender": {
      "type": "stdio",
      "command": "uvx",
      "args": ["blender-mcp"],
      "env": {
        "BLENDER_HOST": "127.0.0.1",
        "BLENDER_PORT": "9876"
      }
    }
  }
}
```

**Quando usare l'Opzione B:**
- Vuoi committare la config nel repo così altri la ricevono con `git clone`
- Hai più progetti con configurazioni MCP diverse
- Vuoi isolare completamente l'ambiente di un progetto specifico

> ⚠️ Su Windows, `BLENDER_PORT` deve essere una **stringa** `"9876"`, non un intero `9876`.

> 💡 Le due opzioni sono compatibili: se esiste sia la config globale che quella locale, Claude Code **dà priorità a quella locale**.

---

## 🧠 Step 3 — Avvia l'AI Locale con Craicek CCOL

**[Craicek CCOL](https://github.com/BitMakerMan/Craicek-ClaudeCodeLauncherOllama)** è il launcher che intercetta le chiamate di Claude Code e le reindirizza verso Ollama, impostando internamente `ANTHROPIC_BASE_URL` al tuo server locale.

> 📥 Scarica il launcher da: **[github.com/BitMakerMan/Craicek-ClaudeCodeLauncherOllama](https://github.com/BitMakerMan/Craicek-ClaudeCodeLauncherOllama)**

### Struttura della cartella di progetto

```
CC_ARC2_Launcher/
├── Craicek_CCOL.bat            ← avvia da qui
├── Craicek_CCOL.ps1
├── Craicek_CCOL-config.json
├── blender_rules.md            ← system prompt (vedi Step 4)
└── .mcp.json                   ← config MCP locale (Opzione B)
```

### Configura `Craicek_CCOL-config.json`

```json
{
  "ollama_url": "http://localhost:11434",
  "keep_alive": "2h",
  "connection_timeout_sec": 5,
  "preload_timeout_sec": 3600,
  "proxmox_drop_cache_cmd": "echo 3 > /proc/sys/vm/drop_caches"
}
```

> 💡 Se Ollama gira su un'altra macchina della rete LAN, sostituisci `localhost` con l'IP remoto (es. `192.168.1.50`).

### Avvia

Fai doppio clic su **`Craicek_CCOL.bat`** → seleziona modello e modalità di elaborazione dal menu.

> ⚠️ Lancia Claude Code **sempre e solo** tramite `Craicek_CCOL.bat`. Avviarlo direttamente con `claude` userebbe i server Anthropic invece di Ollama.

---

## 📜 Step 4 — Crea il System Prompt (`blender_rules.md`)

I modelli locali necessitano di istruzioni esplicite sulle API di Blender.  
Crea il file `blender_rules.md` nella cartella del launcher con questo contenuto:

```markdown
Sei un AI Assistant specializzato in grafica 3D, connesso a Blender tramite BlenderMCP
(Model Context Protocol). Il tuo obiettivo è assistere l'utente nella creazione di scene,
modellazione, shading, illuminazione e automazione all'interno di Blender.

## Strumenti disponibili

Hai a disposizione strumenti per:
- Ispezionare la scena corrente e gli oggetti al suo interno.
- Creare, eliminare e modificare forme geometriche (mesh).
- Creare e applicare materiali e colori.
- Scaricare modelli, texture e sfondi HDRI tramite l'integrazione Poly Haven.
- Generare modelli 3D con AI tramite Hyper3D Rodin.
- Eseguire codice Python arbitrario in Blender tramite il tool `execute_blender_code`.

## Regole per execute_blender_code

- Importa sempre `bpy` all'inizio degli script.
- Mantieni il codice pulito, commentato e mirato al task richiesto.
- Assicurati di essere nel contesto corretto (es. 'OBJECT' mode) prima di operare sulle mesh.

## Prevenzione timeout e crash

- Se il task è complesso, spezzalo in step sequenziali (1. Geometria → 2. Materiali → 3. Luci).
- Avvisa l'utente di salvare il `.blend` prima di operazioni massicce o distruttive.

## Gestione asset e realismo

- Per atmosfere fotorealistiche, usa i tool Poly Haven per HDRI e texture PBR.
- Se un modello è complesso da fare proceduralmente, sfrutta Hyper3D o asset esterni.

## Gestione errori

Se un tool MCP fallisce, NON ripetere la stessa chiamata identica.
Analizza l'errore, correggi il codice (spesso dovuto ad API bpy cambiate o contesti sbagliati)
o informa l'utente spiegando il problema.

## Connessione

Il server socket di Blender è su `127.0.0.1:9876`.
Se i tool MCP falliscono per ConnectionRefusedError o Timeout, avvisa l'utente di verificare
che l'add-on BlenderMCP sia attivo in Blender e che la porta non sia già occupata.
```

---

## 🎨 Step 5 — Usa il tuo Assistente!

Con Blender connesso e il terminale Craicek CCOL aperto:

### 1. Verifica i tool MCP

```bash
/mcp
# Deve apparire "blender" nella lista dei server attivi
```

### 2. Prompt di avvio sessione

```
Leggi il file `blender_rules.md` e usalo come linee guida fisse per questa sessione.
Poi dimmi cosa vedi nella scena di Blender attuale.
```

### 3. Esempi di comandi

```
Aggiungi un cubo al centro della scena con un materiale rosso metallico.
```

```
Crea una lampada puntuale sopra l'oggetto selezionato con intensità 1000W.
```

```
Scarica un HDRI da Poly Haven per simulare una luce da tramonto.
```

```
Scrivi uno script Python che duplica tutti gli oggetti della scena
e li distribuisce in griglia 3x3 con spaziatura 2 unità.
```

```
Analizza la scena attuale: quanti oggetti ci sono, i loro nomi e i materiali applicati.
```

---

## 🐛 Troubleshooting

| Problema | Causa probabile | Soluzione |
|---|---|---|
| `blender` non appare in `/mcp` | Config MCP non registrata | Riesegui `claude mcp add blender uvx blender-mcp` oppure crea `.mcp.json` |
| `ConnectionRefusedError` su 9876 | Add-on non attivo | Clicca **Connect to Claude** nel tab BlenderMCP di Blender |
| Codice `bpy` errato | Contesto mancante | Arricchisci il system prompt o usa `qwen2.5-coder` |
| Ollama non risponde | Servizio non avviato | Esegui `ollama list`; avvia con `ollama serve` se necessario |
| Timeout su task lunghi | Operazione troppo pesante | Spezza in step più piccoli nel prompt |
| `BLENDER_PORT` non riconosciuto | Tipo errato nel JSON | La porta deve essere stringa: `"9876"`, non intero `9876` |
| Claude Code usa ancora Anthropic | ARC2 non attivo | Lancia Claude Code **solo** tramite `Craicek_CCOL.bat` |
| Config MCP ignorata | Conflitto globale/locale | La config locale (`.mcp.json`) ha sempre priorità su quella globale |

---

## 📁 File inclusi in questo repo

| File | Descrizione |
|---|---|
| `blender_rules.md` | System prompt con le regole operative per l'AI |
| `.mcp.json` | Configurazione MCP locale del progetto (Opzione B) |
| `Craicek_CCOL.bat` | Launcher principale (Windows) |
| `Craicek_CCOL.ps1` | Script PowerShell del launcher |
| `Craicek_CCOL-config.json` | Configurazione modello e endpoint Ollama |
| `README.md` | Questa guida |

---

## 🤝 Contribuire

Pull request e issue sono benvenute su **[github.com/BitMakerMan/ClaudeCodeOllamaBlender](https://github.com/BitMakerMan/ClaudeCodeOllamaBlender)**!

Se hai testato altri modelli Ollama con questa configurazione (es. `llama3.3`, `mistral`, `deepseek-coder`), apri una PR con i risultati — aiuta tutta la community.

---

## 📄 Licenza

Distribuito sotto licenza **MIT**. Vedi `LICENSE` per i dettagli.

---

## 🙏 Credits

- **[BlenderMCP](https://github.com/ahujasid/blender-mcp)** — add-on e protocollo MCP per Blender
- **[Craicek CCOL](https://github.com/BitMakerMan/Craicek-ClaudeCodeLauncherOllama)** — launcher per reindirizzare Claude Code verso Ollama, creato da **[@BitMakerMan](https://github.com/BitMakerMan)** (Craicek)
- **[Ollama](https://ollama.com/)** — runtime per modelli AI locali
- **[Anthropic Claude Code](https://docs.anthropic.com/claude-code)** — CLI agente AI
- **[Poly Haven](https://polyhaven.com/)** — asset 3D gratuiti e open source

---

<p align="center">
  Guida creata da <strong><a href="https://github.com/BitMakerMan">Craicek</a></strong> — 
  <a href="https://github.com/BitMakerMan/ClaudeCodeOllamaBlender">github.com/BitMakerMan/ClaudeCodeOllamaBlender</a><br>
  <em>AI locale, privacy totale, costo zero</em>
</p>
