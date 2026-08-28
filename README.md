# Fashion E-Commerce — Analisis Penjualan H1 2026

Analisis penjualan enam bulan untuk perusahaan fashion e-commerce, mengubah data transaksi
mentah menjadi empat rekomendasi bisnis yang bisa ditindaklanjuti.

**Temuan utama:** revenue terlihat stabil di angka +1,0%, tetapi kestabilan itu semu.
Akuisisi customer baru sedang menutupi erosi customer lama, dan yang bocor bukan daya beli
melainkan kehadiran.

<p>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11-333?logo=python&logoColor=white">
<img alt="DuckDB" src="https://img.shields.io/badge/DuckDB-SQL-A86A45">
<img alt="pandas" src="https://img.shields.io/badge/pandas-data-333">
<img alt="matplotlib" src="https://img.shields.io/badge/matplotlib-viz-A86A45">
<img alt="Tableau" src="https://img.shields.io/badge/Tableau-dashboard-333">
</p>

[Penjelasan lengkap project ini terdapat juga di medium](https://medium.com/@aoramaaulia/revenue-naik-1-persen-terdengar-aman-sampai-angkanya-dipecah-8648597f9aa8?sharedUserId=aoramaaulia)
---

## Latar belakang

Sebuah perusahaan fashion e-commerce menjual lewat Website dan Marketplace dengan lima
kategori produk. Sales Manager melihat revenue tumbuh, tetapi ada empat hal yang belum
dipahami:

1. Revenue tumbuh tetapi tidak merata antar kategori
2. Jumlah customer baru meningkat, tetapi kontribusi repeat customer melemah
3. Beberapa kategori mendapat diskon lebih besar dari yang lain
4. Website dan Marketplace punya karakteristik customer yang berbeda

Keempatnya **tidak diperlakukan sebagai kesimpulan yang tinggal dicarikan pembenarannya**,
melainkan sebagai hipotesis yang harus diuji, termasuk kemungkinan datanya tidak mendukung.

---

## Data

| Tabel | Grain | Baris |
|---|---|---|
| `customers.csv` | satu baris = satu customer | 1.056 |
| `products.csv` | satu baris = satu produk | 108 |
| `order_lines.csv` | satu baris = **satu produk dalam satu order** | 4.598 setelah cleaning |

Periode 1 Januari sampai 30 Juni 2026. Total 2.730 order dari 908 customer yang
bertransaksi.

> **Catatan grain.** Satu order bisa punya beberapa baris. Karena itu jumlah order dihitung
> dengan `COUNT(DISTINCT order_id)`, bukan `COUNT(*)`. Salah di titik ini membuat AOV jauh
> meleset.

---

## Cara pengerjaan

```
Profiling  →  Cleaning per tabel  →  Join  →  Dua tabel fakta  →  KPI  →  Analisis
```

**1 · Profiling sebelum cleaning.** Tidak satu pun aturan cleaning ditulis sebelum melihat
output profiling. Ditemukan sembilan masalah kualitas data, tiga di antaranya tidak
disebutkan di brief.

**2 · Cleaning per tabel, sebelum join.** Kalau join dilakukan lebih dulu, satu `product_id`
duplikat akan menggandakan setiap order line yang memakainya, dan revenue-nya ikut dobel
tanpa jejak.

**3 · Dua tabel fakta dengan grain berbeda.**

| Tabel | Grain | Dipakai untuk |
|---|---|---|
| `fct_order_lines` | order line | Revenue, breakdown kategori dan produk |
| `fct_orders` | order | Completed Orders, Cancellation Rate, AOV |

**4 · Validasi lewat assertion.** Jumlah baris sebelum dan sesudah join harus sama, order
Cancelled harus bernilai nol, dan tidak boleh ada net revenue negatif.

### Keputusan cleaning yang perlu dijelaskan

Ada 24 baris dengan `quantity = 0`. Refleks pertama adalah membuangnya, tetapi dampaknya
diperiksa dulu **di level order**, bukan level baris:

- 19 order masih punya baris valid, jadi aman
- **5 order berstatus Completed hanya punya baris itu saja**, dan akan hilang total

Brief mendefinisikan Completed Orders murni berdasarkan `order_status`, tanpa syarat
kuantitas. Membuangnya justru menyimpang dari definisi. Baris itu **dipertahankan**, dan
dampaknya ke Net Revenue nol karena kuantitas nol menghasilkan revenue nol.

---

## Lima KPI wajib

| KPI | Nilai |
|---|---|
| Gross Sales | Rp 8,03 M |
| Net Revenue | Rp 7,03 M |
| Completed Orders | 2.481 |
| Average Order Value | Rp 2,84 jt |
| Cancellation Rate | 9,12% |

Selisih Gross ke Net sebesar **Rp 1,01 M** adalah nilai yang hilang ke diskon dan
pembatalan, setara 12,5% dari penjualan kotor.

### Kenapa lima KPI ini

Tiga di antaranya terikat secara matematis:

```
Net Revenue = Completed Orders × Average Order Value
```

Begitu revenue bergerak, sumbernya bisa langsung ditelusuri: dari jumlah transaksi atau dari
nilai per transaksi. Dua masalah berbeda dengan penanganan berbeda.

Cancellation Rate melengkapi dari sisi kualitas order, karena tiga metrik di atas hanya
mengukur transaksi yang berhasil.

---

## Temuan

### 1 · Revenue datar, bukan tumbuh

| | Januari | Juni | Perubahan |
|---|---|---|---|
| Net Revenue | Rp 1.188 jt | Rp 1.200 jt | **+1,0%** |
| Completed Orders | 394 | 423 | **+7,4%** |
| Average Order Value | Rp 3,01 jt | Rp 2,84 jt | **−5,9%** |
| Discount rate | 3,11% | 4,03% | puncak 4,40% di Mei |

Jumlah order naik, nilai per order turun. Perusahaan memproses lebih banyak transaksi untuk
hasil yang sama.

### 2 · Akuisisi menutupi erosi retensi

| | Januari | Juni | Perubahan |
|---|---|---|---|
| Customer baru | Rp 502 jt | Rp 826 jt | **+64,7%** |
| Customer lama | Rp 671 jt | Rp 351 jt | **−47,7%** |
| Total | Rp 1.173 jt | Rp 1.177 jt | +0,3% |

Kenaikan Rp 324 juta hampir persis menutup penurunan Rp 320 juta.

**Dekomposisi customer lama.** Revenue satu segmen adalah jumlah orang × frekuensi × nilai
order. Ketiganya diperiksa:

| Komponen | Januari | Juni | Perubahan |
|---|---|---|---|
| **Jumlah orang aktif** | 209 | 128 | **−38,8%** |
| Belanja per orang | Rp 3,55 jt | Rp 3,13 jt | −11,8% |
| Frekuensi order | 1,16 | 1,08 | −6,9% |

Ketiganya menurun, tetapi jumlah orang turun lebih dari tiga kali lipat komponen lain.
**Masalahnya kehadiran, bukan daya beli.**

### 3 · Diskon mengalir ke kategori yang lemah

| Kategori | Share revenue | Share unit | Discount rate |
|---|---|---|---|
| Handbags | **42,5%** | 16,6% | **1,78%** |
| Shoes | 30,5% | 25,8% | **6,09%** |
| Ready-to-Wear | 12,6% | 21,1% | **5,31%** |
| Accessories | 7,9% | **22,4%** | 3,40% |
| Small Leather Goods | 6,4% | 14,0% | 2,51% |

Dua kategori berdiskon terbesar bukan penghasil revenue tertinggi. Korelasi di level produk
**−0,228**, lemah, dengan R-kuadrat hanya 0,052.

> **Batas kesimpulan.** Hubungan ini ditulis sebagai **asosiasi, bukan sebab akibat**. Arah
> sebabnya bisa terbalik: produk yang lambat terjual mungkin justru yang diberi diskon lebih
> besar. Karena itu rekomendasinya dirancang sebagai uji coba bertahap.

### 4 · Pembeda channel bukan customer

| Aspek | Website | Marketplace |
|---|---|---|
| Porsi revenue customer baru | 53,2% | 58,6% |
| Average Order Value | Rp 2,86 jt | Rp 2,79 jt |
| **Cancellation rate** | **10,14%** | **7,69%** |

Dua aspek pertama praktis setara, jadi dugaan bahwa kedua channel punya karakteristik
customer berbeda **tidak terbukti**. Yang justru berbeda adalah kualitas order, dan itu
tidak ada di daftar dugaan.

Dipecah lebih jauh per traffic source, selisihnya lebih tajam lagi:

| Sumber | Cancel rate |
|---|---|
| Paid Social | **13,99%** |
| Paid Search | 11,24% |
| Direct | 10,25% |
| Referral | 8,24% |
| Marketplace | 7,69% |
| Organic Search | **7,43%** |

Trafik berbayar membatalkan 1,9 kali lebih sering daripada organik.

### 5 · Konsentrasi revenue dan struktur asortimen

- **22 dari 108 produk** menyumbang separuh revenue
- Top 10 menyumbang 27,5%, Bottom 10 hanya 1,9%
- Bottom 10 **seluruhnya sudah aktif enam bulan penuh**, jadi revenue rendahnya bukan efek
  produk baru rilis
- Ready-to-Wear punya SKU terbanyak (28) tetapi rata-rata terkecil ketiga (Rp 31,6 jt),
  berbanding Handbags Rp 135,8 jt dari 22 produk

---

## Rekomendasi

Diurutkan berdasarkan dampak terhadap revenue. Masing-masing punya dasar bukti, langkah
awal, dan ukuran keberhasilan.

<table>
<tr><th>#</th><th>Rekomendasi</th><th>Ukuran keberhasilan</th></tr>
<tr>
<td><b>01</b></td>
<td><b>Program reaktivasi customer lama</b><br>
Customer lama aktif turun 209 ke 128 sementara belanja per orang hanya turun 11,8%. Mulai
dari yang terakhir bertransaksi tiga sampai enam bulan lalu, uji satu kanal komunikasi dulu.</td>
<td>Jumlah customer lama aktif per bulan mendekati 200, <b>bukan</b> kenaikan nilai transaksi</td>
</tr>
<tr>
<td><b>02</b></td>
<td><b>Audit pembatalan, mulai dari trafik berbayar</b><br>
Paid Social 13,99% berbanding Organic 7,43%. Pada Juni seluruh lini naik bersamaan ke 12,24%.</td>
<td>Cancellation rate mingguan kembali ke kisaran 8% dalam satu kuartal</td>
</tr>
<tr>
<td><b>03</b></td>
<td><b>Tinjau ulang alokasi diskon</b><br>
Diskon naik 3,11% ke 4,40% sementara AOV turun 5,9%. Kurangi bertahap pada satu kategori,
pertahankan kategori lain sebagai pembanding.</td>
<td>Revenue dan unit kategori uji tidak turun signifikan</td>
</tr>
<tr>
<td><b>04</b></td>
<td><b>Rasionalisasi SKU dan perluas Marketplace</b><br>
Ready-to-Wear 28 produk dengan rata-rata Rp 31,6 jt, berbanding Handbags Rp 135,8 jt dari
22 produk.</td>
<td>Revenue per produk Ready-to-Wear naik, order Marketplace tumbuh tanpa AOV turun</td>
</tr>
</table>

> Menyebut apa yang **bukan** ukuran keberhasilan sama pentingnya dengan menyebut apa yang
> iya. Itu yang mencegah sebuah program diklaim berhasil lewat metrik yang salah.

---

## Limitasi

| Limitasi | Konsekuensi |
|---|---|
| Tidak ada data biaya pokok | Seluruh rekomendasi berdiri di atas revenue, bukan profitabilitas. Ini yang paling mengikat |
| Tidak ada data retur | Peringkat kategori dan produk dibaca sebagai indikasi |
| Tidak ada funnel dan belanja pemasaran | Biaya akuisisi dan ROI kampanye di luar cakupan |
| Hubungan bersifat asosiasi | Arah sebab diskon bisa terbalik |
| Periode hanya enam bulan | Belum memisahkan pola musiman dari tren |

### Asumsi cleaning

| Asumsi | Dampak |
|---|---|
| 83 order line tanpa `customer_id` diberi label `Unknown`, tidak dibuang | Revenue tetap masuk KPI total, setara 1,6% |
| 24 baris `quantity = 0` dipertahankan | Net Revenue tidak berubah, Completed Orders 2.481 bukan 2.476 |
| `discount_amount` kosong dianggap nol | Sesuai aturan eksplisit di brief |
| Ranking produk memakai `product_id`, bukan `product_name` | Ada nama produk yang dipakai dua ID berbeda |
| Casing brand dinormalisasi | 17 varian penulisan menjadi 8 brand |



---


## Dashboard

Dashboard interaktif satu halaman dengan lima filter yang berlaku ke seluruh panel:
**Bulan, Sales Channel, Category, Customer Type, dan Traffic Source.**

🔗 **[Buka dashboard di Tableau Public](https://public.tableau.com/views/Book1_17856012709740/FashionE-CommerceSalesPerformance)**

### Empat keputusan yang didukung

| Keputusan | Panel dan filter yang dipakai |
|---|---|
| Alokasi promosi antar channel | Filter Sales Channel, memperlihatkan AOV kedua channel setara |
| Fokus kategori dan asortimen | Filter Category, memperlihatkan kategori yang menang volume tapi kalah revenue |
| Waktu dan sasaran kampanye retensi | Filter Customer Type dipadu Bulan, memperlihatkan sejak kapan customer lama menyusut |
| Deteksi dini masalah operasional | Kartu Cancellation Rate dipadu filter Traffic Source |


---

### Tautan

- 📓 [Notebook analisis lengkap](https://colab.research.google.com/drive/1KtOxVgaHyKdiv2AD3xC3umwa1bgzKUkr?usp=sharing)
- 📊 [Dashboard interaktif](https://public.tableau.com/views/Book1_17856012709740/FashionE-CommerceSalesPerformance)
- 📑 [Presentasi 5 slide](https://drive.google.com/file/d/1Z7z2hD7AA-dnxt1DfuSRAW0IlcHdBS61/view?usp=sharing)

---

<sub>Studi kasus dari <b>WINDATA</b>. Dikerjakan sebagai bagian dari assessment
Entry-Level Data Analyst.</sub>
