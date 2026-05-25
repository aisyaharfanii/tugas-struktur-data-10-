# Sorting & Binary Tree Algorithms

Implementasi Python untuk tugas **Analisis & Desain Algoritma** — Bab 12 (Sorting Lanjutan) dan Bab 13 (Binary Tree & Heap).

---

## Struktur Proyek

```
.
├── advanced_sorter.py    # Bab 12 — AdvancedSorter
├── expr_heap_sorter.py   # Bab 13 — ExprHeapSorter
├── test_all.py           # Test suite (41 test case)
└── README.md
```

---

## Modul 1 — `advanced_sorter.py` (Bab 12)

Kelas `AdvancedSorter` mengimplementasikan tiga algoritma sorting dengan batasan memori ketat.

### 1.1 Array Merge Sort

**Strategi:** Virtual sublists + single `tmpArray`.

- Hanya **satu** array sementara berukuran `n` yang dialokasikan di awal.
- Tidak ada sublist fisik di setiap level rekursi.
- **Stable sort** — elemen bernilai sama mempertahankan urutan relatif aslinya (`<=`).

```python
sorter = AdvancedSorter()
arr = [38, 27, 43, 3, 9, 82, 10]
print(sorter.sort_array(arr))
# → [3, 9, 10, 27, 38, 43, 82]
```

| Kompleksitas | Nilai |
|---|---|
| Waktu | O(n log n) |
| Ruang ekstra | O(n) — satu `tmpArray` |

### 1.2 Linked List Merge Sort

**Strategi:** Fast-slow pointer split + dummy node merge.

- `_split_linked_list()` menemukan titik tengah dalam **satu traversal** menggunakan dua pointer:
  - `midPoint` (slow): bergerak 1 langkah per iterasi.
  - `curNode` (fast): bergerak 2 langkah per iterasi.
  - Ketika `curNode` mencapai akhir, `midPoint` berada di tengah.
- `_merge_linked_lists()` menggunakan **dummy node + tail reference** — hanya memodifikasi pointer `.next`, tidak ada alokasi node baru.
- **Stable sort**.

```python
from advanced_sorter import list_to_linked, linked_to_list

head = list_to_linked([64, 34, 25, 12, 22, 11, 90])
sorted_head = sorter.sort_linked_list(head)
print(linked_to_list(sorted_head))
# → [11, 12, 22, 25, 34, 64, 90]
```

| Kompleksitas | Nilai |
|---|---|
| Waktu | O(n log n) |
| Ruang ekstra | O(log n) — hanya stack rekursi |

### 1.3 Quick Sort (Median-of-Three + Fallback)

**Strategi:** Pivot dipilih sebagai median dari `arr[first]`, `arr[mid]`, `arr[last]` untuk menghindari worst-case O(n²) pada data terurut.

**Fallback otomatis:** Jika kedalaman rekursi melebihi `2 * log₂(n)`, algoritma beralih ke Merge Sort secara otomatis.

```python
data = [10, 9, 8, 7, 6, 5, 4, 3, 2, 1]  # worst-case descending
print(sorter.sort_quick(data))
# → [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

| Kompleksitas | Nilai |
|---|---|
| Waktu (rata-rata) | O(n log n) |
| Waktu (terburuk, dengan fallback) | O(n log n) |
| Ruang ekstra | O(log n) — stack rekursi |

---

## Modul 2 — `expr_heap_sorter.py` (Bab 13)

Kelas `ExprHeapSorter` menggabungkan Expression Tree, In-Place Heapsort, dan Complete Tree Validator.

### 2.1 Expression Tree

Membangun pohon ekspresi dari string terparentheses penuh menggunakan **antrian token + rekursi** (sesuai pola Listing 13.9).

**Format input:** Ekspresi terparentheses penuh, seperti `((8*5)+(9/(7-4)))`.

**Evaluasi:** Traversal **postorder** — evaluasi subpohon kiri → kanan → terapkan operator.

```python
sorter = ExprHeapSorter("((8*5)+(9/(7-4)))")
result = sorter.parse_and_evaluate()
# (8*5)=40, (7-4)=3, (9/3)=3, 40+3=43
print(result[0])   # → 43

# Pembagian nol ditangkap
sorter2 = ExprHeapSorter("(5/(3-3))")
try:
    sorter2.parse_and_evaluate()
except ValueError as e:
    print(e)   # → Pembagian dengan nol terdeteksi!
```

**Pola rekursi `_build_tree()`:**
```
'('  → rekursi kiri → baca operator → rekursi kanan → konsumsi ')'
digit → node daun (operand)
```

### 2.2 Heapsort In-Place

Dua fase, benar-benar in-place (hanya variabel indeks):

1. **Build max-heap** dari daun ke akar: `range(n//2 - 1, -1, -1)` → `_sift_down()`.
2. **Ekstraksi berulang**: tukar `arr[0]` (maks) ke `arr[end]`, kurangi `heap_size`, sift-down ulang.

```python
sorter = ExprHeapSorter()
data = [43, 12, 7, 19, 3, 55, 1, 30]
print(sorter.heapsort_inplace(data))
# → [1, 3, 7, 12, 19, 30, 43, 55]
```

**Rumus indeks (0-based):**
```
left  = 2 * i + 1
right = 2 * i + 2
parent = (i - 1) // 2
```

| Kompleksitas | Nilai |
|---|---|
| Waktu | O(n log n) |
| Ruang ekstra | O(1) — benar-benar in-place |
| Stabilitas | Tidak stabil (inherent) |

### 2.3 Complete Tree Validator

Memvalidasi apakah array memenuhi properti **complete binary tree** menggunakan pemetaan indeks array.

**Algoritma:** Setelah menemukan node pertama yang tidak memiliki anak kiri, semua node berikutnya harus berupa daun. Jika ada anak setelah "lubang" pertama → bukan complete tree.

```python
sorter = ExprHeapSorter()
print(sorter.is_complete_tree([1, 2, 3, 4, 5, 6, 7]))  # → True  (perfect tree)
print(sorter.is_complete_tree([1, 2, 3, 4, 5]))         # → True  (5 node, ok)
print(sorter.is_complete_tree([]))                       # → True  (kosong)
```

---

## Batasan Teknis yang Dipenuhi

| Batasan | Status |
|---|---|
| Dilarang `list.sort()`, `sorted()`, `heapq` | ✅ |
| Dilarang slice `[:]` untuk pemisahan | ✅ |
| Array sort: hanya satu `tmpArray` berukuran n | ✅ |
| Linked list sort: hanya modifikasi pointer `.next` | ✅ |
| Tidak ada alokasi node baru saat sorting | ✅ |
| Stabilitas dijaga (merge sort) | ✅ |
| Quick sort fallback jika depth > 2·log₂(n) | ✅ |
| Tangani pembagian nol & token tidak valid | ✅ |

---

## Cara Menjalankan

**Requirement:** Python 3.7+ (tidak ada library eksternal)

```bash
# Jalankan semua test
python test_all.py

# Jalankan demo modul Bab 12
python advanced_sorter.py

# Jalankan demo modul Bab 13
python expr_heap_sorter.py
```

**Output test:**
```
HASIL: 41/41 test passed  🎉 Semua test lulus!
```

---

## Referensi Teori

### Mengapa Merge Sort unggul untuk Linked List?
Array-based merge sort memerlukan O(n) ruang ekstra untuk menyalin elemen. Pada linked list, kita cukup memodifikasi pointer `.next` — tidak perlu menyalin data, sehingga ruang ekstra turun ke O(log n) (hanya stack rekursi).

### Mengapa Radix Sort tidak melanggar Ω(n log n)?
Batas bawah Ω(n log n) berlaku untuk **comparison sort** — algoritma yang hanya mengandalkan perbandingan dua elemen. Radix Sort tidak melakukan perbandingan; ia mengeksploitasi struktur digit dari kunci. Ini bukan kontradiksi: kedua batas berlaku pada model komputasi yang berbeda.

### Mengapa In-Place Heapsort tetap O(n log n)?
- **Fase build-heap:** O(n) — jumlah total operasi sift-down dari semua node adalah O(n).
- **Fase ekstraksi:** n-1 ekstraksi, masing-masing sift-down O(log n) → total O(n log n).
- Swap berulang tidak mengubah asimptotik karena setiap swap membutuhkan waktu O(1).

---

## Lisensi

Kode ini dibuat untuk keperluan tugas akademik.
