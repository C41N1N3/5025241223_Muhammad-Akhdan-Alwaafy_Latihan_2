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
    FILE *dst = fopen("hai.txt", "w");
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
https://drive.google.com/file/d/1m1PNc5oI1ZQ5HqEAIkskzpE8Hy-iArDz/view?usp=drive_link

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
    FILE *log_clear = fopen("log.txt", "w")
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

### Contoh Isi `log.txt`
```
1. Thread 2
2. Thread 1
3. Thread 3
```

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

### Contoh Isi `log.txt`
```
thread 1 count 1
thread 1 count 2
thread 1 count 3
thread 2 count 1
...
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
