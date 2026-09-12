# Planning Function — Modul 3: Taxonomy & Classification Explorer

**Mata Kuliah:** Pemrograman Fungsional  
**Bahasa:** Rust  
**Cakupan:** Pembangunan pohon taksonomi hirarkis, eksplorasi berbasis takson, analisis keanekaragaman (diversity), analisis coverage/gap, dan eksplorasi taksonomi endemik Kalimantan.  

**Repository:** `kalimantanbio-modul3-taxonomy`

---

## 1. Tujuan Modul

Modul ini bertujuan untuk memproses data mentah klasifikasi biologis dan mengubahnya menjadi struktur hirarkis (pohon taksonomi) yang dapat dieksplorasi, dianalisis statistiknya, dan dibandingkan dengan data referensi global untuk menemukan celah data (*taxonomic gap*).

Modul ini bertanggung jawab untuk:

* Membangun *Taxonomic Tree* dari data spesies dan takson.
* Menghitung statistik keanekaragaman (jumlah family, genus, spesies) untuk kelompok taksonomi tertentu.
* Menganalisis *Taxonomic Gap* (data yang hilang) dengan membandingkannya terhadap *checklist* referensi.
* Mengidentifikasi dan memfilter taksonomi yang berstatus endemik Kalimantan/Borneo.

### Batasan Modul

Modul ini **tidak bertanggung jawab** terhadap:

* Pencarian berbasis *natural language* atau *fuzzy search* (Tanggung jawab Modul 1).
* Perbandingan visual morfologi antar spesies secara *side-by-side* (Tanggung jawab Modul 4).
* Penyimpanan data sitasi, jurnal ilmiah, dan publikasi (Tanggung jawab Modul 5).

---

## 2. Struktur Data (Domain Model)

Struktur data utama yang digunakan oleh modul:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
struct TaxonId(pub u64);

#[derive(Debug, Clone, Copy, PartialEq, Eq)]
enum TaxonomicRank {
    Kingdom,
    Phylum,
    Class,
    Order,
    Family,
    Genus,
    Species,
}

#[derive(Debug, Clone)]
struct Taxon {
    id: TaxonId,
    name: String,
    rank: TaxonomicRank,
    parent_id: Option<TaxonId>,
}

#[derive(Debug, Clone)]
struct Species {
    id: u64,
    scientific_name: String,
    genus_id: TaxonId,
    is_endemic_to_borneo: bool,
    conservation_status: String,
}

#[derive(Debug, Clone, Default)]
struct DiversityStats {
    total_families: usize,
    total_genera: usize,
    total_species: usize,
    endemic_species_count: usize,
}

#[derive(Debug, Clone, Default)]
struct BioDataInput {
    raw_taxons: Vec<RawTaxonRecord>,
    raw_species: Vec<RawSpeciesRecord>,
    reference_genera: std::collections::HashSet<String>,
}
```

> **Catatan:** Struktur data perlu disepakati oleh seluruh anggota kelompok sebelum implementasi karena struktur ini menjadi dasar bagi fungsi-fungsi pada tahap berikutnya.

### Data yang Digunakan

| Data | Tipe | Deskripsi |
| --- | --- | --- |
| `raw_taxons` | `Vec<RawTaxonRecord>` | Data mentah node taksonomi (Ordo, Famili, Genus, dll). |
| `raw_species` | `Vec<RawSpeciesRecord>` | Data mentah daftar spesies yang terdaftar di KalimantanBio. |
| `reference_genera` | `HashSet<String>` | Daftar nama genus dari literatur global untuk Gap Analysis. |

---

## 3. Tahap 1 — Data Ingestion & Parsing

Membaca data mentah, memvalidasi, dan memetakan ke dalam Domain Model tanpa mutasi state.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `parse_taxons` | `fn parse_taxons(raw: &[RawTaxonRecord]) -> Result<Vec<Taxon>, ParseError>` | Memvalidasi dan memetakan data mentah takson menjadi `Vec<Taxon>`. |
| `parse_species` | `fn parse_species(raw: &[RawSpeciesRecord]) -> Result<Vec<Species>, ParseError>` | Memvalidasi data spesies dan memastikan `genus_id` valid. |
| `validate_hierarchy` | `fn validate_hierarchy(taxons: &[Taxon]) -> bool` | Pure function untuk mengecek tidak adanya cyclic reference atau orphan node. |

**Person in Charge:** **[Nama Kamu]**

---

## 4. Tahap 2 — Taxonomic Tree Construction

Membangun struktur pohon (hirarki) menggunakan pendekatan FP (rekursi, folding, dan pengelompokan data).

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `build_taxonomy_map` | `fn build_taxonomy_map(taxons: &[Taxon]) -> HashMap<TaxonId, Vec<TaxonId>>` | Mengelompokkan `parent_id` ke `children_ids` menggunakan `.fold()` atau `.group_by()`. |
| `get_lineage` | `fn get_lineage(taxon_id: &TaxonId, map: &HashMap<TaxonId, Taxon>) -> Vec<Taxon>` | Menelusuri ke atas (iteratif) untuk mendapatkan Kingdom -> ... -> Genus dari satu node. |
| `get_subtree_species` | `fn get_subtree_species(target: &TaxonId, map: &HashMap<TaxonId, Vec<TaxonId>>, species: &[Species]) -> Vec<Species>` | Mengumpulkan semua spesies yang berada di bawah target takson. |
| `extract_our_genera` | `fn extract_our_genera(taxons: &[Taxon]) -> HashSet<String>` | Mengekstrak semua nama Genus yang ada di database lokal untuk keperluan analisis gap. |

**Person in Charge:** **[Nama Kamu]**

---

## 5. Tahap 3 — Diversity & Endemic Analytics

Melakukan agregasi data untuk menghitung statistik dan memfilter spesies endemik menggunakan Higher-Order Functions.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `calculate_diversity` | `fn calculate_diversity(species_subset: &[Species]) -> DiversityStats` | Menggunakan `.iter().fold()` untuk menghitung total genus, famili, dan status endemik. |
| `filter_endemic_taxa` | `fn filter_endemic_taxa(taxons: &[Taxon], species: &[Species]) -> Vec<Taxon>` | Memfilter hanya Genus/Family yang memiliki spesies endemik Kalimantan menggunakan `.filter()`. |
| `rank_taxon_richness` | `fn rank_taxon_richness(map: &HashMap<TaxonId, Vec<TaxonId>>, species: &[Species]) -> Vec<(Taxon, usize)>` | Mengurutkan Famili berdasarkan jumlah spesies terbanyak (Species Richness). |
| `get_subtree_species` | `fn get_subtree_species(target: &TaxonId, map: &HashMap<TaxonId, Vec<TaxonId>>, species: &[Species]) -> Vec<Species>` | Mengumpulkan semua spesies yang berada di bawah target takson. |

**Person in Charge:** **[Nama Kamu]**

---

## 6. Tahap 4 — Coverage & Gap Analysis

Membandingkan data internal dengan referensi eksternal untuk menemukan Taxonomic Gap menggunakan Set Operations.

| Fungsi | Signature | Deskripsi |
| --- | --- | --- |
| `find_missing_taxa` | `fn find_missing_taxa(our_genera: &HashSet<String>, ref_genera: &HashSet<String>) -> Vec<String>` | Menggunakan Set Difference untuk mencari Genus yang belum ada di database. |
| `calculate_coverage_score` | `fn calculate_coverage_score(our_data: &HashSet<String>, ref_data: &HashSet<String>) -> f64` | Menghitung persentase cakupan data (0.0 hingga 1.0). |
| `extract_our_genera` | `fn extract_our_genera(taxons: &[Taxon]) -> HashSet<String>` | Mengekstrak semua nama Genus yang ada di database lokal. |
| `generate_gap_report` | `fn generate_gap_report(missing: &[String], score: f64) -> GapReport` | Menyusun hasil analisis gap menjadi struktur laporan. |

**Person in Charge:** **[Nama Kamu]**

---

## 7. Tahap 5 — Integrasi Pipeline & Testing

Menggabungkan fungsi-fungsi di atas menjadi entry point dan melakukan End-to-End testing.

| Fungsi / Komponen | Signature / Bentuk | Deskripsi |
| --- | --- | --- |
| `generate_taxonomy_report` | `fn generate_taxonomy_report(input: &BioDataInput) -> Result<ExplorationReport, PipelineError>` | Entry point yang merangkai Tahap 1 hingga Tahap 4 menjadi satu laporan utuh. |
| Unit Tests | `mod tests { ... }` | Menguji `get_lineage`, `find_missing_taxa`, dan `calculate_diversity` dengan dummy data. |
| Pipeline Validation | `cargo test` | Menjalankan seluruh skenario pengujian secara otomatis. |

**Person in Charge:** **[Nama Kamu]**

> **Catatan:** Setiap PIC tetap bertanggung jawab terhadap pengujian fungsi yang mereka implementasikan. Jika tahap ini merupakan tahap integrasi, PIC berfokus pada pengujian antar-komponen dan pengujian end-to-end.

---

## 8. Komposisi Pipeline Utama

Setelah seluruh tahap tersedia, fungsi utama modul menggabungkan proses menjadi satu pipeline.

```rust
fn generate_taxonomy_report(input: &BioDataInput) -> Result<ExplorationReport, PipelineError> {
    let step_1_taxons = parse_taxons(&input.raw_taxons)?;
    let step_1_species = parse_species(&input.raw_species)?;
    
    let step_2_map = build_taxonomy_map(&step_1_taxons);
    let our_genera = extract_our_genera(&step_1_taxons);
    
    let step_3_endemic = filter_endemic_taxa(&step_1_taxons, &step_1_species);
    let step_3_richness = rank_taxon_richness(&step_2_map, &step_1_species);
    
    let step_4_missing = find_missing_taxa(&our_genera, &input.reference_genera);
    let step_4_score = calculate_coverage_score(&our_genera, &input.reference_genera);
    
    Ok(ExplorationReport {
        endemic_genera: step_3_endemic,
        richness_rank: step_3_richness,
        missing_taxa: step_4_missing,
        coverage_score: step_4_score,
    })
}
```

Pipeline konseptual:

```text
Input
  ↓
[Stage 1: Parsing & Validation]
  ↓
[Stage 2: Tree Construction]
  ↓
[Stage 3: Diversity & Endemic Filtering]
  ↓
[Stage 4: Set Operations for Gap Analysis]
  ↓
Output (ExplorationReport)
```

Fungsi utama merupakan **entry point** modul dan menjadi contoh penerapan **function composition**, yaitu menggabungkan beberapa fungsi dengan tanggung jawab spesifik menjadi satu proses yang lebih besar.

---

## 9. Prinsip Functional Programming yang Perlu Dipegang Tim

### Pure Functions

Fungsi sebaiknya tidak mengubah state eksternal dan hanya bergantung pada input yang diberikan.

```text
Input → Function → Output
```

Untuk input yang sama, fungsi idealnya menghasilkan output yang sama.

### Immutability

Hindari memodifikasi data input secara langsung.

Gunakan reference/borrow seperti `&T` ketika data hanya perlu dibaca dan hasil transformasi dikembalikan sebagai data baru apabila diperlukan.

### Higher-Order Functions

Manfaatkan iterator dan fungsi seperti:

```rust
.map()
.filter()
.fold()
.sum()
```

untuk melakukan transformasi, filtering, dan agregasi data.

Gunakan pendekatan iterator ketika lebih sesuai dengan karakteristik operasi yang dilakukan.

### Function Composition

Pecah proses utama menjadi fungsi-fungsi kecil yang dapat dikombinasikan.

```text
Function A
    ↓
Function B
    ↓
Function C
    ↓
Function D
```

Setiap fungsi sebaiknya memiliki satu tanggung jawab yang jelas dan dapat diuji secara independen.

---

## 10. Pembagian Kerja — 1 Anggota

| # | Tahap | Fungsi / Tanggung Jawab Utama | PIC | Status |
| --- | --- | --- | --- | --- |
| 1 | Tahap 1 | `parse_taxons`, `parse_species`, `validate_hierarchy` | [Nama Kamu] | Belum dimulai |
| 2 | Tahap 2 | `build_taxonomy_map`, `get_lineage`, `get_subtree_species` | [Nama Kamu] | Belum dimulai |
| 3 | Tahap 3 | `calculate_diversity`, `filter_endemic_taxa`, `rank_taxon_richness` | [Nama Kamu] | Belum dimulai |
| 4 | Tahap 4 | `find_missing_taxa`, `calculate_coverage_score`, `extract_our_genera` | [Nama Kamu] | Belum dimulai |
| 5 | Tahap 5 | `generate_taxonomy_report` & Unit Testing | [Nama Kamu] | Belum dimulai |

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

* Struktur domain model.
* Arti setiap field.
* Input dan output setiap fungsi.
* Function signature.
* Format data yang digunakan antar-tahap.
* Aturan validasi data.
* Aturan transformasi data.
* Metode perhitungan atau scoring jika digunakan.
* Format output akhir.
* Strategi testing.

Tujuannya adalah memastikan fungsi yang dikembangkan oleh anggota berbeda tetap dapat dikombinasikan tanpa perubahan besar pada interface masing-masing.

---

## 12. Independensi Modul

Modul ini dikembangkan sebagai komponen independen dalam KalimantanBio.

Prinsip yang digunakan:

* Modul dapat dikembangkan secara mandiri.
* Modul dapat diuji secara mandiri.
* Modul tidak boleh bergantung pada implementasi internal modul lain.
* Gunakan shared concepts atau data conventions hanya ketika diperlukan.
* Integrasi dengan modul lain bersifat opsional dan dilakukan melalui interface yang telah disepakati.
* Jangan mengasumsikan dependency terhadap modul lain tanpa kebutuhan teknis yang jelas.

```text
              KalimantanBio
                   │
          ┌────────┼────────┐
          │        │        │
         M1       M2       M3 (Modul Kamu - Foundation)
          │        │        │
         M4       M5
          │        │
          └────────┴────────┐
                            │
                    Optional Integration
```

Diagram di atas menggambarkan hubungan **konseptual**, bukan dependency teknis.

---

## 13. Kriteria Selesai Modul

Modul dianggap siap untuk tahap akhir apabila:

* [ ] Seluruh fungsi utama telah diimplementasikan.
* [ ] Setiap fungsi memiliki unit test yang relevan.
* [ ] Pipeline utama dapat berjalan end-to-end.
* [ ] Input dapat diproses sesuai spesifikasi.
* [ ] Output menghasilkan format yang telah disepakati.
* [ ] Tidak terdapat dependency yang tidak diperlukan.
* [ ] Tidak terdapat state global yang tidak diperlukan.
* [ ] Dokumentasi fungsi dan struktur data tersedia.
* [ ] Terdapat demonstrasi penggunaan modul.
* [ ] Modul dapat dijalankan secara independen.

---

## 14. Contoh Skenario Pengujian

### Skenario 1 — Menghitung Keanekaragaman Familia Tertentu

**Input:**

```text
target = "Dipterocarpaceae", species_list = [meranti, kapur, ...]
```

**Expected Output:**

```text
DiversityStats { total_families: 1, total_genera: 15, total_species: 120, endemic_species_count: 45 }
```

**Fungsi yang diuji:**

* `get_subtree_species`
* `calculate_diversity`
* `rank_taxon_richness`

---

### Skenario 2 — Mencari Taxonomic Gap

**Input:**

```text
our_db = ["Genus A", "Genus B"], reference = ["Genus A", "Genus B", "Genus C"]
```

**Expected Output:**

```text
vec!["Genus C"] (Missing Taxa)
```

**Fungsi yang diuji:**

* `find_missing_taxa`
* `calculate_coverage_score`

---

### Edge Cases

| Case | Input | Expected Result |
| --- | --- | --- |
| Empty input | `species_list = []` | Mengembalikan `DiversityStats` dengan semua nilai 0. |
| Invalid input | `genus_id` merujuk pada ID yang tidak ada | `parse_species` mengembalikan `ParseError`. |
| No matching result | Tidak ada spesies endemik di subtree | `filter_endemic_taxa` mengembalikan `Vec` kosong `[]`. |
| Multiple matches | Beberapa Genus memiliki nama yang sama di referensi | Semua Genus yang hilang tetap terdeteksi dan dihitung skor cakupan dengan benar. |

---

## 15. Langkah Selanjutnya

1. Finalisasi domain model bersama seluruh anggota.
2. Sepakati input, output, dan signature me-masing fungsi.
3. Tentukan pembagian fungsi berdasarkan PIC.
4. Setiap PIC membuat signature dan dokumentasi singkat fungsi masing-masing.
5. Review interface sebelum implementasi logic dimulai.
6. Siapkan data dummy atau test fixtures.
7. Implementasikan fungsi secara paralel sesuai pembagian kerja.
8. Setiap PIC membuat unit test untuk fungsi masing-masing.
9. Gabungkan seluruh tahap ke dalam pipeline utama.
10. Lakukan pengujian end-to-end.
11. Dokumentasikan hasil dan contoh penggunaan modul.
12. Review akhir sebelum modul dianggap selesai.
