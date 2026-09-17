# Planning Function — Modul [N]: [Nama Modul]

**Mata Kuliah:** Pemrograman Fungsional
**Bahasa:** Rust
**Cakupan:** [Tuliskan ruang lingkup utama modul]

**Repository:** Part of KalimantanBio monorepo workspace (`crates/[nama-crate]`)
**API Integration:** Production KalimantanBio API (PostgreSQL via shared library)

> **PENTING**: Modul ini menggunakan **Shared Library** (`kalimantanbio-shared`) untuk tipe data dan fungsi umum. Lihat [MASTERPLAN.md](../MASTERPLAN.md) untuk arsitektur lengkap.

Daftar crate yang valid (lihat [MASTERPLAN.md](../MASTERPLAN.md) Section 4):

| Modul | Crate |
| --- | --- |
| 1. Intelligent Species Search | `crates/species-search` |
| 2. Species Relationship Explorer | `crates/species-relationships` |
| 3. Taxonomy & Classification Explorer | `crates/taxonomy` |
| 4. Comparative Species Explorer | `crates/species-comparison` |
| 5. Biodiversity Knowledge & Citation Explorer | `crates/knowledge-citations` |
| Shared Library | `crates/shared` |
| API Server | `crates/api-server` |

---

## 1. Tujuan Modul

[Deskripsikan secara singkat tujuan utama modul dan masalah yang ingin diselesaikan.]

Modul ini bertanggung jawab untuk:

* [Tanggung jawab 1]
* [Tanggung jawab 2]
* [Tanggung jawab 3]
* [Tanggung jawab 4]

### Batasan Modul

Modul ini **tidak bertanggung jawab** terhadap:

* [Hal yang berada di luar scope]
* [Hal yang menjadi tanggung jawab modul lain]
* [Hal yang belum diperlukan untuk versi ini]

---

## 2. Shared Library Integration

Modul ini menggunakan **shared library** (`kalimantanbio-shared`) untuk tipe data dan fungsi umum yang digunakan bersama dengan modul lain.

### 2.1 Shared Types Used

```rust
use kalimantanbio_shared::core::[TypeName];
```

[Deskripsikan tipe-tipe shared yang digunakan dan perannya di modul ini.]

### 2.2 Shared Functions Used

#### From `[library-module]` module:
```rust
use kalimantanbio_shared::[module]::[function_name];
```

- `[function_name]([signature])` - [Deskripsi singkat]

**[Digunakan di]**: [Tahap X]

*(Ulangi blok di atas untuk setiap fungsi/modul shared yang digunakan.)*

### 2.3 Module-Specific Types

Modul ini mendefinisikan tipe tambahan yang spesifik:

```rust
#[derive(Debug, Clone)]
struct [ModuleSpecificType] {
    // field
}
```

### 2.4 Cargo.toml Dependency

```toml
[dependencies]
kalimantanbio-shared = { path = "../shared" }
[library-lain] = { workspace = true }
```

### 2.5 Integration Example

```rust
use kalimantanbio_shared::[module]::[function];

pub fn [example]() {
    // contoh penggunaan shared library
}
```

---

## 3. Struktur Data (Domain Model)

Struktur data utama yang digunakan oleh modul:

```rust
#[derive(Debug, Clone)]
struct [MainEntity] {
    // field
}

#[derive(Debug, Clone)]
struct [SupportingEntity] {
    // field
}

#[derive(Debug, Clone, Default)]
struct [Input/Query/Filter] {
    // field
}
```

> **Catatan:** Tipe data standar (`Species`, `Taxonomy`, `Observation`) sudah distandardisasi di shared library. Hanya definisikan tipe yang spesifik untuk modul ini. Struktur data perlu disepakati oleh seluruh anggota kelompok sebelum implementasi.

### Data yang Digunakan

| Data      | Tipe     | Deskripsi   |
| --------- | -------- | ----------- |
| `[field]` | `[type]` | [Deskripsi] |
| `[field]` | `[type]` | [Deskripsi] |
| `[field]` | `[type]` | [Deskripsi] |

---

## 4. Tahap 1 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |

*(Tandai fungsi yang berasal dari shared library dengan "**FROM SHARED LIBRARY**".)*

**Person in Charge:** **(isi nama anggota)**

---

## 5. Tahap 2 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |
| `[function_4]` | `fn [function_4](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 6. Tahap 3 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |
| `[function_4]` | `fn [function_4](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 7. Tahap 4 — [Nama Tahap]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi         | Signature                     | Deskripsi          |
| -------------- | ----------------------------- | ------------------ |
| `[function_1]` | `fn [function_1](...) -> ...` | [Deskripsi fungsi] |
| `[function_2]` | `fn [function_2](...) -> ...` | [Deskripsi fungsi] |
| `[function_3]` | `fn [function_3](...) -> ...` | [Deskripsi fungsi] |
| `[function_4]` | `fn [function_4](...) -> ...` | [Deskripsi fungsi] |

**Person in Charge:** **(isi nama anggota)**

---

## 8. Tahap 5 — [Nama Tahap / Integrasi / Testing]

[Deskripsi singkat mengenai tanggung jawab tahap ini.]

| Fungsi / Komponen | Signature / Bentuk | Deskripsi   |
| ----------------- | ------------------ | ----------- |
| `[component_1]`   | `...`              | [Deskripsi] |
| `[component_2]`   | `...`              | [Deskripsi] |
| `[component_3]`   | `...`              | [Deskripsi] |

**Person in Charge:** **(isi nama anggota)**

> **Catatan:** Setiap PIC tetap bertanggung jawab terhadap pengujian fungsi yang mereka implementasikan. Jika tahap ini merupakan tahap integrasi, PIC berfokus pada pengujian antar-komponen dan pengujian end-to-end.

---

## 9. Komposisi Pipeline Utama

Setelah seluruh tahap tersedia, fungsi utama modul menggabungkan proses menjadi satu pipeline.

```rust
use kalimantanbio_shared::[module]::[shared_function];

fn [main_function](input: &[Input]) -> Output {
    let step_1 = [function_1](input);
    let step_2 = [shared_function](&step_1);   // shared library
    let step_3 = [function_3](&step_2);        // module-specific

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

## 10. Prinsip Functional Programming yang Perlu Dipegang Tim

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

## 11. Pembagian Kerja — [Jumlah Anggota] Anggota

| # | Tahap     | Fungsi / Tanggung Jawab Utama         | PIC | Status        |
| - | --------- | ------------------------------------- | --- | ------------- |
| 1 | [Tahap 1] | `[functions]`                         |     | Belum dimulai |
| 2 | [Tahap 2] | `[functions]`                         |     | Belum dimulai |
| 3 | [Tahap 3] | `[functions]`                         |     | Belum dimulai |
| 4 | [Tahap 4] | `[functions]`                         |     | Belum dimulai |
| 5 | [Tahap 5] | `[functions / integration / testing]` |     | Belum dimulai |

> **Catatan:** Fungsi yang berasal dari shared library (`kalimantanbio-shared`) **tidak** perlu diimplementasikan oleh modul — cukup gunakan. Jangan duplikasi fungsi yang sudah ada di shared library.

### Pembagian Tanggung Jawab

Setiap PIC bertanggung jawab terhadap:

* Implementasi fungsi yang ditugaskan.
* Unit testing fungsi tersebut.
* Dokumentasi fungsi.
* Menjaga signature/interface yang telah disepakati.
* Melaporkan perubahan yang dapat memengaruhi komponen lain.

---

## 12. Kesepakatan Antaranggota

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

## 13. Independensi Modul

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
               ├── kalimantanbio-shared  (dipakai bersama)
               │
          ┌────┴────┐
          │         │
         M1 ... M5
          │         │
     Independent    Independent
```

Modul hanya bergantung pada **shared library** (`kalimantanbio-shared`), bukan pada implementasi modul lain.

Diagram di atas menggambarkan hubungan **konseptual**, bukan dependency teknis.

---

## 14. Kriteria Selesai Modul

Modul dianggap siap untuk tahap akhir apabila:

* [ ] Seluruh fungsi utama telah diimplementasikan.
* [ ] Fungsi-fungsi **shared library** digunakan, bukan diduplikasi.
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

## 15. Contoh Skenario Pengujian

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

## 16. Langkah Selanjutnya

1. Verifikasi bahwa **shared library** (`kalimantanbio-shared`) sudah tersedia; jika belum, blokir pengerjaan sampai siap.
2. Finalisasi domain model bersama seluruh anggota.
3. Sepakati input, output, dan signature setiap fungsi.
4. Tentukan pembagian fungsi berdasarkan PIC.
5. Setiap PIC membuat signature dan dokumentasi singkat fungsi masing-masing.
6. Review interface sebelum implementasi logic dimulai.
7. Siapkan data dummy atau test fixtures (gunakan fixtures dari shared library bila tersedia).
8. Implementasikan fungsi secara paralel sesuai pembagian kerja.
9. Setiap PIC membuat unit test untuk fungsi masing-masing.
10. Gabungkan seluruh tahap ke dalam pipeline utama.
11. Lakukan pengujian end-to-end.
12. Dokumentasikan hasil dan contoh penggunaan modul.
13. Review akhir sebelum modul dianggap selesai.
