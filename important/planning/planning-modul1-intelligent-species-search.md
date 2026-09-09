# Planning Function — Modul 1: Intelligent Species Search

**Mata Kuliah:** Pemrograman Fungsional
**Bahasa:** Rust
**Cakupan:** Natural-language species search, multi-attribute filtering, relevance ranking, related-query recommendation

**Repository:** `kalimantan-bio-demo-api` (sumber data, backend Python/FastAPI — modul Rust kita mengonsumsi data ini lewat REST API)

---

## 1. Tujuan Modul

Modul ini membangun sistem pencarian spesies yang memungkinkan pengguna mencari data biodiversitas menggunakan query bahasa natural maupun filter atribut spesifik, lalu menampilkan hasil yang diurutkan berdasarkan relevansi.

Modul ini bertanggung jawab untuk:

* Mem-parsing query bahasa natural menjadi filter terstruktur.
* Melakukan filtering spesies berdasarkan atribut yang tersedia (taksonomi, status konservasi, tipe spesies, status verifikasi).
* Menghitung skor relevansi dan mengurutkan (ranking) hasil pencarian.
* Menghasilkan rekomendasi query lanjutan berdasarkan hasil pencarian saat ini.

### Batasan Modul

Modul ini **tidak bertanggung jawab** terhadap:

* Menyimpan atau mengubah data spesies (itu tanggung jawab `kalimantan-bio-demo-api`).
* Visualisasi jaringan relasi antar-spesies (itu Modul 2).
* Eksplorasi hierarki taksonomi mendalam (itu Modul 3).
* Perbandingan antar-spesies secara detail (itu Modul 4).
* Pencarian berbasis publikasi ilmiah (itu Modul 5).

> **Catatan penting:** Deskripsi awal Modul 1 menyebutkan atribut seperti *habitat*, *morphology*, dan *uses*. Setelah dicoba langsung ke `kalimantan-bio-demo-api` (`GET /api/v1/species/{id}`), atribut-atribut tersebut **tidak tersedia** di data aktual. Atribut yang benar-benar ada: `species_type`, `iucn`, `cites`, `p106` (status perlindungan nasional), dan `taxonomy` lengkap. Perlu dikonfirmasi ke project lead apakah ini API sementara/demo saja, atau memang jadi sumber data final — supaya scope pencarian tidak salah asumsi.

---

## 2. Struktur Data (Domain Model)

Struktur data di bawah ini disusun berdasarkan response nyata dari `GET /api/v1/species/{id}` pada `kalimantan-bio-demo-api`, bukan asumsi awal.

```rust
#[derive(Debug, Clone, serde::Deserialize)]
struct Species {
    id: i64,
    scientific_name: String,
    common_name: String,
    description: String,
    species_type: String,               // contoh: "Endemic", "Native", "Introduced"
    iucn: String,                        // status IUCN, mis. "CR", "EN", "VU"
    cites: String,                       // status CITES, mis. "Appendix I"
    p106: String,                        // status perlindungan nasional, mis. "Dilindungi"
    is_verified: bool,
    image_url: Option<String>,
    taxonomy: Taxonomy,
    observation_count: u32,
    recorded_individuals_total: u32,
    latest_observation_year: Option<u32>,
    created_at: String,                  // RFC 3339 timestamp
    updated_at: String,
}

#[derive(Debug, Clone, serde::Deserialize)]
struct Taxonomy {
    kingdom: String,
    kingdom_id: i64,
    phylum_division: String,
    phylum_division_id: i64,
    class: String,
    class_id: i64,
    order: String,
    order_id: i64,
    family: String,
    family_id: i64,
    genus: String,
    genus_id: i64,
}

#[derive(Debug, Clone, Default)]
struct QueryFilters {
    free_text: Vec<String>,       // kata kunci umum, dicocokkan ke nama/deskripsi
    kingdom_id: Option<i64>,
    class_id: Option<i64>,
    iucn: Option<String>,
    cites: Option<String>,
    species_type: Option<String>,
    is_verified: Option<bool>,
    sort: Option<String>,
}
```

> **Catatan:** Struktur data perlu disepakati oleh seluruh anggota kelompok sebelum implementasi karena struktur ini menjadi dasar bagi fungsi-fungsi pada tahap berikutnya. Field ini bisa disederhanakan (mis. hapus `created_at`/`updated_at` jika tidak dipakai untuk pencarian).

### Data yang Digunakan

| Data | Tipe | Deskripsi |
|---|---|---|
| `scientific_name`, `common_name`, `description` | `String` | Sumber utama untuk pencocokan teks bebas (free-text search). |
| `species_type` | `String` | Filter kategori spesies (mis. endemik/asli/introduksi). |
| `iucn`, `cites`, `p106` | `String` | Atribut status konservasi — pengganti "conservation status" pada spesifikasi awal. |
| `taxonomy` | `Taxonomy` | Digunakan untuk filter & scoring berbasis kingdom/class/order/family/genus. |
| `is_verified` | `bool` | Menentukan apakah data sudah diverifikasi; bisa dipakai sebagai filter kualitas hasil. |
| `observation_count`, `recorded_individuals_total`, `latest_observation_year` | `u32` / `Option<u32>` | Bisa dipakai sebagai sinyal tambahan untuk relevansi (mis. spesies yang lebih banyak diobservasi mendapat skor lebih tinggi), opsional. |

---

## 3. Tahap 1 — Parsing & Query Understanding

Mengubah query bahasa natural (String) menjadi `QueryFilters` yang terstruktur.

| Fungsi | Signature | Deskripsi |
|---|---|---|
| `normalize_text` | `fn normalize_text(input: &str) -> String` | Lowercase, hapus tanda baca dan spasi berlebih. |
| `tokenize` | `fn tokenize(text: &str) -> Vec<String>` | Memecah teks menjadi daftar token. |
| `extract_filters` | `fn extract_filters(tokens: &[String]) -> QueryFilters` | Mencocokkan token terhadap kosakata dikenal (nama status IUCN, nama taksonomi umum, kata seperti "endemik"/"dilindungi") dan sisanya masuk `free_text`. |

**Person in Charge:** **(isi nama anggota)**

---

## 4. Tahap 2 — Multi-Attribute Filtering

Menyaring kandidat spesies berdasarkan `QueryFilters`, menggunakan atribut yang benar-benar tersedia di API.

| Fungsi | Signature | Deskripsi |
|---|---|---|
| `matches_free_text` | `fn matches_free_text(species: &Species, filters: &QueryFilters) -> bool` | Cek apakah `scientific_name`, `common_name`, atau `description` memuat kata kunci bebas. |
| `matches_taxonomy` | `fn matches_taxonomy(species: &Species, filters: &QueryFilters) -> bool` | Cek kecocokan `kingdom_id`/`class_id`. |
| `matches_conservation` | `fn matches_conservation(species: &Species, filters: &QueryFilters) -> bool` | Cek kecocokan `iucn`/`cites`. |
| `matches_species_type` | `fn matches_species_type(species: &Species, filters: &QueryFilters) -> bool` | Cek kecocokan `species_type`. |
| `filter_species` | `fn filter_species<'a>(species_list: &'a [Species], filters: &QueryFilters) -> Vec<&'a Species>` | Menggabungkan seluruh predicate di atas via `.iter().filter()`. |

**Person in Charge:** **(isi nama anggota)**

---

## 5. Tahap 3 — Relevance Scoring & Ranking

| Fungsi | Signature | Deskripsi |
|---|---|---|
| `score_text_match` | `fn score_text_match(species: &Species, filters: &QueryFilters) -> f64` | Skor kecocokan teks bebas terhadap nama/deskripsi. |
| `score_taxonomy_match` | `fn score_taxonomy_match(species: &Species, filters: &QueryFilters) -> f64` | Skor kecocokan taksonomi. |
| `score_conservation_match` | `fn score_conservation_match(species: &Species, filters: &QueryFilters) -> f64` | Skor kecocokan status konservasi. |
| `combine_scores` | `fn combine_scores(scores: &[f64]) -> f64` | Menggabungkan skor per atribut (mis. weighted sum). |
| `score_species` | `fn score_species(species: &Species, filters: &QueryFilters) -> f64` | Memanggil seluruh fungsi skor dan mengembalikan skor total. |
| `rank_species` | `fn rank_species<'a>(species_list: &'a [&'a Species], filters: &QueryFilters) -> Vec<(&'a Species, f64)>` | Memasangkan spesies dengan skornya lalu mengurutkan (descending) via `.map()` + `.sort_by()`. |

**Person in Charge:** **(isi nama anggota)**

---

## 6. Tahap 4 — Related-Query Recommendation

| Fungsi | Signature | Deskripsi |
|---|---|---|
| `extract_common_attributes` | `fn extract_common_attributes(results: &[&Species]) -> Vec<String>` | Mengambil pola atribut yang sering muncul (mis. `iucn`/`family` yang dominan) dari hasil saat ini. |
| `generate_candidate_queries` | `fn generate_candidate_queries(filters: &QueryFilters, attrs: &[String]) -> Vec<String>` | Membuat kandidat query lanjutan berdasarkan filter & atribut umum. |
| `score_query_candidate` | `fn score_query_candidate(query: &str, original_filters: &QueryFilters) -> f64` | Menilai relevansi kandidat query terhadap query asal. |
| `rank_related_queries` | `fn rank_related_queries(candidates: Vec<String>, filters: &QueryFilters) -> Vec<(String, f64)>` | Mengurutkan kandidat query berdasarkan skor. |

**Person in Charge:** **(isi nama anggota)**

---

## 7. Tahap 5 — Integrasi & Testing

| Fungsi / Komponen | Signature / Bentuk | Deskripsi |
|---|---|---|
| `fetch_species_data` | `async fn fetch_species_data(base_url: &str) -> Result<Vec<Species>, Error>` | Mengambil data dari `kalimantan-bio-demo-api` (`GET /api/v1/species`) dan mendeserialisasi ke `Vec<Species>` via `serde`. |
| `search` | `fn search(raw_query: &str, all_species: &[Species]) -> Vec<(&Species, f64)>` | Pipeline utama: parse → filter → score → rank. |
| Test fixtures | `Vec<Species>` dummy | Data uji coba independen dari koneksi API, untuk unit test yang tidak butuh server hidup. |

**Person in Charge:** **(isi nama anggota)**

> **Catatan:** Setiap PIC tetap bertanggung jawab terhadap pengujian fungsi yang mereka implementasikan. PIC tahap ini berfokus pada pengujian antar-komponen, koneksi ke API asli, dan pengujian end-to-end.

---

## 8. Komposisi Pipeline Utama

```rust
fn search(raw_query: &str, all_species: &[Species]) -> Vec<(&Species, f64)> {
    let normalized = normalize_text(raw_query);
    let tokens = tokenize(&normalized);
    let filters = extract_filters(&tokens);
    let filtered = filter_species(all_species, &filters);
    rank_species(&filtered, &filters)
}
```

Pipeline konseptual:

```text
Query Mentah (String)
  ↓
Normalisasi & Tokenisasi
  ↓
Ekstraksi Filter (QueryFilters)
  ↓
Filtering Spesies
  ↓
Scoring & Ranking
  ↓
Hasil Terurut (Vec<(&Species, f64)>)
```

Fungsi `search` adalah **entry point** modul dan menjadi contoh penerapan **function composition** — menggabungkan beberapa fungsi kecil bertanggung jawab spesifik menjadi satu proses besar.

---

## 9. Prinsip Functional Programming yang Perlu Dipegang Tim

### Pure Functions
Fungsi tidak mengubah state eksternal dan hanya bergantung pada input yang diberikan. Untuk input yang sama, fungsi menghasilkan output yang sama. `fetch_species_data` adalah pengecualian wajar karena melakukan I/O (network call) — sebaiknya dipisahkan tegas dari fungsi-fungsi murni lainnya di pipeline.

### Immutability
Gunakan `&T` ketika data hanya perlu dibaca; hasil transformasi dikembalikan sebagai data baru, bukan memodifikasi input.

### Higher-Order Functions
Manfaatkan `.map()`, `.filter()`, `.fold()`, `.sum()`, `.sort_by()` untuk transformasi, filtering, dan agregasi — hindari `for` loop manual di logika inti.

### Function Composition
Setiap fungsi punya satu tanggung jawab jelas dan dapat diuji independen: `parse → filter → score → rank`.

---

## 10. Pembagian Kerja — 5 Anggota

| # | Tahap | Fungsi / Tanggung Jawab Utama | PIC | Status |
|---|---|---|---|---|
| 1 | Parsing & Query Understanding | `normalize_text`, `tokenize`, `extract_filters` | | Belum dimulai |
| 2 | Multi-Attribute Filtering | `matches_free_text`, `matches_taxonomy`, `matches_conservation`, `matches_species_type`, `filter_species` | | Belum dimulai |
| 3 | Relevance Scoring & Ranking | `score_text_match`, `score_taxonomy_match`, `score_conservation_match`, `combine_scores`, `score_species`, `rank_species` | | Belum dimulai |
| 4 | Related-Query Recommendation | `extract_common_attributes`, `generate_candidate_queries`, `score_query_candidate`, `rank_related_queries` | | Belum dimulai |
| 5 | Integrasi & Testing | `fetch_species_data`, `search`, test fixtures, e2e test | | Belum dimulai |

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

* Struktur `Species`, `Taxonomy`, `QueryFilters` (termasuk apakah `p106`/`created_at`/`updated_at` dipakai atau tidak).
* Arti setiap field, terutama field yang berasal dari singkatan (`p106`, `cites`).
* Input dan output setiap fungsi.
* Function signature final.
* Format data antar-tahap (`Vec<&Species>` vs `Vec<Species>`, dsb).
* Aturan validasi data (mis. apa yang terjadi jika `image_url` null atau `latest_observation_year` kosong).
* Bobot/formula scoring pada `combine_scores`.
* Format output akhir pipeline `search()`.
* Strategi testing (unit test per fungsi vs integration test lewat API asli).

---

## 12. Independensi Modul

Modul ini dikembangkan sebagai komponen independen dalam KalimantanBio.

Prinsip yang digunakan:

* Modul dapat dikembangkan secara mandiri, terlepas dari status Modul 2–5.
* Modul dapat diuji secara mandiri menggunakan data dummy, tanpa harus selalu terkoneksi ke `kalimantan-bio-demo-api`.
* Modul tidak boleh bergantung pada implementasi internal modul lain.
* Integrasi dengan modul lain (mis. Modul 2 untuk relationship, Modul 3 untuk taxonomy tree) bersifat opsional dan melalui `Species`/`Taxonomy` sebagai shared data convention.

```text
              KalimantanBio
                   │
          ┌────────┼────────┐
          │        │        │
         M1       M2       M3
          │        │        │
         M4       M5
          │        │
          └────────┴────────┐
                            │
                    Optional Integration
```

---

## 13. Kriteria Selesai Modul

* [ ] Seluruh fungsi utama telah diimplementasikan.
* [ ] Setiap fungsi memiliki unit test yang relevan.
* [ ] Pipeline `search()` dapat berjalan end-to-end, baik dengan data dummy maupun data dari `kalimantan-bio-demo-api`.
* [ ] Input query bahasa natural dapat diproses sesuai spesifikasi.
* [ ] Output menghasilkan format yang telah disepakati (`Vec<(&Species, f64)>` terurut).
* [ ] Tidak ada dependency yang tidak diperlukan (mis. tidak bergantung langsung ke Modul 2–5).
* [ ] Tidak ada state global yang tidak diperlukan.
* [ ] Dokumentasi fungsi dan struktur data tersedia.
* [ ] Terdapat demonstrasi penggunaan modul (mis. contoh query & hasilnya).
* [ ] Modul dapat dijalankan secara independen, termasuk fallback jika API tidak tersedia (gunakan data dummy).

---

## 14. Contoh Skenario Pengujian

### Skenario 1 — Pencarian berbasis kata kunci umum

**Input:**
```text
"orangutan kalimantan"
```

**Expected Output:**
```text
[
  { scientific_name: "Pongo pygmaeus", common_name: "Orangutan Borneo", score: <tinggi> },
  ...
]
```

**Fungsi yang diuji:**
* `normalize_text`, `tokenize`, `extract_filters`
* `matches_free_text`, `score_text_match`

---

### Skenario 2 — Pencarian berbasis status konservasi

**Input:**
```text
"spesies berstatus kritis (CR)"
```

**Expected Output:**
```text
Daftar spesies dengan iucn == "CR", diurutkan berdasarkan skor relevansi.
```

**Fungsi yang diuji:**
* `extract_filters` (mendeteksi kata "kritis"/"CR" sebagai filter iucn)
* `matches_conservation`, `score_conservation_match`

---

### Edge Cases

| Case | Input | Expected Result |
|---|---|---|
| Empty input | `""` | Kembalikan list kosong atau seluruh spesies tanpa filter (disepakati tim). |
| Query tidak dikenali | `"asdkjaskjd"` | Tidak ada filter terbentuk, `free_text` diisi token tersebut, kemungkinan hasil kosong. |
| Tidak ada hasil cocok | `"spesies punah dari mars"` | List kosong, tidak error. |
| Banyak hasil cocok | `"mamalia"` | List terurut berdasarkan skor, tidak crash meski hasil banyak. |
| Field opsional null | `image_url: null`, `latest_observation_year: null` | Pipeline tetap berjalan tanpa panic (gunakan `Option<T>`). |

---

## 15. Langkah Selanjutnya

1. Konfirmasi ke project lead: apakah `kalimantan-bio-demo-api` adalah sumber data final Modul 1, dan apakah atribut habitat/morphology/uses akan ditambahkan kemudian atau memang di luar scope.
2. Finalisasi domain model (`Species`, `Taxonomy`, `QueryFilters`) berdasarkan struktur API asli di atas.
3. Sepakati input, output, dan signature setiap fungsi.
4. Konfirmasi pembagian fungsi berdasarkan PIC (isi tabel Bagian 10).
5. Setiap PIC membuat signature dan dokumentasi singkat fungsi masing-masing.
6. Review interface bersama sebelum implementasi logic dimulai.
7. Siapkan data dummy `Species` (independen dari API) untuk unit test.
8. Implementasikan fungsi secara paralel sesuai pembagian kerja.
9. Setiap PIC membuat unit test untuk fungsi masing-masing.
10. Gabungkan seluruh tahap ke dalam pipeline `search()`.
11. Lakukan pengujian end-to-end, termasuk koneksi nyata ke `kalimantan-bio-demo-api`.
12. Dokumentasikan hasil dan contoh penggunaan modul.
13. Review akhir sebelum modul dianggap selesai.
