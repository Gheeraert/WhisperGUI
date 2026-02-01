# WhisperGUI — Transcription audio/vidéo avec Whisper

Une petite application **Tkinter** (Python) pour transcrire des fichiers **audio ou vidéo** avec **Whisper** (bibliothèque `openai-whisper`), avec :

- auto-détection **GPU/CUDA** (PyTorch),
- choix du **modèle** (`tiny` → `large-v3`) et des paramètres principaux,
- exports **TXT**, **SRT**, **VTT**, **JSON**,
- téléchargement **à la demande** des modèles Whisper (cache local),
- interface simple + journal d’exécution,
- avertissement explicite si CUDA n’est pas disponible.

> Objectif : un outil “léger” et pratique pour un poste de travail. Les modèles ne sont **pas** embarqués : ils sont téléchargés automatiquement lors du premier chargement, puis réutilisés depuis le cache.

---

## Fonctionnalités

- **Entrée** : `.wav .mp3 .m4a .flac .ogg .aac .mp4 .mkv .mov .webm ...`
- **Sorties** :
  - `*.txt` : texte brut
  - `*.srt` : sous-titres (horodatés)
  - `*.vtt` : WebVTT
  - `*.json` : sortie détaillée Whisper (segments, scores, etc.)
- **Paramètres** :
  - modèle (menu déroulant),
  - langue (`fr`, `en`, … ou vide = auto),
  - tâche (`transcribe` / `translate`),
  - device (`auto`, `cuda`, `cpu`),
  - FP16 (GPU uniquement),
  - verbosité (console/log).
- **Robustesse UI** :
  - transcription en **thread** (l’interface ne se fige pas),
  - boîte modale “Chargement du modèle…” avec barre d’activité.

---

## Captures / aperçu

Au lancement, un bandeau indique l’état des dépendances (ffmpeg / whisper / torch) et du GPU :

- `ffmpeg: OK | openai-whisper: OK | torch: OK | GPU: ...`
- ou, si CUDA indisponible : `GPU: CPU (CUDA indisponible)`

---

## Prérequis

- **Python** 3.11 ou 3.12 (64 bits recommandé, surtout sous Windows)
- **FFmpeg** accessible dans le `PATH`
- **PyTorch** (idéalement avec CUDA si GPU NVIDIA)
- `openai-whisper`
- Tkinter (inclus avec Python sur Windows/macOS ; sous Linux, dépend du paquet de la distribution)

---

## Installation (recommandée : environnement virtuel)

### 1) Créer/activer un venv

#### Windows (PowerShell)
```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\activate
python -m pip install -U pip setuptools wheel
```

#### macOS / Linux
```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -U pip setuptools wheel
```

### 2) Installer FFmpeg

- Windows : via Chocolatey `choco install ffmpeg -y` ou binaire + PATH
- macOS : `brew install ffmpeg`
- Debian/Ubuntu : `sudo apt-get install ffmpeg`

Vérification :
```bash
ffmpeg -version
```

### 3) Installer PyTorch (GPU CUDA, Windows)

**Option CUDA 11.8 (souvent la plus robuste sur Windows)** :
```powershell
python -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

Vérification GPU :
```powershell
python -c "import torch; print(torch.cuda.is_available()); print(torch.version.cuda); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'no cuda')"
```

> Si `torch.cuda.is_available()` est `False`, l’app fonctionnera quand même, mais sur CPU (plus lent).

### 4) Installer Whisper + PyInstaller (optionnel)

```bash
python -m pip install -U openai-whisper
python -m pip install -U pyinstaller
```

---

## Lancer l’application

```bash
python whisper_tk.py
```

---

## Modèles : téléchargement à la demande (cache)

Au premier chargement d’un modèle (ex. `small`, `medium`…), Whisper télécharge automatiquement les poids, puis les conserve localement.

- Avantage : l’exécutable reste “léger”.
- Inconvénient : première utilisation d’un modèle = attente + connexion réseau.

### Où sont stockés les modèles ?
Selon l’OS, Whisper utilise en général un cache utilisateur (souvent `~/.cache/whisper`).  
Sous Windows, cela peut se trouver dans le profil utilisateur (selon la configuration Python).

---

## Conseils de performance

- Sur **GPU** (NVIDIA + CUDA OK) :
  - `medium` est souvent un très bon compromis qualité/vitesse.
  - FP16 accélère nettement.
- Sur **CPU** :
  - privilégier `small` voire `base` (sinon, très lent).
  - l’app affichera un warning si CUDA est attendu mais indisponible.

---

## Build Windows : créer un exécutable (PyInstaller)

### Recommandation : `--onedir` (plus robuste avec PyTorch)
```powershell
python -m PyInstaller --clean --onedir --noconsole --name WhisperGUI `
  --collect-all whisper `
  --collect-all torch `
  whisper_tk.py
```

Résultat :
- `dist\WhisperGUI\WhisperGUI.exe`

  
## Dépannage

### Warning : “FP16 is not supported on CPU; using FP32 instead”
- S'assurer que CUDA est disponible (`torch.cuda.is_available()`) si l'on souhaite utiliser le GPU.
- Solutions: réinstaller PyTorch CUDA (`cu118`), mettre à jour les drivers NVIDIA
- Test :
```python
import torch
print(torch.cuda.is_available())
```

---

## Roadmap (idées)

- bouton “Précharger le modèle” (téléchargement explicite)
- bouton “Ouvrir le dossier de cache”
- estimation du temps restant (plus compliqué : nécessite instrumentation avancée)
- diarisation (séparation automatique des locuteurs) via outils externes

---

## Licence

Licence MIT.  
---

## Crédits

- Whisper / `openai-whisper`
- PyTorch
- FFmpeg
- Tkinter

---

## Contribuer

Les issues et PR sont bienvenues :
- signaler un bug (OS, version Python, version torch, logs)
- proposer des améliorations UI
