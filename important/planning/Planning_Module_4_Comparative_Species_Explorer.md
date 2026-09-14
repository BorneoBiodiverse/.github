# Planning Function — Modul 4: Comparative Species Explorer

**Mata Kuliah:** Pemrograman Fungsional  
**Bahasa:** Rust  
**Cakupan:** Pembangunan sistem perbandingan beberapa spesies di Kalimantan berdasarkan data taksonomi, atribut spesies, status konservasi, serta data observasi dan distribusi geografis untuk mengidentifikasi persamaan, perbedaan, karakteristik unik, dan tingkat kemiripan antarspesies.

**Repository:** `kalimantanbio-modul4-comparative-explorer`

**Sumber data:** `kalimantan-bio-demo-api` — endpoint spesies, taksonomi, dan observasi.

> **Catatan integrasi:** API demo menyediakan data spesies dengan atribut `species_type`, `iucn`, `cites`, `p106`, taksonomi lengkap (kingdom → genus), serta data observasi per kabupaten. Atribut seperti habitat/morphology tidak tersedia di API aktual; scope perbandingan disesuaikan dengan data yang benar-benar ada.

---

## 1. Tujuan Modul

Modul ini bertujuan untuk membangun sistem perbandingan spesies yang memungkinkan pengguna memilih beberapa spesies di Kalimantan dan melihat persamaan, perbedaan, karakteristik unik, status konservasi, dan pola distribusi geografis berdasarkan data yang tersedia.

Modul ini bertanggung jawab untuk:

* Membandingkan beberapa spesies berdasarkan informasi taksonomi dan atribut spesies yang tersedia.
* Mengidentifikasi atribut yang sama dan atribut yang berbeda atau unik antar spesies.
* Mengidentifikasi karakteristik yang paling membedakan satu spesies dengan spesies lainnya.
* Membandingkan status konservasi (`iucn`, `cites`, `p106`) dan tipe spesies antar spesies.
* Menghitung skor kemiripan antarspesies berdasarkan taksonomi, status konservasi, tipe spesies, dan distribusi observasi.
* Membandingkan distribusi geografis berdasarkan data observasi per kabupaten.
* Menemukan spesies lain yang memiliki tingkat kemiripan tinggi dengan spesies tertentu.
* Menghasilkan hasil perbandingan dalam bentuk data terstruktur yang dapat digunakan untuk visualisasi (tabel, matriks atribut, dsb.).
* Menghasilkan ringkasan tekstual yang menjelaskan persamaan dan perbedaan utama antarspesies.

### Batasan Modul

Modul ini **tidak bertanggung jawab** terhadap:

* Pencarian berbasis bahasa natural atau fuzzy search (tanggung jawab Modul 1).
* Visualisasi jaringan/graf relasi antar-spesies (tanggung jawab Modul 2).
* Pembangunan pohon taksonomi hirarkis dan analisis taxonomic gap (tanggung jawab Modul 3).
* Eksplorasi publikasi ilmiah dan manajemen sitasi (tanggung jawab Modul 5).
* Menyimpan atau mengubah data spesies — modul bersifat read-only terhadap data.
* Inferensi ekologis di luar data yang tersedia (mis. hubungan predator-mangsa).

---

## 2. Struktur Data (Domain Model)

Struktur data di bawah disusun berdasarkan response nyata dari `kalimantan-bio-demo-api` dan kebutuhan komparasi modul ini.

```rust
#[derive(Debug, Clone)]
struct Taxonomy {
    kingdom: String,
    kingdom_id: u64,
    phylum_division: String,
    phylum_division_id: u64,
    class: String,
    class_id: u64,
    order: String,
    order_id: u64,
    family: String,
    family_id: u64,
    genus: String,
    genus_id: u64,
}

#[derive(Debug, Clone)]
struct Species {
    id: u64,
    scientific_name: String,
    common_name: String,
    description: String,
    species_type: String,        // "Endemic", "Native", "Introduced"
    iucn: String,                // "CR", "EN", "VU", "NT", "LC", "DD", "-"
    cites: String,               // "Appendix I", "Appendix II", "Appendix III", "-"
    p106: String,                // "Dilindungi", "Tidak Dilindungi", "-"
    is_verified: bool,
    image_url: String,
    ai_key: i32,
    genus_id: u64,
    taxonomy: Taxonomy,
    observation_count: u32,
    recorded_individuals_total: u32,
    latest_observation_year: Option<u32>,
}

#[derive(Debug, Clone)]
struct Observation {
    id: u64,
    species_id: u64,
    latitude: f64,
    longitude: f64,
    date: String,
    count: i32,
    is_verified: bool,
    source_method: String,
    kabupaten_id: u64,
    kabupaten_name: String,
    province_name: String,
}

#[derive(Debug, Clone, Default)]
struct ComparisonQuery {
    species_ids: Vec<u64>,   // minimal 2 spesies, maksimal disepakati tim (mis. 5)
}

#[derive(Debug, Clone)]
struct ScoringWeights {
    taxonomy: f64,      // default: 0.40
    conservation: f64,  // default: 0.30
    species_type: f64,  // default: 0.15
    distribution: f64,  // default: 0.15
}

#[derive(Debug, Clone)]
struct SimilarityScore {
    species_a_id: u64,
    species_b_id: u64,
    taxonomy_score: f64,       
    conservation_score: f64,   
    type_score: f64,           
    distribution_score: f64,   
    total_score: f64,          
}

#[derive(Debug, Clone)]
struct SpeciesAttributeRow {
    species_id: u64,
    scientific_name: String,
    common_name: String,
    taxonomy_path: String,    
    species_type: String,
    iucn: String,
    cites: String,
    p106: String,
    observation_count: u32,
    kabupaten_list: Vec<String>,
    is_verified: bool,
}

#[derive(Debug, Clone, Default)]
struct ComparisonResult {
    query: ComparisonQuery,
    attribute_matrix: Vec<SpeciesAttributeRow>, 
    shared_attributes: Vec<String>,              
    unique_attributes: Vec<String>,               
    distinguishing_characteristics: Vec<String>, 
    similarity_scores: Vec<SimilarityScore>,      
    summary: String,                               
}
```

> **Catatan:** Struktur data perlu disepakati oleh seluruh anggota kelompok sebelum implementasi karena struktur ini menjadi dasar bagi fungsi-fungsi pada tahap berikutnya. `taxonomy_score` menggunakan level kesamaan takson: genus=1.0, family=0.8, order=0.6, class=0.4, phylum=0.2, kingdom=0.1, tidak ada kesamaan=0.0. Total bobot `ScoringWeights` harus = 1.0.

### Data yang Digunakan

| Data | Tipe | Deskripsi |
| --------- | -------- | ----------- |
| `scientific_name`, `common_name`, `description` | `String` | Informasi identitas dan deskripsi spesies yang digunakan sebagai atribut pembanding. |
| `species_type` | `String` | Menunjukkan kategori spesies, seperti Endemic, Native, atau Introduced. |
| `iucn`, `cites`, `p106` | `String` | Informasi status konservasi spesies berdasarkan status IUCN, CITES, dan perlindungan nasional. |
| `taxonomy` | `Taxonomy` | Informasi klasifikasi taksonomi spesies, meliputi kingdom, phylum/division, class, order, family, dan genus. |
| `is_verified` | `bool` | Menunjukkan apakah data spesies telah diverifikasi. |
| `observation_count` | `u32` | Jumlah observasi yang tercatat untuk suatu spesies. |
| `recorded_individuals_total` | `u32` | Total individu spesies yang tercatat dalam data observasi. |
| `latest_observation_year` | `Option<u32>` | Tahun observasi terbaru yang tersedia untuk suatu spesies. |
| `Observation.kabupaten_id`, `kabupaten_name` | `u64` / `String` | Data distribusi kabupaten untuk perbandingan geografis antar spesies. |

---

## 3. Tahap 1 — Data Retrieval & Validation

Mengambil data spesies dari sumber data (API atau fixture), memvalidasi input query pengguna, dan menyiapkan data yang siap diproses oleh tahap berikutnya.

| Fungsi | Signature | Deskripsi |
| -------------- | ----------------------------- | ------------------ |
| `validate_query` | `fn validate_query(query: &ComparisonQuery) -> Result<(), QueryError>` | Memvalidasi query: minimal 2 ID spesies, tidak ada duplikasi, tidak melebihi batas maksimal yang disepakati. Mengembalikan `QueryError::TooFewSpecies`, `QueryError::DuplicateId`, atau `QueryError::TooManySpecies` sesuai kondisi. |
| `fetch_species_by_ids` | `fn fetch_species_by_ids<'a>(ids: &[u64], all_species: &'a [Species]) -> Result<Vec<&'a Species>, DataError>` | Mengambil spesies berdasarkan ID dari slice data yang tersedia menggunakan `.filter()`. Mengembalikan `DataError::SpeciesNotFound(id)` jika ada ID yang tidak ditemukan. |
| `fetch_observations_for_species` | `fn fetch_observations_for_species(species_id: u64, all_observations: &[Observation]) -> Vec<&Observation>` | Mengambil semua data observasi untuk satu spesies menggunakan `.filter()`. Mengembalikan slice kosong jika tidak ada observasi. |
| `build_kabupaten_set` | `fn build_kabupaten_set(observations: &[&Observation]) -> std::collections::HashSet<u64>` | Mengekstrak set unik `kabupaten_id` dari observasi suatu spesies menggunakan `.map().collect()`. |

**Person in Charge:** **(isi nama anggota)**

---

## 4. Tahap 2 — Attribute Matrix Construction

Membangun matriks atribut per spesies sebagai representasi terstruktur dari setiap spesies yang dibandingkan, termasuk menyusun taxonomy path yang mudah dibaca.

| Fungsi | Signature | Deskripsi |
| -------------- | ----------------------------- | ------------------ |
| `build_taxonomy_path` | `fn build_taxonomy_path(taxonomy: &Taxonomy) -> String` | Menyusun string path taksonomi menggunakan format `"Kingdom > Phylum > Class > Order > Family > Genus"` (mis. `"Animalia > Chordata > Mammalia > Primates > Hominidae > Pongo"`). |
| `build_attribute_row` | `fn build_attribute_row(species: &Species, observations: &[&Observation]) -> SpeciesAttributeRow` | Membangun satu baris atribut spesies termasuk taxonomy path dan daftar nama kabupaten observasi unik. |
| `build_attribute_matrix` | `fn build_attribute_matrix(species_list: &[&Species], all_observations: &[Observation]) -> Vec<SpeciesAttributeRow>` | Menerapkan `build_attribute_row` ke setiap spesies menggunakan `.map().collect()` untuk menghasilkan matriks atribut lengkap. |
| `extract_attribute_values` | `fn extract_attribute_values(matrix: &[SpeciesAttributeRow], key: &str) -> Vec<String>` | Mengekstrak nilai satu atribut tertentu (mis. `"iucn"`, `"species_type"`) dari seluruh baris matriks, untuk digunakan pada analisis kesamaan dan perbedaan. |

**Person in Charge:** **(isi nama anggota)**

---

## 5. Tahap 3 — Shared & Unique Attribute Analysis

Menganalisis matriks atribut untuk menemukan atribut yang sama di semua spesies, atribut yang unik per spesies, dan karakteristik paling membedakan.

| Fungsi | Signature | Deskripsi |
| -------------- | ----------------------------- | ------------------ |
| `is_attribute_shared` | `fn is_attribute_shared(values: &[String]) -> bool` | Pure function: mengembalikan `true` jika semua nilai dalam slice identik (tidak kosong). Diimplementasikan menggunakan `.windows(2).all(|w| w[0] == w[1])`. |
| `find_shared_attributes` | `fn find_shared_attributes(matrix: &[SpeciesAttributeRow]) -> Vec<String>` | Menemukan atribut (taksonomi, iucn, cites, p106, species_type) yang bernilai sama pada semua spesies menggunakan `extract_attribute_values` dan `is_attribute_shared`. Hasilnya dalam format `"key=value"` (mis. `"kingdom=Animalia"`, `"iucn=CR"`). |
| `find_unique_attributes` | `fn find_unique_attributes(matrix: &[SpeciesAttributeRow]) -> Vec<String>` | Menemukan atribut yang nilainya hanya dimiliki tepat satu spesies dalam kumpulan yang dibandingkan. Hasilnya dalam format `"species_id:key=value"` (mis. `"123:species_type=Endemic"`). |
| `find_distinguishing_characteristics` | `fn find_distinguishing_characteristics(matrix: &[SpeciesAttributeRow]) -> Vec<String>` | Mengidentifikasi atribut dengan variasi terbesar antarspesies — yaitu atribut paling membedakan. Diprioritaskan: iucn > species_type > family > order > class > phylum. Hasilnya berupa deskripsi teks (mis. `"iucn: CR vs LC vs VU"`). |

**Person in Charge:** **(isi nama anggota)**

---

## 6. Tahap 4 — Similarity Scoring

Menghitung skor kemiripan untuk setiap pasang spesies yang dibandingkan berdasarkan empat dimensi: taksonomi, konservasi, tipe spesies, dan distribusi geografis.

| Fungsi | Signature | Deskripsi |
| -------------- | ----------------------------- | ------------------ |
| `score_taxonomy` | `fn score_taxonomy(a: &Taxonomy, b: &Taxonomy) -> f64` | Menghitung skor taksonomi (0.0–1.0) berdasarkan level takson tertinggi yang sama. Skala: genus=1.0, family=0.8, order=0.6, class=0.4, phylum=0.2, kingdom=0.1, tidak ada kesamaan=0.0. Diimplementasikan dengan pengecekan bertahap dari bawah ke atas menggunakan perbandingan ID. |
| `score_conservation` | `fn score_conservation(a: &Species, b: &Species) -> f64` | Menghitung rata-rata kesamaan tiga atribut konservasi (`iucn`, `cites`, `p106`). Tiap atribut: 1.0 jika identik, 0.5 jika salah satu bernilai `"-"`, 0.0 jika berbeda. Rata-rata dari ketiga nilai. |
| `score_species_type` | `fn score_species_type(a: &Species, b: &Species) -> f64` | Mengembalikan 1.0 jika `species_type` identik (case-insensitive), 0.0 jika berbeda. |
| `score_distribution` | `fn score_distribution(obs_a: &std::collections::HashSet<u64>, obs_b: &std::collections::HashSet<u64>) -> f64` | Menghitung Jaccard Similarity dari set kabupaten observasi: `|intersection| / |union|`. Mengembalikan 0.0 jika salah satu atau keduanya kosong (menghindari division by zero). |
| `calculate_similarity_score` | `fn calculate_similarity_score(a: &Species, b: &Species, obs_a: &std::collections::HashSet<u64>, obs_b: &std::collections::HashSet<u64>, weights: &ScoringWeights) -> SimilarityScore` | Menggabungkan keempat skor dengan bobot dari `weights` menjadi `total_score` menggunakan weighted sum. |
| `calculate_all_pairs` | `fn calculate_all_pairs(species_list: &[&Species], observations_map: &std::collections::HashMap<u64, std::collections::HashSet<u64>>, weights: &ScoringWeights) -> Vec<SimilarityScore>` | Menghasilkan semua kombinasi pasang spesies (n*(n-1)/2) dan menghitung skor tiap pasang. Menggunakan nested iterator dengan `.enumerate().flat_map()`. |
| `find_most_similar` | `fn find_most_similar(target_id: u64, scores: &[SimilarityScore]) -> Option<SimilarityScore>` | Menemukan pasang dengan `total_score` tertinggi yang melibatkan `target_id` menggunakan `.filter().max_by()`. Mengembalikan `None` jika tidak ada pasang yang ditemukan. |

**Person in Charge:** **(isi nama anggota)**

---

## 7. Tahap 5 — Summary Generation & Integration Pipeline

Menggabungkan seluruh output tahap sebelumnya menjadi `ComparisonResult` lengkap dan menghasilkan ringkasan teks yang menjelaskan persamaan dan perbedaan utama antarspesies.

| Fungsi / Komponen | Signature / Bentuk | Deskripsi |
| ----------------- | ------------------ | ----------- |
| `generate_summary` | `fn generate_summary(result: &ComparisonResult) -> String` | Menghasilkan ringkasan teks perbandingan dari `shared_attributes`, `unique_attributes`, `distinguishing_characteristics`, dan `similarity_scores` tertinggi. Pure function, tidak melakukan I/O. |
| `compare_species` | `fn compare_species(query: &ComparisonQuery, all_species: &[Species], all_observations: &[Observation], weights: &ScoringWeights) -> Result<ComparisonResult, ComparisonError>` | Entry point pipeline utama: merangkai Tahap 1–4 menjadi `ComparisonResult` lengkap. |
| Unit Tests | `mod tests { ... }` | Menguji setiap fungsi dari Tahap 1–4 secara independen menggunakan data dummy `Vec<Species>` dan `Vec<Observation>`. |
| Pipeline Validation | `cargo test` | Menjalankan seluruh skenario pengujian secara otomatis termasuk edge cases. |

**Person in Charge:** **(isi nama anggota)**

> **Catatan:** Setiap PIC tetap bertanggung jawab terhadap pengujian fungsi yang mereka implementasikan. PIC tahap ini berfokus pada pengujian antar-komponen dan pengujian end-to-end.

---

## 8. Komposisi Pipeline Utama

Setelah seluruh tahap tersedia, fungsi utama modul menggabungkan proses menjadi satu pipeline.

```rust
fn compare_species(
    query: &ComparisonQuery,
    all_species: &[Species],
    all_observations: &[Observation],
    weights: &ScoringWeights,
) -> Result<ComparisonResult, ComparisonError> {
    validate_query(query)?;
    let species_list = fetch_species_by_ids(&query.species_ids, all_species)?;
    let observations_map: HashMap<u64, HashSet<u64>> = species_list
        .iter()
        .map(|s| {
            let obs = fetch_observations_for_species(s.id, all_observations);
            let kabupaten_set = build_kabupaten_set(&obs);
            (s.id, kabupaten_set)
        })
        .collect();

    let attribute_matrix = build_attribute_matrix(&species_list, all_observations);

    let shared_attributes = find_shared_attributes(&attribute_matrix);
    let unique_attributes = find_unique_attributes(&attribute_matrix);
    let distinguishing_characteristics = find_distinguishing_characteristics(&attribute_matrix);

    let similarity_scores = calculate_all_pairs(&species_list, &observations_map, weights);

    let mut result = ComparisonResult {
        query: query.clone(),
        attribute_matrix,
        shared_attributes,
        unique_attributes,
        distinguishing_characteristics,
        similarity_scores,
        summary: String::new(),
    };
    result.summary = generate_summary(&result);

    Ok(result)
}
```

Pipeline konseptual:

```text
ComparisonQuery (species_ids)
  |
[Tahap 1: Data Retrieval & Validation]
  | Vec<&Species> + HashMap<u64, HashSet<u64>>
[Tahap 2: Attribute Matrix Construction]
  | Vec<SpeciesAttributeRow>
[Tahap 3: Shared & Unique Attribute Analysis]
  | shared_attributes, unique_attributes, distinguishing_characteristics
[Tahap 4: Similarity Scoring]
  | Vec<SimilarityScore>
[Tahap 5: Summary Generation & Assembly]
  |
ComparisonResult (output lengkap)
```

Fungsi `compare_species` adalah **entry point** modul dan menjadi contoh penerapan **function composition** — menggabungkan beberapa fungsi kecil bertanggung jawab spesifik menjadi satu proses besar.

---

## 9. Prinsip Functional Programming yang Perlu Dipegang Tim

### Pure Functions

Fungsi sebaiknya tidak mengubah state eksternal dan hanya bergantung pada input yang diberikan.

```text
Input -> Function -> Output
```

Untuk input yang sama, fungsi idealnya menghasilkan output yang sama. `compare_species` merupakan pengecualian wajar jika melibatkan I/O (network call ke API) — sebaiknya dipisahkan tegas dari fungsi-fungsi murni lainnya di pipeline.

### Immutability

Hindari memodifikasi data input secara langsung.

Gunakan reference/borrow seperti `&T` ketika data hanya perlu dibaca dan hasil transformasi dikembalikan sebagai data baru apabila diperlukan.

### Higher-Order Functions

Manfaatkan iterator dan fungsi seperti:

```rust
.map()
.filter()
.fold()
.flat_map()
.all()
.any()
.max_by()
.collect()
```

untuk melakukan transformasi, filtering, dan agregasi data. Hindari `for` loop manual di logika inti analitik.

### Function Composition

Pecah proses utama menjadi fungsi-fungsi kecil yang dapat dikombinasikan.

```text
validate_query
    |
fetch_species_by_ids
    |
build_attribute_matrix
    |
find_shared_attributes / find_unique_attributes / find_distinguishing_characteristics
    |
calculate_all_pairs
    |
generate_summary
```

Setiap fungsi sebaiknya memiliki satu tanggung jawab yang jelas dan dapat diuji secara independen.

---

## 10. Pembagian Kerja — 5 Anggota

| # | Tahap | Fungsi / Tanggung Jawab Utama | PIC | Status |
| - | --------- | ------------------------------------- | --- | ------------- |
| 1 | Tahap 1: Data Retrieval & Validation | `validate_query`, `fetch_species_by_ids`, `fetch_observations_for_species`, `build_kabupaten_set` | | Belum dimulai |
| 2 | Tahap 2: Attribute Matrix Construction | `build_taxonomy_path`, `build_attribute_row`, `build_attribute_matrix`, `extract_attribute_values` | | Belum dimulai |
| 3 | Tahap 3: Shared & Unique Attribute Analysis | `is_attribute_shared`, `find_shared_attributes`, `find_unique_attributes`, `find_distinguishing_characteristics` | | Belum dimulai |
| 4 | Tahap 4: Similarity Scoring | `score_taxonomy`, `score_conservation`, `score_species_type`, `score_distribution`, `calculate_similarity_score`, `calculate_all_pairs`, `find_most_similar` | | Belum dimulai |
| 5 | Tahap 5: Integration & Testing | `generate_summary`, `compare_species`, unit tests, pipeline validation | | Belum dimulai |

### Pembagian Tanggung Jawab

Setiap PIC bertanggung jawab terhadap:

* Implementasi fungsi yang ditugaskan.
* Unit testing fungsi tersebut.
* Dokumentasi fungsi.
* Menjaga signature/interface yang telah disepakati.
* Melaporkan perubahan yang dapat memengaruhi komponen lain.

---

## 11. Kesepakatan Antaranggota

Sebelum implementasi dimulai, seluruh anggota perlu menyepakati:

* Struktur `Species`, `Taxonomy`, `Observation`, `ComparisonQuery`, `ScoringWeights`, `SimilarityScore`, dan `ComparisonResult`.
* Arti setiap field, terutama field yang berasal dari singkatan (`iucn`, `cites`, `p106`).
* Jumlah maksimal spesies yang dapat dibandingkan sekaligus dalam satu `ComparisonQuery`.
* Input dan output setiap fungsi, termasuk format string `shared_attributes` dan `unique_attributes`.
* Function signature final untuk setiap tahap.
* Formula dan bobot scoring pada `ScoringWeights` (default: taxonomy=0.40, conservation=0.30, type=0.15, distribution=0.15).
* Formula `score_taxonomy`: level takson yang sama (genus=1.0, family=0.8, order=0.6, class=0.4, phylum=0.2, kingdom=0.1).
* Formula `score_distribution`: Jaccard Similarity dari set kabupaten observasi.
* Format `shared_attributes` (mis. `"iucn=CR"`) dan `unique_attributes` (mis. `"123:species_type=Endemic"`).
* Format output `summary` (teks bebas atau template terstruktur).
* Strategi testing (unit test per fungsi dengan data dummy, tanpa koneksi API aktif).

Tujuannya adalah memastikan fungsi yang dikembangkan oleh anggota berbeda tetap dapat dikombinasikan tanpa perubahan besar pada interface masing-masing.

---

## 12. Independensi Modul

Modul ini dikembangkan sebagai komponen independen dalam KalimantanBio.

Prinsip yang digunakan:

* Modul dapat dikembangkan secara mandiri, terlepas dari status Modul 1, 2, 3, dan 5.
* Modul dapat diuji secara mandiri menggunakan data dummy `Vec<Species>` dan `Vec<Observation>`, tanpa harus terkoneksi ke API.
* Modul tidak boleh bergantung pada implementasi internal modul lain.
* Gunakan shared concepts (`Species`, `Taxonomy`) sebagai data convention yang disetujui bersama.
* Integrasi dengan modul lain bersifat opsional — misalnya hasil pencarian Modul 1 dapat digunakan sebagai input `species_ids` ke Modul 4.
* Jangan mengasumsikan dependency terhadap modul lain tanpa kebutuhan teknis yang jelas.

```text
              KalimantanBio
                   |
          .------.-.------.
          |       |        |
         M1      M2       M3
          |       |        |
         M4      M5
          |       |
          .-------.---------.
                            |
                    Optional Integration
```

Diagram di atas menggambarkan hubungan **konseptual**, bukan dependency teknis.

---

## 13. Kriteria Selesai Modul

Modul dianggap siap untuk tahap akhir apabila:

* [ ] Seluruh fungsi utama telah diimplementasikan.
* [ ] Setiap fungsi memiliki unit test yang relevan.
* [ ] Pipeline `compare_species()` dapat berjalan end-to-end dengan data dummy maupun data dari API.
* [ ] Input `ComparisonQuery` dapat diproses sesuai spesifikasi, termasuk validasi jumlah spesies.
* [ ] Output menghasilkan `ComparisonResult` dengan semua field terisi sesuai format yang disepakati.
* [ ] `SimilarityScore` menghasilkan nilai dalam rentang 0.0-1.0 untuk semua dimensi dan total skor.
* [ ] Tidak terdapat dependency yang tidak diperlukan (tidak bergantung langsung ke Modul 1, 2, 3, atau 5).
* [ ] Tidak terdapat state global yang tidak diperlukan.
* [ ] Dokumentasi fungsi dan struktur data tersedia.
* [ ] Terdapat demonstrasi penggunaan modul (contoh query dengan 2-3 spesies dan hasilnya).
* [ ] Modul dapat dijalankan secara independen, termasuk fallback jika API tidak tersedia.

---

## 14. Contoh Skenario Pengujian

### Skenario 1 — Membandingkan Dua Spesies Orangutan

**Input:**

```text
ComparisonQuery { species_ids: [1, 2] }
Spesies 1: Pongo pygmaeus (Orangutan Kalimantan, Endemic, iucn=CR, cites="Appendix I", p106="Dilindungi")
Spesies 2: Pongo abelii   (Orangutan Sumatera, Native,   iucn=CR, cites="Appendix I", p106="Dilindungi")
```

**Expected Output:**

```text
ComparisonResult {
  shared_attributes: [
    "kingdom=Animalia", "iucn=CR", "cites=Appendix I",
    "p106=Dilindungi", "taxonomy.genus=Pongo"
  ],
  unique_attributes: [
    "1:common_name=Orangutan Kalimantan", "1:species_type=Endemic",
    "2:common_name=Orangutan Sumatera",   "2:species_type=Native"
  ],
  distinguishing_characteristics: [
    "species_type: Endemic vs Native",
    "Distribusi kabupaten observasi berbeda"
  ],
  similarity_scores: [
    SimilarityScore {
      species_a_id: 1, species_b_id: 2,
      taxonomy_score: 1.0,     // genus sama (Pongo)
      conservation_score: 1.0, // iucn=CR, cites=Appendix I, p106=Dilindungi semua sama
      type_score: 0.0,         // Endemic != Native
      distribution_score: 0.2, // sedikit overlap kabupaten observasi (contoh)
      total_score: 0.79,       // 1.0*0.40 + 1.0*0.30 + 0.0*0.15 + 0.2*0.15
    }
  ],
  summary: "Kedua spesies termasuk genus yang sama (Pongo) dengan status konservasi identik (IUCN CR, CITES Appendix I, Dilindungi). Perbedaan utama: tipe spesies (Endemic vs Native) dan distribusi observasi yang berbeda."
}
```

**Fungsi yang diuji:**

* `validate_query`, `fetch_species_by_ids`
* `build_taxonomy_path`, `build_attribute_matrix`
* `find_shared_attributes`, `find_unique_attributes`, `find_distinguishing_characteristics`
* `score_taxonomy`, `score_conservation`, `score_species_type`, `score_distribution`
* `calculate_similarity_score`, `generate_summary`

---

### Skenario 2 — Membandingkan Tiga Spesies dari Kingdom Berbeda

**Input:**

```text
ComparisonQuery { species_ids: [10, 20, 30] }
Spesies 10: Rafflesia arnoldii  (Plantae, Endemic, iucn=CR)
Spesies 20: Pongo pygmaeus      (Animalia, Endemic, iucn=CR)
Spesies 30: Agathis borneensis  (Plantae, Endemic, iucn=EN)
```

**Expected Output:**

```text
ComparisonResult {
  shared_attributes: ["species_type=Endemic"],
  unique_attributes: [
    "10:kingdom=Plantae", "10:iucn=CR",
    "20:kingdom=Animalia", "20:iucn=CR",
    "30:kingdom=Plantae", "30:iucn=EN"
  ],
  distinguishing_characteristics: [
    "kingdom: Plantae vs Animalia vs Plantae",
    "iucn: CR vs CR vs EN"
  ],
  similarity_scores: [
    // Pasang 10-20: taxonomy_score=0.0 (kingdom berbeda), conservation beda
    // Pasang 10-30: taxonomy_score=0.2 (phylum sama: Tracheophyta)
    // Pasang 20-30: taxonomy_score=0.0 (kingdom berbeda)
    ...
  ],
  summary: "Ketiga spesies sama-sama berstatus Endemic. Spesies 10 dan 30 berasal dari Kingdom Plantae sedangkan Spesies 20 dari Animalia. Perbedaan utama terletak pada kingdom dan status IUCN."
}
```

**Fungsi yang diuji:**

* `calculate_all_pairs` (3 kombinasi pasang: 10-20, 10-30, 20-30)
* `find_shared_attributes` dengan 3 baris matriks
* `find_distinguishing_characteristics`

---

### Skenario 3 — Menemukan Spesies Paling Mirip dengan Target

**Input:**

```text
target_id: 1
scores: [semua SimilarityScore dari dataset]
```

**Expected Output:**

```text
Some(SimilarityScore { ... total_score: <nilai tertinggi yang melibatkan id=1> })
```

**Fungsi yang diuji:**

* `find_most_similar`

---

### Edge Cases

| Case | Input | Expected Result |
| --- | --- | --- |
| Hanya 1 ID spesies | `species_ids: [1]` | `validate_query` mengembalikan `Err(QueryError::TooFewSpecies)`. |
| ID spesies tidak ditemukan | `species_ids: [1, 99999]` | `fetch_species_by_ids` mengembalikan `Err(DataError::SpeciesNotFound(99999))`. |
| ID duplikat | `species_ids: [1, 1]` | `validate_query` mengembalikan `Err(QueryError::DuplicateId)`. |
| Spesies tanpa data observasi | Spesies dengan `observation_count=0` | `build_kabupaten_set` mengembalikan `HashSet` kosong; `score_distribution` mengembalikan `0.0` tanpa panic. |
| Semua spesies identik | Dua spesies dengan semua atribut identik | `shared_attributes` berisi semua atribut; `unique_attributes` kosong; `total_score = 1.0`. |
| Field opsional null | `latest_observation_year: None` | Pipeline tetap berjalan tanpa panic, menggunakan `Option<T>` dan `.unwrap_or`. |
| Melebihi batas maksimal | `species_ids: [1,2,3,4,5,6]` (jika max=5) | `validate_query` mengembalikan `Err(QueryError::TooManySpecies)`. |
| Satu spesies tanpa observasi, satu ada | obs_a kosong, obs_b tidak kosong | `score_distribution` mengembalikan `0.0` (union tidak kosong, intersection kosong). |

---

## 15. Langkah Selanjutnya

1. Finalisasi domain model bersama seluruh anggota (`Species`, `Taxonomy`, `Observation`, `ScoringWeights`, dan semua struct terkait).
2. Sepakati nilai default bobot scoring dan formula untuk setiap dimensi skor.
3. Sepakati batas maksimal spesies yang dapat dibandingkan sekaligus dalam satu `ComparisonQuery`.
4. Sepakati format string untuk `shared_attributes`, `unique_attributes`, dan `distinguishing_characteristics`.
5. Konfirmasi field `Species` yang akan digunakan (pastikan sesuai dengan response aktual `kalimantan-bio-demo-api`).
6. Tentukan pembagian fungsi berdasarkan PIC (isi tabel Bagian 10).
7. Setiap PIC membuat signature dan dokumentasi singkat fungsi masing-masing.
8. Review interface bersama sebelum implementasi logic dimulai.
9. Siapkan data dummy `Vec<Species>` dan `Vec<Observation>` (independen dari API) untuk unit test.
10. Implementasikan fungsi secara paralel sesuai pembagian kerja.
11. Setiap PIC membuat unit test untuk fungsi masing-masing.
12. Gabungkan seluruh tahap ke dalam pipeline `compare_species()`.
13. Lakukan pengujian end-to-end menggunakan data dummy dan kemudian data dari API.
14. Dokumentasikan hasil dan contoh penggunaan modul.
15. Review akhir sebelum modul dianggap selesai.
