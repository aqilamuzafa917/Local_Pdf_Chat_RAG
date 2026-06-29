<div align="center">
<h1>📚 Sistem Tanya Jawab Cerdas Lokal (Versi FAISS)</h1>
<p>
<img src="https://img.shields.io/badge/Python-3.9%2B-blue" alt="Python Version">
<img src="https://img.shields.io/badge/License-MIT-green" alt="License">
<img src="https://img.shields.io/badge/RAG-Document%20%2B%20Web%20(Optional)-orange" alt="RAG Type">
<img src="https://img.shields.io/badge/UI-Gradio-blueviolet" alt="Interface">
<img src="https://img.shields.io/badge/VectorStore-FAISS-yellow" alt="Vector Store">
<img src="https://img.shields.io/badge/LLM-Ollama%20%7C%20SiliconFlow-lightgrey" alt="LLM Support">
</p>
</div>

## 🎯 Tujuan Pembelajaran Utama

Proyek ini bertujuan untuk menyediakan platform pembelajaran praktis bagi pengembang yang ingin memahami secara mendalam prinsip-prinsip teknis RAG (Retrieval-Augmented Generation, pencarian yang disempurnakan untuk pembuatan).

*   **Membongkar Kotak Hitam RAG**: Mengimplementasikan sendiri seluruh alur kerja mulai dari pemuatan dokumen, pemecahan teks, vektorisasi, pencarian, hingga pembuatan jawaban.
*   **Menguasai Pemilihan Teknologi Kunci**: Mengalami strategi hibrida dari pencarian vektor FAISS dan pencarian kata kunci BM25.
*   **Mempraktikkan Teknik Optimasi Performa**: Mempelajari cara meningkatkan akurasi sistem RAG melalui fitur canggih seperti pengurutan ulang Cross-Encoder dan pencarian rekursif.
*   **Membangun Kemampuan Adaptasi Multi-Model**: Mengintegrasikan Ollama lokal dan API SiliconFlow cloud, menguasai strategi integrasi mesin LLM yang berbeda.

## 🌟 Fitur Utama

*   📁 **Pemprosesan Dokumen**: Mendukung pengunggahan dan pemrosesan berbagai jenis dokumen (.pdf, .txt, .docx, .md, .html, .csv, .xls, .xlsx), pemisahan otomatis, dan vektorisasi.
*   🔍 **Pencarian Hibrida**: Pencarian semantik FAISS + Pencarian kata kunci BM25, meningkatkan tingkat recall dan akurasi pencarian.
*   🔄 **Pengurutan Ulang Hasil**: Mendukung Cross-Encoder dan LLM untuk mengurutkan ulang hasil pencarian.
*   🌐 **Penyempurnaan Pencarian Web (Opsional)**: Memperoleh informasi web waktu nyata melalui SerpAPI (memerlukan konfigurasi kunci API).
*   🗣️ **Lokal/Cloud**: Dapat memilih menggunakan model besar Ollama lokal atau API SiliconFlow cloud untuk inferensi.
*   🤖 **Fallback Cerdas**: Mendeteksi backend LLM yang tersedia secara otomatis saat memulai, memprioritaskan layanan yang telah dikonfigurasi.
*   🖥️ **Antarmuka Ramah Pengguna**: Antarmuka Web interaktif yang dibangun menggunakan Gradio.
*   📊 **Visualisasi Pemecahan Teks**: Menampilkan situasi pemecahan teks dokumen pada UI untuk membantu memahami proses pengolahan data.

## 📂 Struktur Proyek (Jalur Pembelajaran)

Proyek ini dibagi menjadi modul-modul independen berdasarkan **Alur Kerja RAG**, disarankan untuk mempelajari modul demi modul sesuai dengan urutan berikut:

```
├── config.py                 # ⚙️ Konfigurasi Pusat (Variabel lingkungan, hyperparameter, deteksi otomatis LLM)
├── rag_demo.py               # 🖥️ Entri Utama (Gradio UI + Peluncuran)
├── api_router.py             # 🔌 REST API Router
│
├── core/                     # 🧠 Modul Inti RAG (Disarankan dipelajari berurutan berdasarkan alur kerja)
│   ├── document_loader.py    # 1️⃣ Pemuat Dokumen — Ekstraksi teks multi-format
│   ├── text_splitter.py      # 2️⃣ Pembagi Teks — Strategi pemecahan teks panjang
│   ├── embeddings.py         # 3️⃣ Vektorisasi — Pemetaan Teks → Vektor
│   ├── vector_store.py       # 4️⃣ Penyimpanan Vektor — Indeks FAISS (Pilihan adaptif)
│   ├── bm25_index.py         # 5️⃣ Pencarian Jarang (Sparse Retrieval) — Pencarian kata kunci BM25
│   ├── retriever.py          # 6️⃣ Pencarian Hibrida — Gabungan Semantik + Kata Kunci + Pencarian Rekursif
│   ├── reranker.py           # 7️⃣ Pengurutan Ulang (Rerank) — Pengurutan halus Cross-Encoder/LLM
│   └── generator.py          # 8️⃣ Jawaban Generator — Konstruksi Prompt + Panggilan LLM
│
├── features/                 # ✨ Fitur Tambahan
│   ├── web_search.py         # Pencarian Web (SerpAPI)
│   ├── conflict_detector.py  # Deteksi Konflik
│   └── thinking_chain.py     # Pemrosesan Chain of Thought (DeepSeek-R1)
│
└── utils/                    # 🔧 Modul Utilitas
    └── network.py            # HTTP Session + Deteksi Port
```

## 🔧 System Architecture

```mermaid
graph TD
    subgraph UI["User Interaction Layer"]
        A[Gradio Web UI] -->|Upload Document| B[Document Loader<br>Docling + OCR fallback]
        A -->|Ask Question| C[Query Entry<br>query_answer / stream_answer]
    end

    subgraph DP["Data Processing Layer"]
        B --> B1[Text Splitter<br>LangChain - chunk 400 overlap 60]
        B1 --> B2[Embedding<br>nomic-embed-text via Ollama]
        B2 --> D1["FAISS Vector Store<br>Auto: FlatIP / IVFFlat / IVFPQ"]
        B1 --> D2["BM25 Sparse Index<br>rank_bm25"]
    end

    subgraph RET["Retrieval Layer"]
        C -->|Encode Query| R1[Semantic Search<br>FAISS inner-product]
        C --> R2[Keyword Search<br>BM25Okapi]
        D1 --> R1
        D2 --> R2
        R1 --> H[Hybrid Merge<br>0.7 x semantic + 0.3 x BM25]
        R2 --> H
        H --> RR["Re-ranker<br>BAAI/bge-reranker-base CrossEncoder"]
    end

    subgraph LOOP["Recursive Retrieval Loop"]
        RR --> QJ{"Sufficient context?<br>LLM decides"}
        QJ -->|No - rewrite query| C
        QJ -->|Yes - DONE| CTX["Build Context<br>+ Conflict Detector"]
    end

    subgraph GEN["Answer Generation Layer"]
        CTX --> P["Prompt Builder<br>factoid / comparison / enumeration"]
        P --> LLM["LLM Inference<br>Ollama: Qwen3 / DeepSeek-R1"]
        LLM --> THK["Thinking Chain Processor<br>DeepSeek-R1 think tag handler"]
    end

    THK -->|Final Answer| A
```

### Technology Stack at a Glance

| Layer | Component | Library / Model |
|---|---|---|
| Document Parsing | PDF, DOCX, XLSX, PPTX | **Docling** (+ OCR fallback via pypdf) |
| Text Splitting | Recursive chunking | **LangChain** `text_splitters` |
| Embedding | Dense vector encoding | **nomic-embed-text** via **Ollama** |
| Vector Store | ANN search | **FAISS** — auto-selects `IndexFlatIP` / `IndexIVFFlat` / `IndexIVFPQ` |
| Sparse Retrieval | Keyword search | **BM25Okapi** (`rank_bm25`) |
| Hybrid Fusion | Score merging | Weighted sum (`α=0.7` semantic + `0.3` BM25) |
| Re-ranking | Cross-encoder scoring | **BAAI/bge-reranker-base** (`sentence-transformers` CrossEncoder) |
| LLM | Text generation | **Qwen3 / DeepSeek-R1** via **Ollama** |
| UI | Web interface | **Gradio** |
| API | REST endpoints | **FastAPI** + **Uvicorn** |

### Diagram 1: Indexing Pipeline (PDF → Store)

```mermaid
graph LR
    A[PDF Document] --> B[Document Loader<br>Docling + OCR]
    B --> C[Chunking<br>400 tokens, 60 overlap]
    C --> D[Generate Embeddings<br>nomic-embed-text]
    C --> E[Build Sparse Index<br>BM25]
    D --> F[(FAISS<br>Vector Store)]
    E --> G[(BM25<br>Inverted Index)]

    style A fill:#e1f5fe
    style F fill:#c8e6c9
    style G fill:#c8e6c9
```

> First, we ingest PDFs using Docling with OCR fallback. Documents are chunked into 400-token pieces with 60-token overlap. Each chunk gets a dense embedding via nomic-embed-text and a sparse BM25 index. We store these in FAISS and a BM25 inverted index for fast retrieval.

### Diagram 2: Query Pipeline (Question → Answer)

```mermaid
graph LR
    A[User Question] --> B[Hybrid Search]
    B --> C[FAISS<br>Semantic]
    B --> D[BM25<br>Keyword]
    C --> E[Fusion<br>0.7 dense + 0.3 sparse]
    D --> E
    E --> F[Cross-Encoder<br>Reranking]
    F --> G[LLM<br>Qwen3 / DeepSeek-R1]
    G --> H[Final Answer]

    F -.->|Optional: if low confidence| I[Query Rewrite]
    I -.-> B

    style A fill:#e1f5fe
    style H fill:#c8e6c9
    style I fill:#fff3e0,stroke-dasharray: 5 5
```

> When a user asks a question, we run both semantic and keyword search in parallel. Results are fused with 70% weight on semantic and 30% on keyword. A cross-encoder reranks the top results for precision. The LLM generates the final answer. If the system detects insufficient context, it optionally rewrites the query and retries.

## 🚀 Cara Penggunaan

### Persiapan Lingkungan

1.  **Membuat dan Mengaktifkan Lingkungan Virtual** (Direkomendasikan Python 3.9+):

    **Metode 1: Menggunakan venv (Direkomendasikan)**

    Mac / Linux:
    ```bash
    python3 -m venv rag_env
    source rag_env/bin/activate
    ```

    Windows:
    ```bash
    python -m venv rag_env
    rag_env\Scripts\activate
    ```

    **Metode 2: Menggunakan Conda (Opsional)**
    ```bash
    conda create -n rag_env python=3.10 -y
    conda activate rag_env
    ```

2.  **Menginstal Dependensi**:
    ```bash
    pip install -r requirements.txt
    ```

3.  **Mengonfigurasi Variabel Lingkungan**:
    ```bash
    # Salin contoh file konfigurasi
    cp example.env .env

    # Edit .env dan masukkan API Key Anda
    # Konfigurasikan setidaknya salah satu dari berikut:
    # - SILICONFLOW_API_KEY: Model besar cloud (Direkomendasikan, tidak memerlukan GPU lokal)
    # - Layanan Ollama dimulai secara lokal (perlu mengunduh model)
    ```

4.  **Menginstal dan Memulai Layanan Ollama** (Opsional, jika ingin menggunakan model besar lokal):
    *   Kunjungi [https://ollama.com/download](https://ollama.com/download) untuk mengunduh dan menginstal Ollama.
    *   Mulai layanan Ollama: `ollama serve`
    *   Tarik model yang diperlukan: `ollama pull deepseek-r1:8b`

### Deteksi Otomatis Backend LLM

Sistem akan mendeteksi backend LLM yang tersedia secara otomatis saat memulai:

| Prioritas | Kondisi | Tindakan |
|-----------|---------|----------|
| 1 | `.env` memiliki konfigurasi `SILICONFLOW_API_KEY` | Secara default menggunakan API SiliconFlow cloud |
| 2 | Layanan Ollama lokal tersedia | Secara default menggunakan model Ollama lokal |
| 3 | Keduanya tidak tersedia | Meminta pengguna untuk mengonfigurasinya |

> Di UI, Anda dapat beralih model secara manual kapan saja melalui menu tarik-turun.

### Memulai Layanan

```bash
python rag_demo.py
```

Setelah layanan dimulai, secara otomatis akan membuka `http://127.0.0.1:17995` di browser.

> ⏰ Model vektorisasi akan diunduh secara otomatis saat dijalankan pertama kali (sekitar 80MB), mohon bersabar.

## 📦 Dependensi Utama (Berdasarkan Lapisan Fungsional)

### Lapisan Interaksi Pengguna
* gradio: Membangun antarmuka Web interaktif dengan cepat

### Lapisan Pemrosesan Data
* pdfminer.six: Ekstraksi teks PDF
* langchain-text-splitters: Alat pemisah segmen teks
* sentence-transformers: Vektorisasi teks + Pengurutan ulang semantik
* faiss-cpu: Pustaka pencarian vektor yang efisien
* jieba: Pemecahan kata Bahasa Mandarin
* rank_bm25: Pencarian kata kunci BM25

### Pencarian dan Panggilan Eksternal
* requests, urllib3: Permintaan HTTP dan mekanisme coba ulang

### Sistem dan Alat Bantu
* python-dotenv: Manajemen variabel lingkungan
* psutil: Pemantauan sumber daya sistem
* numpy: Perhitungan vektor

### Layanan API Opsional
* fastapi, uvicorn: Layanan REST API independen

## 💡 Arah Tingkat Lanjut & Ekstensi

1.  **Dukungan Pencarian Multi-Langkah & Chain of Reasoning** — Menangani pertanyaan rumit yang memerlukan siklus pencarian-penalaran berulang (Sulit)
2.  **Pencarian Hibrida & Adaptasi Multi-Modal** — Mengintegrasikan pencarian konten multi-modal seperti gambar, tabel, dll. (Sedang hingga Sulit)
3.  **Evaluasi Mandiri & Siklus Optimasi Pencari** — LLM mengevaluasi kualitas pencarian dan secara otomatis meningkatkannya (Sulit)
4.  **Pembelajaran Berkelanjutan Berdasarkan Umpan Balik Pengguna** — Penyesuaian dinamis menggunakan umpan balik pengguna (Sulit)
5.  **Strategi Pembaruan Cerdas untuk Cache & Indeks** — Pembaruan indeks inkremental + lapisan cache cerdas (Sedang)

Selamat datang semua untuk menjelajah dan berkontribusi berdasarkan proyek ini!

