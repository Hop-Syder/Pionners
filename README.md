# 🧱 Plan Global du Projet : Transcription Automatique en Langue Fon

## Étape 0 : Objectif Final
> Créer un **système robuste et adaptable** qui prend une vidéo/audio en entrée (en langue Fon) et produit une **transcription textuelle fidèle**, malgré les faibles ressources linguistiques disponibles.

---

## 🔁 Étapes Clés du Pipeline

| Étape | Description | Outils Recommandés |
|------|-------------|-------------------|
| 1. Collecte de Données Audio | Télécharger des vidéos YouTube en langue Fon, extraire l’audio | `yt-dlp`, `youtube-dl` |
| 2. Prétraitement Audio | Nettoyage, segmentation vocale, conversion au format standard | `pydub`, `librosa`, `VAD` |
| 3. Reconnaissance Vocale (ASR) | Utilisation d’un modèle ASR multilingue adapté | `Whisper (OpenAI)` |
| 4. Correction Linguistique | Post-traitement avec lexique Fon et/ou LLM léger | `SpellChecker`, TinyLLM |
| 5. Évaluation & Feedback | Comparaison avec transcriptions manuelles | WER, BLEU, CER |
| 6. Interface Utilisateur (optionnel) | Interface CLI ou Web pour faciliter l’utilisation | `Streamlit`, `Gradio` |

---

# ✅ Étape par Étape : Repartons de Zéro

---

## 📥 Étape 1 : Collecte de Données Audio (YouTube)

### But :
Trouver et télécharger des vidéos parlées en **langue Fon** sur YouTube.

### Script Simple :

```bash
pip install yt-dlp
```

```bash
yt-dlp --extract-audio --audio-format wav \
       -o "videos/%(title)s.%(ext)s" \
       --max-downloads 20 \
       "ytsearch:language fon site:youtube.com"
```

> Ce script recherche 20 vidéos contenant le mot clé “fon” ou liées à la langue Fon.

---

## 🎵 Étape 2 : Extraction et Prétraitement Audio

### But :
Transformer les vidéos téléchargées en fichiers audio propres.

```python
from pydub import AudioSegment
import os

def extract_audio(video_file, output_file):
    audio = AudioSegment.from_file(video_file)
    audio.export(output_file, format="wav")

video_dir = "videos/"
audio_dir = "audios/"
os.makedirs(audio_dir, exist_ok=True)

for file in os.listdir(video_dir):
    if file.endswith(".mp4"):
        input_path = os.path.join(video_dir, file)
        output_path = os.path.join(audio_dir, file.replace(".mp4", ".wav"))
        extract_audio(input_path, output_path)
```

---

## 🗣️ Étape 3 : Transcription via Whisper (ASR)

### But :
Utiliser Whisper pour générer une transcription brute.

```bash
pip install openai-whisper
```

```python
import whisper

model = whisper.load_model("large")  # Vous pouvez aussi utiliser "medium" ou "small"

def transcribe_audio(audio_file):
    result = model.transcribe(audio_file, language="fr")
    return result["text"]
```

> Note : Nous utilisons `"fr"` comme langue car le modèle ne reconnaît pas nativement le **Fon**.

---

## 📚 Étape 4 : Correction Linguistique en Langue Fon

### But :
Améliorer la transcription grâce à un correcteur basé sur un **lexique Fon**.

#### Solution 1 : Lexique + Spellchecker

```bash
pip install spellchecker
```

Créez un fichier texte (`fon_dict.txt`) contenant les mots Fon connus, un par ligne.

```python
from spellchecker import SpellChecker

spell = SpellChecker(language=None, local_dictionary='fon_dict.txt')

def correct_transcript(text):
    words = text.split()
    corrected_words = [spell.correction(word) for word in words]
    return " ".join(corrected_words)
```

#### Solution 2 : Modèle de Langue ou LLM Léger (TinyLLM / Phi-3)

Fine-tunez un petit modèle sur un corpus Fon disponible (ex : [African Language Resources](https://github.com/topics/african-languages)) ou utilisez un prompt pour corriger la syntaxe :

```python
def prompt_corrector(transcribed_text):
    prompt = f"""
Corrige cette transcription en respectant l'orthographe et la grammaire en langue Fon :
{transcribed_text}
"""
    # À connecter à un LLM local ou API
    corrected_text = call_local_llm(prompt)  # Exemple fictif
    return corrected_text
```

---

## 📊 Étape 5 : Évaluation Quantitative

### But :
Mesurer la qualité des transcriptions.

```bash
pip install jiwer
```

```python
from jiwer import wer, cer

reference = "Transcription correcte en langue Fon."
hypothesis = "Transcription bruitee en lang Fon."

print("WER:", wer(reference, hypothesis))
print("CER:", cer(reference, hypothesis))
```

---

## 🖥️ Étape 6 : Interface Utilisateur (Facultatif mais utile)

### But :
Créer une interface simple pour tester facilement vos transcriptions.

```bash
pip install streamlit
```

```python
import streamlit as st
import os

st.title("Transcripteur Automatique en Langue Fon")

uploaded_file = st.file_uploader("Téléchargez une vidéo ou un fichier audio", type=["mp4", "wav"])

if uploaded_file:
    with open(os.path.join("audios", uploaded_file.name), "wb") as f:
        f.write(uploaded_file.getbuffer())
    st.success("Fichier téléchargé !")
    
    # Appel à la fonction de transcription
    transcript = transcribe_audio(f"audios/{uploaded_file.name}")
    st.subheader("Transcription brute")
    st.write(transcript)
    
    corrected = correct_transcript(transcript)
    st.subheader("Transcription corrigée")
    st.write(corrected)
```

Lancez avec :

```bash
streamlit run app.py
```

---

# 🧠 Résumé du Workflow Final

```
YouTube Vidéo → Extraction Audio → Whisper → Correction Orthographique → Transcription Finale
```

---

# 📦 Structure de Dossier Proposée

```
projet-fon/
├── videos/
│   └── video1.mp4 ...
├── audios/
│   └── audio1.wav ...
├── transcripts/
│   ├── raw/
│   └── corrected/
├── models/
│   └── whisper/
├── tools/
│   ├── audio_utils.py
│   ├── asr_pipeline.py
│   └── correction.py
├── results/
│   ├── metrics.csv
│   └── samples/
└── app.py
```

---

