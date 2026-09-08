# Planning Function — Modul [N]: [Nama Modul]

**Mata Kuliah:** Pemrograman Fungsional
**Bahasa:** Rust
**Cakupan:** [Tuliskan ruang lingkup utama modul]

**Repository:** `[nama-repository]`

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

## 2. Struktur Data (Domain Model)

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

> **Catatan:** Struktur data perlu disepakati oleh seluruh anggota kelompok sebelum implementasi karena struktur ini menjadi dasar bagi fungsi-fungsi pada tahap berikutnya.

### Data yang Digunakan

| Data      | Tipe     | Deskripsi   |
| --------- | -------- | ----------- |
| `[field]` | `[type]` | [Deskripsi] |
| `[field]` | `[type]` | [Deskripsi] |
| `[field]` | `[type]` | [Deskripsi] |

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
