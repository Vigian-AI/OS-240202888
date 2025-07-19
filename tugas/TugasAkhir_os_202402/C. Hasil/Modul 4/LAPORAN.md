
# 📝 Laporan Tugas Akhir

**Mata Kuliah**: Sistem Operasi  
**Semester**: Genap / Tahun Ajaran 2024–2025  
**Nama**: Vigian Agus Isnaeni  
**NIM**: 240202888  
**Modul yang Dikerjakan**:  
Modul 4 – Subsistem Kernel Alternatif (xv6-public)

---

## 📌 Deskripsi Singkat Tugas

* **Modul 4 – Subsistem Kernel Alternatif**:  
  Menambahkan dua fitur ke dalam kernel xv6, yaitu syscall `chmod(path, mode)` untuk mengatur mode file (`read-only` atau `read-write`), dan menambahkan pseudo-device `/dev/random` untuk menghasilkan byte acak menggunakan driver sederhana.

---

## 🛠️ Rincian Implementasi

* Menambahkan field `short mode` pada struktur `inode` di `fs.h` sebagai indikator akses file (0 = read-write, 1 = read-only)
* Menambahkan syscall `chmod()` dengan:
  - Nomor syscall baru pada `syscall.h`
  - Deklarasi fungsi di `user.h` dan `usys.S`
  - Implementasi fungsi `sys_chmod()` di `sysfile.c`
  - Registrasi syscall pada `syscall.c`
* Menambahkan pengecekan mode akses file pada `filewrite()` di `file.c`
* Membuat program uji `chmodtest.c` untuk mengecek pembatasan write ke file dengan mode read-only
* Menambahkan file baru `random.c` berisi implementasi pseudo-device `/dev/random`:
  - Fungsi `randomread()` menghasilkan byte acak dengan algoritma PRNG sederhana
* Registrasi driver ke `devsw[]` di `file.c` menggunakan major number 3
* Membuat device node `/dev/random` pada `init.c` menggunakan `mknod()`
* Membuat program uji `randomtest.c` untuk membaca 8 byte dari `/dev/random`
* Menambahkan program uji ke `Makefile` melalui `UPROGS`

---

## ✅ Uji Fungsionalitas

* `chmodtest`: membuat file, mengatur menjadi read-only, lalu mencoba menulis kembali
* `randomtest`: membuka `/dev/random` dan mencetak 8 byte acak

---

## 📷 Hasil Uji

### 📍 Contoh Output `chmodtest`:


### 📸 Screenshot:
![hasil ptest dan rtest](./screenshots/4.png)

---

## ⚠️ Kendala yang Dihadapi

 **Kesalahan inisialisasi `devsw`**  
   Inisialisasi `devsw` seharusnya menggunakan deklarasi tanpa ukuran tetap:  
 **Konflik fungsi `fileread`, `filewrite`, `pipe` di `devsw`**  
   Terjadi kesalahan saat mendaftarkan device di array `devsw[]`. Beberapa implementasi keliru menggunakan fungsi high-level seperti `filewrite()` atau `piperead()` untuk entri device driver. Fungsi-fungsi ini tidak semestinya ada di `devsw` karena akan bentrok dengan sistem file dan pipa yang sudah ada. Solusinya adalah **hapus entri `devsw[1]` dan `devsw[2]`**, dan hanya gunakan entri `devsw[3]` untuk device `/dev/random` dengan fungsi `randomread()`.

 **`randomread` tidak dikenali saat linking**  
   
 **Gagal membuka `/dev/random`**  
   Program user gagal membuka path `/dev/random` karena file node yang dibuat oleh `mknod()` mungkin memiliki nama berbeda, misalnya `/random`. Akibatnya, `open("/dev/random", 0)` gagal. 

 **Program `chmodtest` tidak berhenti / infinite loop**  
   Setelah mengatur file ke `read-only` dan gagal menulis (sesuai harapan), program tidak mengubah mode kembali ke `read-write`, sehingga tidak memiliki akhir yang jelas. Akibatnya terlihat seperti tidak selesai. 

---

## 📚 Referensi

* Buku xv6 MIT: [https://pdos.csail.mit.edu/6.828/2018/xv6/book-rev11.pdf](https://pdos.csail.mit.edu/6.828/2018/xv6/book-rev11.pdf)
* Repositori xv6-public: [https://github.com/mit-pdos/xv6-public](https://github.com/mit-pdos/xv6-public)
* Diskusi praktikum dan dokumentasi di Stack Overflow dan GitHub Issues

---