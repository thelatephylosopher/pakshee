# Pakshee • पक्षी
> *A sonified air quality experience — hear how India breathes.*

Made my Cuta little baybe

**Pakshee** (Hindi for *bird*) is an interactive sonic cartography of ambient air quality across the Indian subcontinent. Rather than relying solely on abstract numbers and color bands, Pakshee transforms atmospheric particulate matter (**PM2.5**) into real-time harmonic dissonance, texture, and timbre.

Users can click any monitoring station across India, explore scroll-driven regional narratives, and investigate the **"Silent Zones"**—districts where millions of citizens breathe unmonitored air.

---

## 🌊 Architecture & Data Flow

```
   ┌─────────────────────────────────────────────────────────────┐
   │                Scheduled Pipeline (GitHub Actions)          │
   │                                                             │
   │  Every 6 hours ──► fetch_openaq.py (765 stations, India)   │
   │                ──► fetch_districts.py (Census bias layer)   │
   │                ──► spatial_audit.py (Integrity checks)      │
   │                ──► Commits to data/processed/*.geojson      │
   └──────────────────────────────┬──────────────────────────────┘
                                  │
                                  ▼
   ┌─────────────────────────────────────────────────────────────┐
   │                  Web Experience (Client-side)               │
   │                                                             │
   │   • MapLibre GL JS : Dark-matter basemap & glow markers     │
   │   • Tone.js        : Formant poly-synth, FM & bitcrush      │
   │   • D3.js          : NAQI color interpolation & scales      │
   │   • Static hosting : Zero CORS friction on GitHub Pages     │
   └─────────────────────────────────────────────────────────────┘
```

---

## 🎵 The Sonic Mapping (PM2.5 to Sound)

Pakshee translates particulate density into acoustic parameters via [Tone.js](https://tonejs.github.io/):

| PM2.5 Range (µg/m³) | NAQI Category | Harmonic Character | Acoustic Timbre |
|:---:|:---:|:---|:---|
| **0 – 30** | **Good** | Consonant D Major 9 (`D4 F#4 A4 E5`) | Crystal clear, open lowpass (9 kHz), airy bird-like FM modulation |
| **31 – 60** | **Satisfactory** | Major with subtle flat 7th (`D4 F#4 A4 C5`) | Mild analog warmth, light chorus detuning (+5 cents) |
| **61 – 90** | **Moderate** | Suspended Minor 4th (`D4 G4 A4 C5`) | Acoustic tension, cutoff lowered to 3.8 kHz, chorus detuning (+15 cents) |
| **91 – 120** | **Poor** | Tritone Dissonance (`D4 G#4 C5 F5`) | Diminished 5th discordance, 35% distortion drive, pink noise undertone |
| **121 – 250** | **Very Poor** | Half-step Clusters (`D4 Eb4 Ab4 A4`) | Abrasive semitone clash, 60% saturation, suffocating 1.4 kHz choke |
| **250+** | **Severe** | Degraded Noise Cluster (`C#4 D4 D#4 G#4`) | Full bitcrusher degradation, sub-1000 Hz choke, oppressive noise wash |

> [!TIP]
> **Custom Voice / Soundscape Samples**: Drop your own `.mp3` or `.ogg` stems into `web/audio/` and register them in [`web/js/audio.js`](file:///Users/ri/Downloads/pakshee/web/js/audio.js).

---

## 🗺️ The Bias Layer: Silent Zones

Air pollution is not just a scientific problem; it is an equity problem. Pakshee compares station GPS locations against 594 Indian administrative districts:

- **270 Monitored Districts**: Covered by 765 active continuous ambient monitoring stations (CPCB / SPCB / OpenAQ).
- **324 Silent Zones**: Districts with **zero** official continuous monitoring stations.
- **123 Critical Silent Zones**: High-density districts (>400 people/km²) with no real-time public data. In Pakshee, clicking these zones triggers a haunting, muted hollow frequency—highlighting what remains unheard.

---

## 🚀 Quickstart & Local Development

### 1. Prerequisites
- Python 3.9+ with virtual environment
- Node.js (optional, for local static serving)

### 2. Environment Setup
Create a `.env` file at the root of the repository with your free OpenAQ API key:
```env
OPENAQ_API_KEY=your_openaq_api_key_here
```
*(Get a free key in 30 seconds at [explore.openaq.org](https://explore.openaq.org/)).*

### 3. Run the Data Pipeline
```bash
# Activate your virtual environment
source .venv/bin/activate

# Fetch all Indian stations + live PM2.5 readings
python3 pipeline/fetch_openaq.py

# Build the district boundary & monitoring bias layer
python3 pipeline/fetch_districts.py

# Run spatial and data quality audit
python3 pipeline/spatial_audit.py
```

### 4. Run the Web Application
```bash
# Using python's built-in web server from the repository root:
python3 -m http.server 3000

# Open your browser at:
# http://localhost:3000/web/
```

Or using Node:
```bash
cd web
npx serve . -p 3000
# Open http://localhost:3000
```

---

## ⚙️ Automated GitHub Actions

1. **`update-data.yml`**: Runs every 6 hours via cron, queries OpenAQ v3 for new measurements, recomputes the bias layer, and auto-commits the refreshed `stations.geojson` back to the repository.
2. **`deploy-pages.yml`**: Deploys the static web app and GeoJSON datasets directly to **GitHub Pages**.

> [!IMPORTANT]
> To enable automated pipeline updates in your GitHub repo:
> 1. Go to your repo on GitHub $\rightarrow$ **Settings** $\rightarrow$ **Secrets and variables** $\rightarrow$ **Actions**.
> 2. Click **New repository secret**.
> 3. Name: `OPENAQ_API_KEY`
> 4. Value: *[Your OpenAQ API Key]*

---

## 📜 License
MIT © [Riya Kashive](https://github.com/thelatephylosopher)
