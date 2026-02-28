# PinPointer AI - Indexed. Offline. Yours. ⚡

A privacy-first, fully offline intelligent indexing and search engine built directly for your mobile device. 

In the modern digital age, our devices are overflowing with thousands of screenshots, downloaded PDFs, financial statements, and ID documents named nonsensically like `IMG_8492.jpg` or `scan_234.pdf`. Finding a specific tax return or Aadhaar card from a year ago usually requires aimlessly scrolling or relying on privacy-invasive cloud services (like Google Drive) to read your personal files.

**PinPointer AI** changes this paradigm completely. We bring enterprise-grade search, machine-learning OCR, and intelligent file classification directly to the edge. It scans your media and documents, understands their contexts, and securely stores the metadata locally. **100% Offline. Zero Data Leaves Your Device.**

---

## 🏆 What We Built (Main Features for the Hackathon)

We transformed a core AI SDK template into a flagship, offline-first intelligent platform for your mobile device. Here is what makes PinPointer powerful:

### 1. ⚡ Universal Sync (The Core Engine)
The heart of PinPointer. **Universal Sync** silently sweeps through your entire device's storage (Fragmented Gallery, Downloads folders, WhatsApp Documents) and analyzes every image and PDF using on-device ML Vision algorithms. It reads the text, creates an invisible index, and structures your chaotic device into a searchable database.

### 2. 🤖 Privacy-First On-Device AI Models (Voice & Text)
PinPointer guarantees that your data never leaves your phone by running heavy neural models locally:
*   **Speech-To-Text (STT):** Powered by Sherpa ONNX Whisper, enabling real-time, offline transcription.
*   **Text-To-Speech (TTS):** Neural voice synthesis via Piper TTS for offline read-aloud capabilities.
*   **LLM Text Generation:** On-device Large Language Models (LLaMa Cpp) for private, offline intelligence.

### 3. 🏦 The Intelligent Document Vault (Auto-Classification)
As Universal Sync processes your device, our ML-powered engine (`DocumentClassifier.ts`) reads the raw text to intelligently sort unstructured files into secure vaults:
*   **Identity Documents:** Aadhaar, PAN cards, Driving Licenses.
*   **Financial Documents:** Bank Statements, Credit Card Bills.
*   **Medical & Education:** Reports, Transcripts, Exam admit cards.

### 4. 🔍 Neural Search Engine
A sub-millisecond SQLite `FTS5` engine that doesn't just search filenames — it searches the *content* inside them.
*   **Phonetic Matching:** If you search `Adhar`, the engine uses mathematical Soundex algorithms to realize you mean `Aadhaar`.
*   **Contextual Snippets:** Like Google, search results show exactly *where* your keyword was found inside the document with highlighted text.

### 5. 🛡️ Absolute Privacy & PII Redaction
*   **Air-Gapped Security:** The app works perfectly in Airplane Mode. 
*   **In-Memory Masking:** As the app scans your files, it actively looks for highly sensitive strings (Aadhaar Numbers, PAN Numbers, Phone Numbers) using Regex. Before saving to the local SQLite database, it aggressively masks them (e.g., `XXXX-XXXX-1234`). Only you can see the raw file.

---

## 🧠 Technical Challenges & How We Solved Them

Building an offline, on-device AI scanner introduces massive constraints regarding battery life, memory, and false positives. Here is how we engineered around them:

### Challenge 1: Battery Drain from Unnecessary ML Models
**Problem:** Running OCR (Text Recognition) and Object Labeling on 5,000 photos will drain a battery in minutes and overheat the phone.
**Solution (Early Exit Pipeline):** We structured our `VisionPipeline.ts` sequentially. We run the highly-efficient OCR pass first. *If we detect text, we immediately exit the pipeline and save battery.* We only ever invoke the heavier Image Labeling model if the image is completely devoid of text (like a photo of a dog or a landscape).

### Challenge 2: "Dirty Data" & Garbled OCR
**Problem:** A blurry photo of a receipt might output OCR text like `x$3@##kk`. Storing this pollutes the database and ruins search relevance.
**Solution (Garbage Filtering):** We wrote a heuristic `isGarbageText()` function. It analyzes the OCR output in real-time. If less than 30% of the text is alphanumeric, or if it consists mostly of single-character chunks, the pipeline silently rejects the text as "noise."

### Challenge 3: Password-Protected PDFs
**Problem:** Users download Bank Statements and e-Aadhaar cards that are heavily encrypted. ML Kit OCR cannot read inside them, so our Vault AI couldn't classify them.
**Solution (Heuristic Filename Fallbacks):** We built a customized regex fallback layer. Even if the PDF is locked, if we detect specific banking or government filename patterns (e.g., `Your 811 account details` or `^e-?Aadhaar`), we instantly bypass the OCR failure and accurately route it to the Financial or Identity vault.

### Challenge 4: Aggressive Misclassification (The "Education" Problem)
**Problem:** The ML was too eager. Generic PDFs (like a "Notice for Release of Admit Card") were being misclassified as highly sensitive "Aadhaar Cards" simply because they contained overlapping keywords like "Provided" or "Male."
**Solution (Strict Confidence & Safe Second-Pass):** 
1. We increased the required AI confidence threshold to 30% and mandated at least *two* distinct keywords to trigger a high-security classification.
2. We introduced a completely new "Education" category. To ensure it never "stole" a true Identity document, we built a `scoreEducation()` algorithm that operates safely: it *only* runs as a second pass on files that the AI already deemed a "General Document".

### Challenge 5: UI Freezing (Out Of Memory)
**Problem:** Analyzing and saving hundreds of documents at once locked the React Native main thread and caused Out-Of-Memory (OOM) crashes on low-ram devices.
**Solution (Batch Transactions & Caps):** We rewrote the SQLite ingestion loop to use `BEGIN TRANSACTION` and `COMMIT` blocks. This batching reduced a 30-second database sync to a blistering 2 seconds. Furthermore, we implemented a hard safety limit: the app only syncs 50 documents per batch, guaranteeing structural stability.

---

## 🎨 Premium UI / UX Redesign

We completely overhauled the standard template to make it feel like a flagship application.
*   **Haptic Intelligence:** Integrated micro-vibrations (10ms) across all major UI interactions.
*   **Glassmorphism & Gradients:** Replaced flat colors with `react-native-linear-gradient` glowing aurora backgrounds to make the app feel alive and premium.
*   **Dynamic Dashboard Layout:** Rebuilt the HomeScreen from a generic grid into a carefully weighted flex layout, anchoring the crucial "Doc Vault" feature in a dominant, full-height card to draw the user's eye.
*   **Categorical Emojis:** Native visual cues (🏦 Bank Statements, 🪪 ID Cards, 🎓 Education) replace boring generic file icons throughout the vault and search screens.

---

## 🚀 Getting Started (Run It Yourself)

### Prerequisites
*   Node.js 18+
*   React Native CLI environment
*   Android Studio (JDK 17+, Android SDK 36, NDK 27.1.12297006)

### Installation
```bash
# Clone the repository
git clone https://github.com/nishant-kumar-yadav/PinPointer.Ai.git
cd PinPointer.Ai

# Install dependencies
npm install

# Run on Android
npx react-native run-android
```
*(Note: Because of the custom Native C++ modules, the first Android build takes 5-10 minutes).*

---

## 🏗 Architecture Blueprint

```
src/
├── Database.ts                  # SQLite + FTS5 search engine
├── services/
├── utils/
│   ├── VisionPipeline.ts       # OCR + Image Labeling pipeline
│   ├── DocumentPipeline.ts     # Document indexing & Vault processing
│   ├── DocumentClassifier.ts   # AI classification heuristics
│   ├── DataMasking.ts          # Sensitive info redaction (In-memory)
│   ├── Soundex.ts              # Phonetic search mathematical models
│   └── HindiTranslit.ts        # Devanagari → Latin transliteration
├── screens/
│   ├── HomeScreen.tsx          # Feature navigation (Premium Layout)
│   ├── PinpointerScreen.tsx    # Neural Search Dashboard
│   ├── DocumentVaultScreen.tsx # AI-Classified Document Views
...
```

---

*Built with ❤️ for the Hackathon. Fast, Private, and intelligently yours.*
