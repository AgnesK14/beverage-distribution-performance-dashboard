# 📊 Beverage Distribution Performance Dashboard

> Analisis performa penjualan, pola distribusi channel, dan karakteristik supplier dari data transaksi distribusi minuman tahun 2020.

---

## 🗂️ Project Overview

| Item | Detail |
|------|--------|
| **Dataset** | Beverage Distribution Transactions 2020 |
| **Jumlah Data** | 30.000 rows, 9 columns |
| **Tools** | Microsoft Excel (Power Query), Looker Studio |
| **Tujuan** | Mengidentifikasi performa penjualan, efisiensi channel distribusi, dan top supplier/produk |

---

## 📁 Struktur File

```bash
📦 beverage-distribution-performance-dashboard
│
├── README.md
│
├── data/
│   ├── Retail and wherehouse Sale.csv
│   └── Cleaned Sales Data.xlsx
│
├── assets/
│   ├── dashboard-overview.png
│   └── channel-deep-dive.png

```

---

## 🔍 Dataset Info

```python
RangeIndex: 30.000 entries
Columns: 9

#   Column            Non-Null    Dtype
0   YEAR              30000       int64
1   MONTH             30000       int64
2   SUPPLIER          29967       object   ← ada missing value
3   ITEM CODE         30000       object
4   ITEM DESCRIPTION  30000       object
5   ITEM TYPE         30000       object
6   RETAIL SALES      29999       float64  ← ada missing value
7   RETAIL TRANSFERS  30000       float64
8   WAREHOUSE SALES   30000       float64
```

---

## 🛠️ Data Cleaning Process (Excel – Power Query)

### 1. Promoted Headers
Baris pertama dataset dijadikan nama kolom menggunakan fitur:

**Home → Use First Row as Headers**

---

### 2. Changed Data Type
Tipe data numerik disesuaikan secara manual menggunakan locale `en-US` agar format angka dan desimal konsisten.

Langkah:

**Right Click Column → Change Type → Using Locale**

Kolom yang disesuaikan:

- `RETAIL SALES`
- `RETAIL TRANSFERS`
- `WAREHOUSE SALES`

---

### 3. Filtered Blank Supplier
Record dengan nilai `SUPPLIER` kosong (~1% berdasarkan **Column Quality** Power Query) dihapus menggunakan:

**Dropdown Filter → Uncheck (blank)**

Tujuan:

Menghindari kategori supplier kosong saat proses grouping dan visualisasi.

---

### 4. Filtered Rows "DUNNAGE"
Kategori operasional non-revenue `DUNNAGE` dihapus menggunakan:

**Dropdown Filter pada ITEM TYPE → Uncheck DUNNAGE**

Tujuan:

Karena kategori ini merepresentasikan material operasional/logistik, bukan produk yang menghasilkan revenue.

---

### 5. Added MONTH NAME
Kolom baru ditambahkan untuk mengubah kode bulan numerik menjadi label bisnis yang lebih mudah dibaca.

Langkah:

**Add Column → Conditional Column**

Mapping:

```python
1 → Jan
3 → Mar
7 → Jul
9 → Sep
```

---

### 6. Trimmed Text – SUPPLIER
Spasi berlebih pada kolom `SUPPLIER` dihapus menggunakan:

**Transform → Format → Trim**

Tujuan:

Menghindari duplikasi kategori seperti:

```python
"Corona"
" Corona"
```

---

### 7. Filtered Negative Values – Retail Sales
Record dengan nilai:

```python
RETAIL SALES < 0
```

dihapus menggunakan:

**Number Filters → Greater Than or Equal To → 0**

Tujuan:

Nilai negatif merepresentasikan transaksi retur dan tidak menjadi fokus utama analisis penjualan.

---

### 8. Added WAREHOUSE RETURNS
Kolom baru dibuat untuk memisahkan nilai negatif pada `WAREHOUSE SALES`.

Langkah:

**Add Column → Conditional Column**

Logic:

```python
if WAREHOUSE SALES < 0
then WAREHOUSE SALES
else 0
```

Tujuan:

Agar transaksi retur/distribusi negatif tidak tercampur dengan perhitungan penjualan aktif.

---

### 9. Added WAREHOUSE SALES CLEAN
Kolom baru dibuat untuk membersihkan nilai negatif pada `WAREHOUSE SALES`.

Langkah:

**Add Column → Conditional Column**

Logic:

```python
if WAREHOUSE SALES >= 0
then WAREHOUSE SALES
else 0
```

Tujuan:

Memastikan hanya nilai distribusi positif yang dihitung sebagai revenue.

---

### 10. Added TOTAL SALES
Kolom baru dibuat untuk menghitung total penjualan dari seluruh channel distribusi.

Langkah:

**Add Column → Custom Column**

Formula:

```python
TOTAL SALES = RETAIL SALES 
            + RETAIL TRANSFERS 
            + WAREHOUSE SALES CLEAN
```

---

### 11. Removed Duplicates
Duplikasi transaksi dihapus berdasarkan kombinasi:

- `SUPPLIER`
- `ITEM CODE`
- `MONTH`

Langkah:

**Select Columns → Right Click → Remove Duplicates**

Tujuan:

Memastikan satu supplier tidak memiliki SKU identik pada bulan yang sama.

---

### 12. Added CHANNEL_COUNT
Kolom baru dibuat untuk menghitung jumlah channel aktif (0–3) pada setiap transaksi.

Langkah:

**Add Column → Custom Column**

Logic:

```python
(if RETAIL SALES > 0 then 1 else 0) +
(if RETAIL TRANSFERS > 0 then 1 else 0) +
(if WAREHOUSE SALES CLEAN > 0 then 1 else 0)
```

Interpretasi:

| Nilai | Arti |
|---|---|
| 0 | Tidak ada channel aktif |
| 1 | Single channel |
| 2 | Multi channel |
| 3 | Full channel |

---

### 13. Added CHANNEL_PROFILE
Kolom baru dibuat untuk mengkategorikan pola distribusi berdasarkan kombinasi channel aktif.

Langkah:

**Add Column → Conditional Column**

Kategori:

| Kategori | Kondisi |
|---|---|
| **Inactive** | Tidak ada channel aktif |
| **Full Channel** | Ketiga channel aktif |
| **Retail Only** | Hanya Retail Sales aktif |
| **Transfer Only** | Hanya Retail Transfers aktif |
| **Warehouse Only** | Hanya Warehouse Sales aktif |
| **Mixed Channel** | Kombinasi dua channel aktif |

---

## 📊 Dashboard Preview

### Page 1 – Business Performance Overview

Menampilkan:

- KPI utama penjualan
- Tren penjualan bulanan
- Distribusi penjualan per item type
- Top product by total sales
- Top supplier by total sales

---

### Page 2 – Channel & Distribution Deep Dive

Menampilkan:

- Distribusi channel profile
- Efisiensi channel distribusi
- Zero / inactive product analysis
- Supplier channel strategy
- Channel contribution by month

---

> 🔗 **https://datastudio.google.com/reporting/a228e131-6599-431b-ab03-ae5e3dcd29b7**  

---

## 💡 Key Insights

- **Total Penjualan:** $1.238.363 dari 29.824 transaksi
- **Average Sales:** $41 per transaksi
- **Beer & Liquor** menyumbang lebih dari 85% total revenue
- **Full Channel** menunjukkan performa tertinggi dengan rata-rata $137/transaksi
- **Retail Only** hanya menghasilkan rata-rata $1,18/transaksi
- Channel omnichannel terbukti **116x lebih efisien** dibanding single-channel retail
- **Produk terlaris:** Corona Extra Loose NR-12oz ($49.110)
- **Top supplier:** Crown Imports ($201.334)
- **127 SKU** teridentifikasi sebagai **Inactive Product**, membuka peluang rasionalisasi portofolio

---

## 🚀 How to Use

### 1. Download dataset
Download file:

```bash
beverage_dashboard_dataset.xlsx
```

---

### 2. Open in Excel
Gunakan:

**Microsoft Excel 2016+**

---

### 3. Refresh Power Query
Buka:

**Data → Refresh All**

Untuk memuat ulang transformasi data.

---

### 4. Explore Interactive Dashboard
Buka dashboard melalui link Looker Studio pada bagian **Dashboard Preview**.

---

## Author

**Agnes Kamelia Narjun**

📧 agneskameliaa@gmail.com  
🔗 https://www.linkedin.com/public-profile/settings?trk=d_flagship3_profile_self_view_public_profile
