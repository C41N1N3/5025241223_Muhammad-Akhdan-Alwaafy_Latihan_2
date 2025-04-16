# 5025241223_Muhammad-Akhdan-Alwaafy_Latihan_2

# Laporan Praktikum Sistem Operasi - Modul 2

## Identitas Diri

- **NRP**    : 5025241223
- **Nama**   : Muhammad Akhdan Alwaafy
- **Kelas**  : E
- **Kelompok** : E10

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
#include <string.h>

int main() {
    int stats;
    stats = mkdir("halo", 0777);
    if (stats == -1) {
        perror("folder creation failed");
        return 1;
    } else {
        printf("folder halo is done\n");
    }
    char filepath[100] = "hai.txt";
    int file = open(filepath, O_CREAT | O_WRONLY, 0644);
    if (file == -1) {
        perror("hai.txt creation failed");
        return 1;
    } else {
        printf("hai.txt done\n");
    }
    close(file);
    FILE *src = fopen("hai.txt", "r");
    FILE *dst = fopen("halo/hai.txt", "w");
    if (src == NULL || dst == NULL) {
        perror("error copy");
        return 1;
    }
    char ch;
    while ((ch = fgetc(src)) != EOF) {
        fputc(ch, dst);
    }
    fclose(src);
    fclose(dst);
    printf("success");
    return 0;
}
```

### Tutorial

---

### Header

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
```

Header-header ini digunakan untuk berbagai fungsi:
- `stdio.h` → untuk fungsi input/output standar (`printf`, `fopen`, dll.)
- `stdlib.h` → fungsi umum seperti `exit()`, alokasi memori, dll.
- `sys/stat.h`, `sys/types.h` → untuk fungsi `mkdir()` dan pengaturan mode akses file.
- `unistd.h` → untuk fungsi `close()`, `write()`, dll.
- `fcntl.h` → untuk `open()` dan flags seperti `O_CREAT`, `O_WRONLY`.
- `string.h` → untuk manipulasi string, walaupun di sini tidak digunakan.

---

### Membuat folder bernama `halo`

```c
int stats;
stats = mkdir("halo", 0777);
if (stats == -1) {
    perror("folder creation failed");
    return 1;
} else {
    printf("folder halo is done\n");
}
```

- `mkdir("halo", 0777)` membuat folder bernama `halo` dengan permission full akses (baca, tulis, eksekusi untuk semua user).
- Jika gagal, program akan menampilkan error menggunakan `perror`.
- Jika berhasil, akan mencetak `folder halo is done`.

---

### Membuat file `hai.txt`

```c
char filepath[100] = "hai.txt";
int file = open(filepath, O_CREAT | O_WRONLY, 0644);
if (file == -1) {
    perror("hai.txt creation failed");
    return 1;
} else {
    printf("hai.txt done\n");
}
close(file);
```

- Membuat file baru bernama `hai.txt` dengan mode tulis (`O_WRONLY`) dan jika file belum ada akan dibuat (`O_CREAT`).
- Permission file adalah `0644` (owner: read-write, group: read, others: read).
- Jika berhasil akan menampilkan `hai.txt done`, lalu file ditutup.

---

### Membuka file `hai.txt` dua kali

```c
FILE *src = fopen("hai.txt", "r");
FILE *dst = fopen("halo/hai.txt", "w");
if (src == NULL || dst == NULL) {
    perror("error copy");
    return 1;
}
```

- `src` dibuka dengan mode baca (`r`)
- `dst` dibuka dengan mode tulis (`w`) → **Hati-hati!** mode `w` langsung mengosongkan file!
- Karena `hai.txt` belum diisi data apa pun sebelumnya, file akan kosong sebelum proses penyalinan dimulai → **penyalinan akan gagal karena tidak ada yang bisa dibaca**.

---

### Penyalinan isi file (yang sebenarnya kosong)

```c
char ch;
while ((ch = fgetc(src)) != EOF) {
    fputc(ch, dst);
}
```

- Melakukan pembacaan per karakter dari `src` dan menulis ke `dst`.
- Tapi karena `dst` dibuka lebih dulu dalam mode `w`, isinya sudah kosong → proses ini tidak melakukan apa-apa.

---

### Menutup file dan mencetak `success`

```c
fclose(src);
fclose(dst);
printf("success");
```

---

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
https://drive.google.com/file/d/1DvA_8bM1XrR5c77XHcabIbX4UTi07h9N/view?usp=drive_link

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
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

pthread_mutex_t log_mutex;
int log_counter = 0;

void log_thread(const char* thread_name) {
    pthread_mutex_lock(&log_mutex);
    log_counter++;
    FILE *log_file = fopen("log.txt", "a");
    if (log_file != NULL) {
        fprintf(log_file, "%d. %s\n", log_counter, thread_name);
        fclose(log_file);
    }
    pthread_mutex_unlock(&log_mutex);
}

void* thread1_func(void* arg) {
    FILE *f = fopen("count.txt", "w");
    if (f != NULL) {
        for (int i = 1; i <= 100; i++) {
            fprintf(f, "%d\n", i);
        }
        fclose(f);
    }
    log_thread("Thread 1");
    return NULL;
}

void* thread2_func(void* arg) {
    FILE *f = fopen("print.txt", "w");
    if (f != NULL) {
        fprintf(f, "Saya pintar mengerjakan thread\n");
        fclose(f);
    }
    log_thread("Thread 2");
    return NULL;
}

void* thread3_func(void* arg) {
    FILE *f = fopen("count_2.txt", "w");
    if (f != NULL) {
        for (int i = 2; i <= 100; i += 2) {
            fprintf(f, "%d\n", i);
        }
        fclose(f);
    }
    log_thread("Thread 3");
    return NULL;
}

int main() {
    pthread_t t1, t2, t3;
    pthread_mutex_init(&log_mutex, NULL);
    FILE *log_clear = fopen("log.txt", "w");
    if (log_clear != NULL) fclose(log_clear);
    pthread_create(&t1, NULL, thread1_func, NULL);
    pthread_create(&t2, NULL, thread2_func, NULL);
    pthread_create(&t3, NULL, thread3_func, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);
    pthread_mutex_destroy(&log_mutex);
    return 0;
}
```

### Tutorial

---

## Header & Variabel Global

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

pthread_mutex_t log_mutex;
int log_counter = 0;
```

- `pthread.h`: untuk pemrograman multithread (POSIX thread).
- `pthread_mutex_t log_mutex`: mutex digunakan agar akses ke log tidak tumpang tindih antar-thread.
- `log_counter`: penghitung jumlah log yang ditulis ke `log.txt`.

---

## Fungsi `log_thread`

```c
void log_thread(const char* thread_name) {
    pthread_mutex_lock(&log_mutex);
    log_counter++;
    FILE *log_file = fopen("log.txt", "a");
    if (log_file != NULL) {
        fprintf(log_file, "%d. %s\n", log_counter, thread_name);
        fclose(log_file);
    }
    pthread_mutex_unlock(&log_mutex);
}
```

- Fungsi ini digunakan semua thread untuk mencatat dirinya ke `log.txt`.
- Mutex (`log_mutex`) mencegah dua thread menulis ke file log secara bersamaan (race condition).
- Menulis `N. Thread X` ke `log.txt`, di mana `N` adalah `log_counter` saat ini.

---

## Thread 1: Menulis angka 1-100 ke `count.txt`

```c
void* thread1_func(void* arg) {
    FILE *f = fopen("count.txt", "w");
    if (f != NULL) {
        for (int i = 1; i <= 100; i++) {
            fprintf(f, "%d\n", i);
        }
        fclose(f);
    }
    log_thread("Thread 1");
    return NULL;
}
```

---

## Thread 2: Menulis kalimat ke `print.txt`

```c
void* thread2_func(void* arg) {
    FILE *f = fopen("print.txt", "w");
    if (f != NULL) {
        fprintf(f, "Saya pintar mengerjakan thread\n");
        fclose(f);
    }
    log_thread("Thread 2");
    return NULL;
}
```

---

## Thread 3: Menulis angka genap 2–100 ke `count_2.txt`

```c
void* thread3_func(void* arg) {
    FILE *f = fopen("count_2.txt", "w");
    if (f != NULL) {
        for (int i = 2; i <= 100; i += 2) {
            fprintf(f, "%d\n", i);
        }
        fclose(f);
    }
    log_thread("Thread 3");
    return NULL;
}
```

---

## Fungsi `main`: Menjalankan semua thread

```c
int main() {
    pthread_t t1, t2, t3;

    pthread_mutex_init(&log_mutex, NULL);  // Inisialisasi mutex

    FILE *log_clear = fopen("log.txt", "w");  // Kosongkan log.txt sebelum mulai
    if (log_clear != NULL) fclose(log_clear);

    pthread_create(&t1, NULL, thread1_func, NULL);
    pthread_create(&t2, NULL, thread2_func, NULL);
    pthread_create(&t3, NULL, thread3_func, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);

    pthread_mutex_destroy(&log_mutex);  // Hapus mutex
    return 0;
}
```

- Tiga thread dijalankan secara paralel.
- `pthread_join` memastikan `main()` menunggu semua thread selesai.
- `log.txt` dikosongkan di awal (agar tidak terus menumpuk dari eksekusi sebelumnya).
- Mutex dihancurkan setelah semua selesai.

---

## 📝 Hasil program

Setelah dijalankan, kamu akan mendapat:

- `count.txt` → angka 1–100
- `count_2.txt` → angka genap 2–100
- `print.txt` → teks `"Saya pintar mengerjakan thread"`
- `log.txt` → urutan thread yang selesai, misalnya:

  ```
  1. Thread 2
  2. Thread 1
  3. Thread 3
  ```

Urutan log bisa berbeda-beda tiap kali dijalankan karena eksekusi thread tidak pasti (nondeterministic).

---

### Screenshot
https://drive.google.com/file/d/1ol_FwGVRvcOZJ_fIdrlZ-qiyUHSQjIvf/view?usp=drive_link

---

## Soal 3 - Thread dengan Mutex

### Penjelasan Program
- Program membuat 5 thread.
- Masing-masing menghitung dari 1 sampai 3 dan menulis ke `log.txt` dengan format `thread {id} count {angka}`.
- Mutex digunakan agar tidak terjadi race condition saat menulis ke file.

### Kode Program: `thread_mutex_log.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define THREAD_COUNT 5

pthread_mutex_t lock;

void* count_and_log(void* arg) {
    int id = *(int*)arg;
    pthread_mutex_lock(&lock);
    for (int i = 1; i <= 3; i++) {
        FILE *f = fopen("log.txt", "a");
        if (f != NULL) {
            fprintf(f, "thread %d count %d\n", id, i);
            fclose(f);
        }
        sleep(1);
    }
    pthread_mutex_unlock(&lock);
    free(arg);
    return NULL;
}

int main() {
    pthread_t threads[THREAD_COUNT];
    pthread_mutex_init(&lock, NULL);
    FILE *clear = fopen("log.txt", "w");
    if (clear != NULL) fclose(clear);
    for (int i = 0; i < THREAD_COUNT; i++) {
        int* id = malloc(sizeof(int));
        *id = i + 1;
        pthread_create(&threads[i], NULL, count_and_log, id);
    }
    for (int i = 0; i < THREAD_COUNT; i++) {
        pthread_join(threads[i], NULL);
    }
    pthread_mutex_destroy(&lock);
    return 0;
}
```

### Tutorial

### 1. Header & Global

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define THREAD_COUNT 5

pthread_mutex_t lock;
```

- `pthread.h`: untuk multithreading
- `unistd.h`: untuk `sleep()`
- `lock`: mutex global untuk mengontrol akses ke `log.txt`
- `THREAD_COUNT`: konstanta untuk jumlah thread (5 thread)

---

### 2. Fungsi Thread

```c
void* count_and_log(void* arg) {
    int id = *(int*)arg;
    pthread_mutex_lock(&lock);
    for (int i = 1; i <= 3; i++) {
        FILE *f = fopen("log.txt", "a");
        if (f != NULL) {
            fprintf(f, "thread %d count %d\n", id, i);
            fclose(f);
        }
        sleep(1);
    }
    pthread_mutex_unlock(&lock);
    free(arg);
    return NULL;
}
```

- `id = *(int*)arg` → ambil ID unik dari pointer argumen.
- `pthread_mutex_lock()` → thread ini memegang kunci dulu.
- Di dalam loop `1 sampai 3`, thread menulis ke `log.txt` dan tidur 1 detik.
- Setelah selesai, `pthread_mutex_unlock()` agar thread lain bisa masuk.
- `free(arg)` → karena ID di-`malloc`, wajib di-`free`.

> Karena seluruh `for` loop berada **di dalam mutex**, maka setiap thread **menulis 3 baris sekaligus secara eksklusif**.

---

### 3. Fungsi `main`

```c
int main() {
    pthread_t threads[THREAD_COUNT];
    pthread_mutex_init(&lock, NULL);
    FILE *clear = fopen("log.txt", "w");
    if (clear != NULL) fclose(clear);
```

- Inisialisasi mutex.
- Bersihkan isi file `log.txt` dengan membuka dalam mode tulis (`w`), lalu ditutup langsung.

```c
    for (int i = 0; i < THREAD_COUNT; i++) {
        int* id = malloc(sizeof(int));
        *id = i + 1;
        pthread_create(&threads[i], NULL, count_and_log, id);
    }
```

- Loop untuk membuat 5 thread.
- Masing-masing `id` disiapkan pakai `malloc` supaya bisa di-passing sebagai pointer unik untuk tiap thread.

```c
    for (int i = 0; i < THREAD_COUNT; i++) {
        pthread_join(threads[i], NULL);
    }
    pthread_mutex_destroy(&lock);
    return 0;
}
```

- Tunggu semua thread selesai.
- Hancurkan mutex setelah semua thread selesai.

---

```

### Screenshot
https://drive.google.com/file/d/1WIp6vwhMleeL3l25rqp648GPAMCkzRH-/view?usp=drive_link

---

## Soal 4 - IPC

### 4a. Shared Memory

#### Kode
- `sender.c`
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <string.h>

int main() {
    key_t key = 1234;
    int shmid = shmget(key, 1024, IPC_CREAT | 0666);
    if (shmid < 0) {
        perror("shmget");
        return 1;
    }
    char *data = (char *)shmat(shmid, NULL, 0);
    if (data == (char *)(-1)) {
        perror("shmat");
        return 1;
    }
    strcpy(data, "aku lagi belajar ipc");
    shmdt(data);
    return 0;
}
```
- `receiver.c`
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>

int main() {
    key_t key = 1234;
    int shmid = shmget(key, 1024, 0666);
    if (shmid < 0) {
        perror("shmget");
        return 1;
    }
    char *data = (char *)shmat(shmid, NULL, 0);
    if (data == (char *)(-1)) {
        perror("shmat");
        return 1;
    }
    printf("%s\n", data);
    shmdt(data);
    shmctl(shmid, IPC_RMID, NULL);
    return 0;
}

```

#### Output
```
Sender: Pesan dikirim ke shared memory.
Receiver: Pesan diterima dari shared memory: aku lagi belajar ipc
```

#### Screenshot
https://drive.google.com/file/d/1fCTRoVK8vJMXdmWgciDeX0eUJ6wMFAx7/view?usp=drive_link

---

### 4b. Message Queue

#### Kode: `sisoplatsol4b.c`
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct message {
    long msg_type;
    char msg_text[100];
};

int main() {
    key_t key = 1234;
    int msgid = msgget(key, IPC_CREAT | 0666);
    if (msgid == -1) {
        perror("msgget");
        return 1;
    }
    struct message msg;
    msg.msg_type = 1;
    strcpy(msg.msg_text, "yah belajar ipc mulu");
    msgsnd(msgid, &msg, sizeof(msg.msg_text), 0);
    printf("sent: %s\n", msg.msg_text);
    msgrcv(msgid, &msg, sizeof(msg.msg_text), 1, 0);
    printf("received: %s\n", msg.msg_text);
    msgctl(msgid, IPC_RMID, NULL);
    return 0;
}
```

#### Output
```
Pesan dikirim: yah belajar ipc mulu
Pesan diterima: yah belajar ipc mulu
```

#### Screenshot
https://drive.google.com/file/d/19HlRLClpDa873nHbaULbR3Vr7s-iHwu5/view?usp=drive_link

---

### 4c. Pipe dengan `fork()`

#### Kode: `sisoplatsol4c.c`
```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd[2];
    pipe(fd);
    pid_t pid = fork();
    if (pid > 0) {
        close(fd[0]);
        char pesan[] = "hai, anak sisop 24";
        write(fd[1], pesan, strlen(pesan) + 1);
        close(fd[1]);
    } else if (pid == 0) {
        close(fd[1]);
        char buffer[100];
        read(fd[0], buffer, sizeof(buffer));
        printf("%s\n", buffer);
        close(fd[0]);
    }
    return 0;
}
```

#### Output
```
hai, anak sisop 24
```

#### Screenshot
https://drive.google.com/file/d/1H-8nXiQVzxoWB4ZTXqTZ8gyIhzEJLlPP/view?usp=drive_link
