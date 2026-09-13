# Planning Function — Modul 4: Comparative Species Explorer

**Mata Kuliah:** Pemrograman Fungsional
**Bahasa:** Rust
**Cakupan:** Pembangunan sistem perbandingan beberapa spesies di Kalimantan berdasarkan data taksonomi, atribut spesies, status konservasi, serta data observasi dan distribusi geografis untuk mengidentifikasi persamaan, perbedaan, karakteristik unik, dan tingkat kemiripan antarspesies.

**Repository:** `[nama-repository]`

---

## 1. Tujuan Modul

Modul ini bertujuan untuk membangun sistem perbandingan spesies yang memungkinkan pengguna memilih beberapa spesies di Kalimantan dan melihat persamaan, perbedaan, karakteristik unik, status koservasi, dan pola distribusi geografis berdasarkan data yang tersedia.

Modul ini bertanggung jawab untuk:

* Membandingkan beberapa spesies berdasarkan informasi taksomi dan atribut spesies yang tersedia.
* MMengidentifikasi atribut yang sama dan atribut yang berbeda atau unik antar spesies.
* Mengidentifikasi karakteristik yang paling membedakan satu spesies dengan spesies lainnya.
* Membandingkan karakteristik habitas atau informasi ekologis yang tersedia.
* Menghitung tingkat kemiripan antarspesies berdasarkan taxonomy, atribut spesies, distribusi, konservasi, dan atribut biodiversitas lain yang tersedia.
* Membandingkan status konservasi dan distribusi geografis berdasarkan data yang tersedia.
* Menemukan spesies lain yang memiliki tingkat kemiripan tinggi dengan spesies tertentu.
* Menghasilkan hasil perbandingan dalam bentuk data terstruktur yang dapat digunakan untuk visualisasi seperti tabel, matriks atribut, atau bentuk visualisasi lainnya.
* Menghasilkan ringkasan hasil perbandingan yang menjelaskan persamaan dan perbedaan utama antarspesies.

### Batasan Modul

Modul ini **tidak bertanggung jawab** terhadap:

* [Hal yang berada di luar scope]
* [Hal yang menjadi tanggung jawab modul lain]
* [Hal yang belum diperlukan untuk versi ini]

---

## 2. Struktur Data (Domain Model)

Struktur data utama yang digunakan oleh modul:

```rust
#[derive(Debug, Clone)]
struct Kingdom {
    id: u64,
    name: String,
    description: String,
}

#[derive(Debug, Clone)]
struct PhylumDivision {
    id: u64,
    name: String,
    rank_type: String,
    kingdom_id: u64,
}

#[derive(Debug, Clone)]
struct TaxonClass {
    id: u64,
    name: String,
    phylum_division_id: u64,
}

#[derive(Debug, Clone)]
struct TaxonOrder {
    id: u64,
    name: String,
    class_id: u64,
}

#[derive(Debug, Clone)]
struct Family {
    id: u64,
    name: String,
    order_id: u64,
}

#[derive(Debug, Clone)]
struct Genus {
    id: u64,
    name: String,
    family_id: u64,
}

#[derive(Debug, Clone)]
struct Species {
    id: u64,
    scientific_name: String,
    common_name: String,
    description: String,
    species_type: String,
    iucn: String,
    cites: String,
    p106: String,
    is_verified: bool,
    image_url: String,
    ai_key: i32,
    genus_id: u64,
}

#[derive(Debug, Clone)]
struct Province {
    id: u64,
    name: String,
}

#[derive(Debug, Clone)]
struct Kabupaten {
    id: u64,
    name: String,
    province_id: u64,
    forest_coverage: f64,
    centroid_latitude: f64,
    centroid_longitude: f64,
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
}

// Untuk input user memilih spesies
#[derive(Debug, Clone, Default)] 
struct ComparisonQuery {
    species_ids: Vec<u64>,
}

// Untuk menyimpan hasil scoring antarspesies.
#[derive(Debug, Clone)]
struct SimilarityScore {
    species_a: u64,
    species_b: u64,
    taxonomy_score: f64,
    attribute_score: f64,
    conservation_score: f64,
    distribution_score: f64,
    total_score: f64,
}

// Untuk menyimpan hasil akhir perbandingan
#[derive(Debug, Clone, Default)]
struct ComparisonResult {
    species: Vec<Species>,
    shared_attributes: Vec<String>,
    unique_attributes: Vec<String>,
    distinguishing_characteristics: Vec<String>,
    similarity_scores: Vec<SimilarityScore>,
}
```

> **Catatan:** Struktur data perlu disepakati oleh seluruh anggota kelompok sebelum implementasi karena struktur ini menjadi dasar bagi fungsi-fungsi pada tahap berikutnya.

### Data yang Digunakan
// Ngikut contoh modul 1, cuman ubah deskripsi aja
| Data      | Tipe     | Deskripsi   |
| --------- | -------- | ----------- |
| `scientific_name`, `common_name`, `description` | `String` | Informasi identitas dan deskripsi spesies yang digunakan sebagai atribut pembanding. |
| `species_type` | `String` | Menunjukkan kategori spesies, seperti Endemic, Native, atau Introduced. |
| `iucn`, `cites`, `p106` | `String` | Informasi status konservasi spesies berdasarkan status IUCN, CITES, dan perlindungan nasional. |
| `taxonomy` | `Taxonomy` | Informasi klasifikasi taksonomi spesies, meliputi kingdom, phylum/division, class, order, family, dan genus. |
| `is_verified` | `bool` | Menunjukkan apakah data spesies telah diverifikasi. |
| `observation_count` | `u32` | Jumlah observasi yang tercatat untuk suatu spesies. |
| `recorded_individuals_total` | `u32` | Total individu spesies yang tercatat dalam data observasi. |
| `latest_observation_year` | `Option<u32>` | Tahun observasi terbaru yang tersedia untuk suatu spesies. |


---

## 3. Tahap 1 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 4. Tahap 2 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |
| `[function_4]` | `fn [function_4](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 5. Tahap 3 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |
| `[function_4]` | `fn [function_4](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 6. Tahap 4 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |
| `[function_4]` | `fn [function_4](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 7. Tahap 5 — [Nama Tahap / Integrasi / Testing]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi / Komponen | Signature / Bentuk | Deskripsi   |
| ----------------- | ------------------ | ----------- |
| `[component_1]`   | `...`              | [Deskripsi] |
| `[component_2]`   | `...`              | [Deskripsi] |
| `[component_3]`   | `...`              | [Deskripsi] |

**Person in Charge:** **(isi nama anggota)**

> **Catatan:** Setiap PIC tetap bertanggung jawab terhadap pengujian fungsi yang mereka implementasikan. Jika tahap ini merupakan tahap integrasi, PIC berfokus pada pengujian antar-komponen dan pengujian end-to-end.

---

## 8. Komposisi Pipeline Utama

Setelah seluruh tahap tersedia, fungsi utama modul menggabungkan proses menjadi satu pipeline.

```rust
fn [main_function](input: &[Input]) -> Output {
    let step_1 = [function_1](input);
    let step_2 = [function_2](&step_1);
    let step_3 = [function_3](&step_2);

    [function_4](&step_3)
}
```

Pipeline konseptual:

```text
Input
  ↓
[Stage 1]
  ↓
[Stage 2]
  ↓
[Stage 3]
  ↓
[Stage 4]
  ↓
Output
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

## 10. Pembagian Kerja — [Jumlah Anggota] Anggota

| # | Tahap     | Fungsi / Tanggung Jawab Utama         | PIC | Status        |
| - | --------- | ------------------------------------- | --- | ------------- |
| 1 | [Tahap 1] | `[functions]`                         |     | Belum dimulai |
| 2 | [Tahap 2] | `[functions]`                         |     | Belum dimulai |
| 3 | [Tahap 3] | `[functions]`                         |     | Belum dimulai |
| 4 | [Tahap 4] | `[functions]`                         |     | Belum dimulai |
| 5 | [Tahap 5] | `[functions / integration / testing]` |     | Belum dimulai |

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
         M1       M2       M3
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

### Skenario 1 — [Nama Skenario]

**Input:**

```text
[contoh input]
```

**Expected Output:**

```text
[expected output]
```

**Fungsi yang diuji:**

* `[function]`
* `[function]`
* `[function]`

---

### Skenario 2 — [Nama Skenario]

**Input:**

```text
[contoh input]
```

**Expected Output:**

```text
[expected output]
```

**Fungsi yang diuji:**

* `[function]`
* `[function]`

---

### Edge Cases

| Case               | Input     | Expected Result |
| ------------------ | --------- | --------------- |
| Empty input        | `[input]` | `[result]`      |
| Invalid input      | `[input]` | `[result]`      |
| No matching result | `[input]` | `[result]`      |
| Multiple matches   | `[input]` | `[result]`      |

---

## 15. Langkah Selanjutnya

1. Finalisasi domain model bersama seluruh anggota.
2. Sepakati input, output, dan signature setiap fungsi.
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
