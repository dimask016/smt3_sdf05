
TUGAS SESI 2 - Prinsip Clean Code:
Penamaan & Fungsi
Universitas Cakrawala . Prodi Ilmu Komputer IK102 Topik: Clean code - penamaan dan
fungsi. Bobot: 5% Individu Praktikum di kelas Deadline: sebelum Sesi 3 Sub-CPMK:
Mahasiswa mampu menerapkan prinsip clean code dalam penamaan dan fungsi.
1. Analisis kode awal

Kode awal:

```python
def f(a, b, c, d, e):
    x = a * b
    s = 0
    for i in c:
        s = s + i
    if s > 100:
        x = x * 0.9
    else:
        x = x * 0.95
    t = 0
    for i in c:
        t = t + i
    if d:
        x = x + 0
    else:
        x = x + e
    return x
```

Masalah utamanya:

· Nama f, a, b, c, d, e, x, s, t tidak menjelaskan maksud.
· Fungsi melakukan banyak hal sekaligus: hitung subtotal, hitung total item, tentukan diskon, hitung ongkir.
· Ada duplikasi: s dan t sama-sama menjumlahkan c.
· t sebenarnya tidak dipakai setelah dihitung.

Semantik yang harus dipertahankan:

total pesanan = (harga_satuan × jumlah) × diskon + ongkir
diskon 10% jika total item > 100, selain itu 5%
ongkir gratis jika member, selain itu biaya kirim.

---

2. Bagian A — Perbaikan penamaan

Nama lama Nama baru Alasan
f hitung_total_pesanan Menjelaskan tujuan fungsi
a harga_satuan Harga per item
b jumlah Jumlah barang yang dibeli
c daftar_kuantitas_item List kuantitas item untuk dijumlahkan
d is_member Status member
e biaya_kirim Ongkos kirim
x subtotal, total_setelah_diskon, total_pesanan Sesuai tahap perhitungan
s, t total_item Cukup satu variabel, karena t duplikat
i kuantitas_item Item yang sedang dijumlahkan

Nama yang paling penting menurut saya: hitung_total_pesanan dan daftar_kuantitas_item.
hitung_total_pesanan langsung menjelaskan tujuan utama fungsi, sedangkan daftar_kuantitas_item memperjelas bahwa parameter c adalah kumpulan kuantitas item yang harus dijumlahkan.

---

3. Bagian B — Refactor fungsi

Identifikasi “satu hal”

Fungsi utama sebenarnya melakukan satu hal besar: menghitung total pesanan.
Namun di dalamnya ada beberapa tanggung jawab kecil:

1. Menghitung subtotal.
2. Menghitung total item.
3. Menghitung total setelah diskon.
4. Menghitung ongkir.

Maka kita pecah menjadi fungsi-fungsi kecil.

Kode hasil refactor

```python
def hitung_subtotal(harga_satuan, jumlah):
    """Menghitung subtotal sebelum diskon dan ongkir."""
    return harga_satuan * jumlah


def hitung_total_item(daftar_kuantitas_item):
    """Menjumlahkan seluruh kuantitas item."""
    return sum(daftar_kuantitas_item)


def hitung_total_setelah_diskon(subtotal, total_item):
    """
    Menghitung total setelah diskon.
    Diskon 10% jika total_item > 100, selain itu 5%.
    """
    if total_item > 100:
        return subtotal * 0.9   # bayar 90%, diskon 10%
    return subtotal * 0.95      # bayar 95%, diskon 5%


def hitung_ongkir(is_member, biaya_kirim):
    """Member gratis ongkir; non-member dikenakan biaya kirim."""
    if is_member:
        return 0
    return biaya_kirim


def hitung_total_pesanan(
    harga_satuan,
    jumlah,
    daftar_kuantitas_item,
    is_member,
    biaya_kirim,
):
    """Menghitung total pesanan akhir."""
    subtotal = hitung_subtotal(harga_satuan, jumlah)
    total_item = hitung_total_item(daftar_kuantitas_item)
    total_setelah_diskon = hitung_total_setelah_diskon(subtotal, total_item)
    ongkir = hitung_ongkir(is_member, biaya_kirim)

    return total_setelah_diskon + ongkir
```

Duplikasi s dan t dihapus. Sekarang total item hanya dihitung sekali melalui hitung_total_item.

---

4. Uji perilaku tidak berubah

Contoh pengujian:

```python
if __name__ == "__main__":
    # total item 110 > 100, member => diskon 10%, ongkir gratis
    assert hitung_total_pesanan(10000, 2, [60, 50], True, 15000) == 18000.0

    # total item 50 <= 100, non-member => diskon 5%, ongkir 15000
    assert hitung_total_pesanan(5000, 2, [20, 30], False, 15000) == 24500.0

    # total item tepat 100 => diskon 5%, member => ongkir gratis
    assert hitung_total_pesanan(1000, 5, [100], True, 20000) == 4750.0

    print("Semua uji perilaku sama.")
```

Perbandingan manual:

Input Kode awal Kode refactor
(10000, 2, [60,50], True, 15000) 18000 18000
(5000, 2, [20,30], False, 15000) 24500 24500
(1000, 5, [100], True, 20000) 4750 4750

---

5. Bagian C — Refleksi 3 kalimat

1. Keputusan penamaan terpenting saya adalah mengganti f menjadi hitung_total_pesanan dan c menjadi daftar_kuantitas_item, karena keduanya langsung menjelaskan tujuan fungsi dan asal nilai total_item.
2. Saya memastikan perilaku tidak berubah dengan mempertahankan rumus asli: subtotal = harga_satuan * jumlah, total item = sum(daftar_kuantitas_item), diskon memakai faktor 0.9 atau 0.95, dan ongkir 0 jika member.
3. Setelah refactor, saya menguji beberapa kasus, yaitu total item > 100, total item <= 100, member, dan non-member, dan hasilnya sama dengan kode awal.



---

code phyton hasil refactor : https://colab.research.google.com/drive/1r6D_bHwcTele3anMwh10MtHuXUmT95nq#scrollTo=uvERLTPW2Sm_



