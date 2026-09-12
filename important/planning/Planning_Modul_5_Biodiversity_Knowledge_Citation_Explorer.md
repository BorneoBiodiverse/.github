# Planning Function - Modul 5: Biodiversity Knowledge & Citation Explorer

**Mata Kuliah:** Pemrograman Fungsional  
**Bahasa:** Rust  
**Cakupan:** Eksplorasi publikasi ilmiah, topik riset, lokasi penelitian, timeline riset, coverage analysis, dan rekomendasi sitasi biodiversitas Kalimantan.

**Repository:** `kalimantanbio-modul5-knowledge-citation`

**Sumber data:** `kalimantan-bio-demo-api` dan dataset publikasi ilmiah terstruktur.

> Catatan integrasi: API demo saat ini menyediakan data spesies, taksonomi, observasi, lokasi, institusi, dan dataset, tetapi belum menyediakan endpoint publikasi ilmiah. Karena itu, data publikasi pada tahap awal dapat dibaca dari fixture JSON/CSV atau adapter API terpisah. Integrasi endpoint publikasi dapat ditambahkan tanpa mengubah fungsi analitik murni.

---

## 1. Tujuan Modul

Modul ini membangun sistem eksplorasi pengetahuan ilmiah yang menghubungkan spesies biodiversitas Kalimantan dengan publikasi, topik riset, lokasi penelitian, peneliti, dan referensi sitasi.

Modul ini bertanggung jawab untuk:

* Menemukan publikasi yang berhubungan dengan spesies atau kelompok taksonomi.
* Mengeksplorasi publikasi berdasarkan topik biodiversitas dan lokasi penelitian.
* Menganalisis perkembangan penelitian dari waktu ke waktu.
* Mengukur coverage penelitian dan menemukan spesies atau takson yang understudied.
* Memberikan rekomendasi referensi serta mengekspor sitasi ke format akademik umum.

### Batasan Modul

Modul ini **tidak bertanggung jawab** terhadap:

* Menyimpan atau mengubah master data spesies dan taksonomi.
* Menentukan kebenaran ilmiah isi publikasi secara otomatis.
* Mengunduh teks penuh yang dibatasi hak aksesnya.
* Membuat visualisasi UI secara langsung; modul hanya menghasilkan data graph dan report.
* Menggantikan proses peer review atau validasi bibliografis oleh ahli.

---

## 2. Struktur Data (Domain Model)

```rust
#[derive(Debug, Clone, serde::Deserialize)]
struct Publication {
    id: String,
    title: String,
    abstract_text: Option<String>,
    publication_year: u16,
    authors: Vec<Author>,
    venue: Option<String>,
    doi: Option<String>,
    url: Option<String>,
    source_type: SourceType,
    species_ids: Vec<u64>,
    taxon_names: Vec<String>,
    topics: Vec<ResearchTopic>,
    locations: Vec<ResearchLocation>,
    citation_count: Option<u32>,
}

#[derive(Debug, Clone, serde::Deserialize)]
struct Author {
    id: Option<String>,
    name: String,
    affiliation: Option<String>,
}

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
enum SourceType {
    JournalArticle,
    ConferencePaper,
    Thesis,
    Report,
    BookChapter,
    Dataset,
}

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
enum ResearchTopic {
    Taxonomy,
    Ecology,
    Conservation,
    Ethnobotany,
    Habitat,
    SpeciesIdentification,
    Other(String),
}

#[derive(Debug, Clone, serde::Deserialize)]
struct ResearchLocation {
    name: String,
    province: Option<String>,
    regency: Option<String>,
    location_type: LocationType,
}

#[derive(Debug, Clone, PartialEq, Eq, Hash)]
enum LocationType {
    Province,
    Regency,
    Forest,
    ConservationArea,
    Other(String),
}

#[derive(Debug, Clone, Default)]
struct PublicationQuery {
    text: Option<String>,
    species_id: Option<u64>,
    taxon_name: Option<String>,
    topic: Option<ResearchTopic>,
    location: Option<String>,
    year_from: Option<u16>,
    year_to: Option<u16>,
    source_type: Option<SourceType>,
}

#[derive(Debug, Clone, Default)]
struct CoverageRecord {
    entity_name: String,
    publication_count: usize,
    species_count: usize,
    first_year: Option<u16>,
    latest_year: Option<u16>,
}
```

### Data yang Digunakan

| Data | Tipe | Deskripsi |
| --- | --- | --- |
| `Publication` | `struct` | Metadata publikasi dan hubungan ilmiahnya. |
| `species_ids` | `Vec<u64>` | ID spesies yang dibahas dalam publikasi. |
| `taxon_names` | `Vec<String>` | Genus, famili, atau takson lain yang berkaitan. |
| `topics` | `Vec<ResearchTopic>` | Kategori topik biodiversitas penelitian. |
| `locations` | `Vec<ResearchLocation>` | Lokasi studi di Kalimantan. |
| `publication_year` | `u16` | Tahun terbit untuk timeline dan analisis tren. |
| `citation_count` | `Option<u32>` | Jumlah sitasi jika tersedia dari sumber data. |

---

## 3. Tahap 1 - Data Ingestion, Normalization & Validation

Membaca data publikasi dari JSON/CSV/API, menormalisasi metadata, dan menghapus atau menandai record yang tidak valid.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `parse_publications` | `fn parse_publications(raw: &str) -> Result<Vec<Publication>, ParseError>` | Mendeserialisasi input publikasi terstruktur. |
| `normalize_publication` | `fn normalize_publication(publication: Publication) -> Publication` | Menormalkan judul, DOI, nama penulis, topik, dan lokasi. |
| `validate_publication` | `fn validate_publication(publication: &Publication) -> Result<(), ValidationError>` | Memeriksa ID, judul, tahun, dan metadata minimum. |
| `deduplicate_publications` | `fn deduplicate_publications(items: &[Publication]) -> Vec<Publication>` | Menghapus duplikasi berdasarkan ID, DOI, atau fingerprint judul. |
| `build_species_index` | `fn build_species_index(items: &[Publication]) -> HashMap<u64, Vec<String>>` | Membuat indeks spesies ke ID publikasi. |

**Person in Charge:** **[Nama anggota]**

---

## 4. Tahap 2 - Species, Topic & Location Explorer

Menyediakan query murni untuk menemukan publikasi berdasarkan spesies, topik, lokasi, dan kombinasi filter.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `publications_for_species` | `fn publications_for_species<'a>(items: &'a [Publication], species_id: u64) -> Vec<&'a Publication>` | Mengambil publikasi yang terkait dengan spesies. |
| `publications_for_taxon` | `fn publications_for_taxon<'a>(items: &'a [Publication], taxon: &str) -> Vec<&'a Publication>` | Mencari publikasi berdasarkan nama genus/famili/takson. |
| `publications_for_topic` | `fn publications_for_topic<'a>(items: &'a [Publication], topic: &ResearchTopic) -> Vec<&'a Publication>` | Memfilter publikasi berdasarkan topik riset. |
| `publications_for_location` | `fn publications_for_location<'a>(items: &'a [Publication], location: &str) -> Vec<&'a Publication>` | Memfilter publikasi berdasarkan lokasi studi. |
| `query_publications` | `fn query_publications<'a>(items: &'a [Publication], query: &PublicationQuery) -> Vec<&'a Publication>` | Menggabungkan seluruh filter query menggunakan iterator predicates. |

**Person in Charge:** **[Nama anggota]**

---

## 5. Tahap 3 - Timeline & Research Coverage Analysis

Mengubah kumpulan publikasi menjadi statistik perkembangan dan cakupan penelitian.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `build_research_timeline` | `fn build_research_timeline(items: &[Publication]) -> BTreeMap<u16, usize>` | Menghitung jumlah publikasi per tahun. |
| `calculate_topic_distribution` | `fn calculate_topic_distribution(items: &[Publication]) -> HashMap<ResearchTopic, usize>` | Menghitung distribusi publikasi berdasarkan topik. |
| `calculate_location_distribution` | `fn calculate_location_distribution(items: &[Publication]) -> HashMap<String, usize>` | Menghitung jumlah publikasi per lokasi. |
| `calculate_entity_coverage` | `fn calculate_entity_coverage(items: &[Publication], species_ids: &[u64]) -> Vec<CoverageRecord>` | Membandingkan jumlah publikasi antarspesies. |
| `find_understudied_species` | `fn find_understudied_species(coverage: &[CoverageRecord], threshold: usize) -> Vec<&CoverageRecord>` | Menemukan spesies dengan jumlah publikasi di bawah threshold. |
| `calculate_coverage_score` | `fn calculate_coverage_score(published: usize, expected: usize) -> f64` | Menghasilkan skor coverage dalam rentang 0.0 hingga 1.0. |

**Person in Charge:** **[Nama anggota]**

> Aturan threshold dan definisi `expected` harus disepakati tim. Contoh awal: spesies dengan 0-1 publikasi masuk kategori understudied.

---

## 6. Tahap 4 - Citation Ranking, Recommendation & Export

Mengurutkan referensi yang paling relevan dan menghasilkan format sitasi yang dapat digunakan dalam tulisan akademik.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `score_publication_relevance` | `fn score_publication_relevance(publication: &Publication, query: &PublicationQuery) -> f64` | Menghitung skor berdasarkan kecocokan spesies, topik, lokasi, tahun, dan teks. |
| `rank_publications` | `fn rank_publications<'a>(items: &[&'a Publication], query: &PublicationQuery) -> Vec<(&'a Publication, f64)>` | Mengurutkan publikasi berdasarkan skor relevansi. |
| `recommend_citations` | `fn recommend_citations<'a>(items: &'a [Publication], query: &PublicationQuery, limit: usize) -> Vec<&'a Publication>` | Menghasilkan rekomendasi referensi teratas. |
| `format_apa` | `fn format_apa(publication: &Publication) -> String` | Mengekspor satu publikasi ke format APA sederhana. |
| `format_bibtex` | `fn format_bibtex(publication: &Publication) -> String` | Mengekspor satu publikasi ke format BibTeX. |
| `export_citations` | `fn export_citations(items: &[Publication], format: CitationFormat) -> String` | Menghasilkan kumpulan sitasi dalam format yang dipilih. |

**Person in Charge:** **[Nama anggota]**

> Citation ranking hanya merekomendasikan referensi berdasarkan metadata. Hasilnya tetap perlu diverifikasi pengguna sebelum digunakan sebagai sitasi akademik.

---

## 7. Tahap 5 - Knowledge Network & Integrasi

Membangun representasi relasi antarspesies, publikasi, topik, lokasi, dan peneliti, lalu menguji seluruh pipeline.

| Fungsi / Komponen | Signature / Bentuk | Deskripsi |
| --- | --- | --- |
| `build_knowledge_graph` | `fn build_knowledge_graph(items: &[Publication]) -> KnowledgeGraph` | Membuat node dan edge dari relasi publikasi. |
| `related_publications` | `fn related_publications<'a>(graph: &KnowledgeGraph, publication_id: &str, items: &'a [Publication]) -> Vec<&'a Publication>` | Mencari publikasi yang berbagi spesies, topik, lokasi, atau penulis. |
| `fetch_publications` | `async fn fetch_publications(base_url: &str) -> Result<Vec<Publication>, SourceError>` | Adapter I/O untuk sumber publikasi eksternal. |
| `explore_knowledge` | `fn explore_knowledge(input: &KnowledgeInput) -> Result<KnowledgeReport, PipelineError>` | Entry point integrasi seluruh tahap. |
| Unit tests | `mod tests { ... }` | Menguji parsing, filtering, timeline, coverage, ranking, dan export. |

**Person in Charge:** **[Nama anggota]**

Knowledge graph minimal:

```text
Species ---- discussed_in ---- Publication ---- written_by ---- Researcher
   |                              |
   |                              +---- about ---- Topic
   |
   +----------------------------- studied_at ---- Location
```

---

## 8. Komposisi Pipeline Utama

```rust
fn explore_knowledge(input: &KnowledgeInput) -> Result<KnowledgeReport, PipelineError> {
    let parsed = parse_publications(&input.raw_publications)?;
    let normalized = parsed
        .into_iter()
        .map(normalize_publication)
        .collect::<Vec<_>>();
    let publications = deduplicate_publications(&normalized);

    let matching = query_publications(&publications, &input.query);
    let ranked = rank_publications(&matching, &input.query);
    let timeline = build_research_timeline(&publications);
    let coverage = calculate_entity_coverage(&publications, &input.species_ids);
    let understudied = find_understudied_species(&coverage, input.understudied_threshold);
    let graph = build_knowledge_graph(&publications);

    Ok(KnowledgeReport {
        ranked_publications: ranked,
        timeline,
        coverage,
        understudied,
        graph,
    })
}
```

Pipeline konseptual:

```text
Raw Publications
  ↓
Parsing, Normalization & Validation
  ↓
Deduplication & Indexing
  ↓
Species / Topic / Location Query
  ↓
Timeline & Coverage Analysis
  ↓
Ranking & Citation Recommendation
  ↓
Knowledge Graph + Citation Export
```

Fungsi `explore_knowledge` menjadi entry point modul dan menggabungkan fungsi-fungsi murni dengan adapter I/O yang dipisahkan pada batas integrasi.

---

## 9. Prinsip Functional Programming yang Perlu Dipegang Tim

### Pure Functions

Parsing, filtering, scoring, agregasi, graph construction, dan formatting harus menghasilkan output hanya berdasarkan inputnya. Network request pada `fetch_publications` dipisahkan sebagai fungsi I/O.

### Immutability

Jangan memodifikasi koleksi publikasi sumber. Gunakan reference untuk membaca data dan kembalikan koleksi baru untuk hasil filter, ranking, atau agregasi.

### Higher-Order Functions

Gunakan `.map()`, `.filter()`, `.fold()`, `.flat_map()`, `.group_by()` atau pola iterator Rust yang sesuai untuk transformasi dan agregasi.

### Function Composition

Pertahankan pipeline yang jelas:

```text
parse → normalize → deduplicate → query → analyze → rank → export
```

Setiap tahap harus dapat diuji tanpa harus menjalankan database atau service eksternal.

---

## 10. Pembagian Kerja - 5 Anggota

| # | Tahap | Fungsi / Tanggung Jawab Utama | PIC | Status |
| --- | --- | --- | --- | --- |
| 1 | Ingestion & Validation | `parse_publications`, `normalize_publication`, `deduplicate_publications` | | Belum dimulai |
| 2 | Explorers | Query spesies, takson, topik, dan lokasi | | Belum dimulai |
| 3 | Timeline & Coverage | Timeline, distribusi, coverage, understudied explorer | | Belum dimulai |
| 4 | Citation | Ranking, recommendation, APA, BibTeX export | | Belum dimulai |
| 5 | Integration | Knowledge graph, adapter API, pipeline, integration tests | | Belum dimulai |

Setiap PIC bertanggung jawab terhadap implementasi, unit test, dokumentasi, dan menjaga signature yang telah disepakati.

---

## 11. Kesepakatan Antaranggota

Sebelum implementasi dimulai, seluruh anggota perlu menyepakati:

* Format sumber data publikasi: JSON, CSV, API, atau gabungan.
* Field minimum yang wajib dimiliki setiap publikasi.
* Cara mencocokkan publikasi dengan species ID dan nama takson.
* Kosakata resmi topik penelitian.
* Standar penamaan dan hierarki lokasi.
* Definisi coverage dan threshold understudied.
* Formula ranking dan bobot setiap sinyal relevansi.
* Penanganan publikasi duplikat tanpa DOI.
* Format citation export yang wajib didukung.
* Batas antara data mentah, fungsi analitik, dan adapter I/O.

---

## 12. Kriteria Selesai Modul

* [ ] Data publikasi dapat diparse dari fixture lokal.
* [ ] Data publikasi tervalidasi, dinormalisasi, dan dideduplikasi.
* [ ] Species-to-publication explorer berjalan.
* [ ] Research topic explorer berjalan.
* [ ] Research location explorer berjalan.
* [ ] Species research timeline dapat dihasilkan.
* [ ] Research coverage analysis dapat membandingkan entitas.
* [ ] Understudied species explorer menghasilkan hasil berdasarkan threshold yang terdokumentasi.
* [ ] Citation recommendation menghasilkan ranking deterministik.
* [ ] Citation export mendukung minimal APA dan BibTeX.
* [ ] Knowledge network menghasilkan node dan edge yang valid.
* [ ] Setiap fungsi inti memiliki unit test.
* [ ] Pipeline dapat berjalan tanpa service eksternal menggunakan dummy fixture.
* [ ] Adapter sumber publikasi dapat diuji secara terpisah dari fungsi murni.

---

## 13. Contoh Skenario Pengujian

### Skenario 1 - Species-to-Publication Explorer

**Input:** species ID `4`.

**Expected:** hanya publikasi yang memuat species ID `4`, terurut berdasarkan skor relevansi atau tahun sesuai query.

### Skenario 2 - Research Topic Explorer

**Input:** topik `Conservation`.

**Expected:** semua publikasi bertopik konservasi, tanpa mengubah data sumber.

### Skenario 3 - Research Location Explorer

**Input:** lokasi `Kalimantan Barat`.

**Expected:** publikasi yang memiliki lokasi studi di provinsi tersebut atau lokasi turunannya.

### Skenario 4 - Coverage dan Understudied Species

**Input:** daftar species ID dan threshold `1`.

**Expected:** spesies dengan nol atau satu publikasi masuk daftar understudied secara deterministik.

### Skenario 5 - Citation Export

**Input:** satu publikasi dengan penulis, tahun, judul, venue, dan DOI.

**Expected:** output APA dan BibTeX berisi metadata yang sama serta dapat diproses sebagai teks.

### Skenario 6 - Knowledge Network

**Input:** dua publikasi yang memiliki spesies atau topik yang sama.

**Expected:** graph memiliki node publikasi dan edge relasi yang sesuai, tanpa edge duplikat.
