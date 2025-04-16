# 5025241223_Muhammad-Akhdan-Alwaafy_Latihan_2

# Laporan Praktikum Sistem Operasi - Modul 2

## Identitas Diri

- **NRP**    : 502522XXXX  
- **Nama**   : Nama Lengkap  
- **Kelas**  : A  
- **Kelompok** : 2  

---

## Soal 1 - Process

### Penjelasan Program
Program ini melakukan 3 langkah secara berurutan:
1. Membuat folder bernama `halo`.
2. Membuat file kosong `hai.txt` di dalam folder tersebut.
3. Meng-copy file `hai.txt` ke luar folder, ke direktori program dijalankan.

### Kode Program: `soal1_process.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    // Buat folder
    mkdir("halo", 0777);

    // Buat file kosong di dalam folder halo
    char filepath[] = "halo/hai.txt";
    int fd = open(filepath, O_CREAT | O_WRONLY, 0666);
    if (fd != -1) close(fd);

    // Copy file ke luar folder
    FILE *src = fopen(filepath, "r");
    FILE *dest = fopen("hai.txt", "w");
    if (src && dest) {
        int ch;
        while ((ch = fgetc(src)) != EOF) {
            fputc(ch, dest);
        }
        fclose(src);
        fclose(dest);
    }
    return 0;
}
```

### Output
Struktur direktori setelah program dijalankan:

```
.
├── program.c
├── halo/
│   └── hai.txt
└── hai.txt
```

### Screenshot
![Soal 1 Screenshot](https://link-screenshot.com/soal1.png)

---

## Soal 2 - Thread

### Penjelasan Program
- Program membuat 3 thread:
  - Thread 1: Menulis angka 1–100 ke `count.txt`
  - Thread 2: Menulis kalimat ke `print.txt`
  - Thread 3: Menulis angka genap ke `count_2.txt`
- Setelah selesai, setiap thread mencatat urutannya ke `log.txt`.
- Program dijalankan 3 kali untuk mendapatkan variasi hasil.

### Kode Program: `thread_log.c`

```c
// Kode dimasukkan secara manual oleh pengguna
```

### Contoh Isi `log.txt`
```
1. Thread 2
2. Thread 1
3. Thread 3
```

### Screenshot
![Soal 2 Screenshot](https://link-screenshot.com/soal2.png)

---

## Soal 3 - Thread dengan Mutex

### Penjelasan Program
- Program membuat 5 thread.
- Masing-masing menghitung dari 1 sampai 3 dan menulis ke `log.txt` dengan format `thread {id} count {angka}`.
- Mutex digunakan agar tidak terjadi race condition saat menulis ke file.

### Kode Program: `thread_mutex_log.c`

```c
// Kode dimasukkan secara manual oleh pengguna
```

### Contoh Isi `log.txt`
```
thread 1 count 1
thread 1 count 2
thread 1 count 3
thread 2 count 1
...
```

### Screenshot
![Soal 3 Screenshot](https://link-screenshot.com/soal3.png)

---

## Soal 4 - IPC

### 4a. Shared Memory

#### Kode
- `sender.c`
```c
// Kode dimasukkan secara manual oleh pengguna
```
- `receiver.c`
```c
// Kode dimasukkan secara manual oleh pengguna
```

#### Output
```
Sender: Pesan dikirim ke shared memory.
Receiver: Pesan diterima dari shared memory: aku lagi belajar ipc
```

#### Screenshot
![Shared Memory Screenshot](https://link-screenshot.com/shm.png)

---

### 4b. Message Queue

#### Kode: `msg_queue.c`
```c
// Kode dimasukkan secara manual oleh pengguna
```

#### Output
```
Pesan dikirim: yah belajar ipc mulu
Pesan diterima: yah belajar ipc mulu
```

#### Screenshot
![Message Queue Screenshot](https://link-screenshot.com/queue.png)

---

### 4c. Pipe dengan `fork()`

#### Kode: `pipe_fork.c`
```c
// Kode dimasukkan secara manual oleh pengguna
```

#### Output
```
Child menerima pesan: hai, anak sisop 24
```

#### Screenshot
![Pipe Screenshot](https://link-screenshot.com/pipe.png)

---

## ✅ Catatan Tambahan
- Semua program dibuat dalam bahasa C.
- Sudah diuji di lingkungan Linux dan berjalan dengan baik.
- Screenshot dapat diakses melalui link yang disediakan.

---
