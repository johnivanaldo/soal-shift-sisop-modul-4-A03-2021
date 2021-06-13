# soal-shift-sisop-modul-4-A03-2021

Anggota : 

### 1. Naufal Fajar  I  05111940000007
### 2. Johnivan Aldo S  05111940000051 
### 3. Abdun Nafi'      05111940000066
***

## Soal No 1

untuk pertama kita buat struct dan fungsi main sebagi berikut :
```

static struct fuse_operations xmp_oper = {
    .getattr = xmp_getattr,
    .readdir = xmp_readdir,
    .read = xmp_read,
    .mkdir = xmp_mkdir,
    .mknod = xmp_mknod,
    .unlink = xmp_unlink,
    .rmdir = xmp_rmdir,
    .rename = xmp_rename,
    .open = xmp_open,
    .write = xmp_write,
};

int main(int argc, char *argv[]) {
    umask(0);

    return fuse_main(argc, argv, &xmp_oper, NULL);
}
```

Sebelumnya untuk mengubah nama file dnegan atbash cipher kita membuat fungsi sebagai berikut :
```
void atbash(char *str, int start, int end) {
    for (int i = start; i < end; i++) {
        if (str[i] == '/') continue;
        if (i != start && str[i] == '.') break;

        if (str[i] >= 'A' && str[i] <= 'Z')
            str[i] = 'Z' + 'A' - str[i];
        else if (str[i] >= 'a' && str[i] <= 'z')
            str[i] = 'z' + 'a' - str[i];
    }
}

void encode_atbash(char *str) {
    if (!strcmp(str, ".") || !strcmp(str, "..")) return;
    atbash(str, 0, strlen(str));

    printf("==== enc:atb:%s\n", str);
}

void decode_atbash(char *str) {
    if (!strcmp(str, ".") || !strcmp(str, "..")) return;
    if (strstr(str, "/") == NULL) return;
    printf("==== before:%s\n", str);
    int str_length = strlen(str), s = 0, i;
    for (i = str_length; i >= 0; i--) {
        if (str[i] == '/') break;

        if (str[i] == '.') {
            str_length = i;
            break;
        }
    }
    for (i = 0; i < str_length; i++) {
        if (str[i] == '/') {
            s = i + 1;
            break;
        }
    }

    atbash(str, s, str_length);
    printf("==== dec:atb:%s\n", str);
}
```
### 1a
Jika sebuah direktori dibuat dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.
untuk sourcode dari soal ini sebagai berikut : 

```
void encode_atbash(char *str) {
    if (!strcmp(str, ".") || !strcmp(str, "..")) return;
    atbash(str, 0, strlen(str));

    printf("==== enc:atb:%s\n", str);
} 
```

### 1b
Jika sebuah direktori di-rename dengan awalan “AtoZ_”, maka direktori tersebut akan menjadi direktori ter-encode.
untuk sourcode perintah rename dengan atoz menggungakn fungsi berikut :

```
static int xmp_rename(const char *from, const char *to) {
    printf("rename\n");
    char fileFrom[1000], fileTo[1000];
    sprintf(fileFrom, "%s%s", dirpath, from);
    sprintf(fileTo, "%s%s", dirpath, to);

    log_v1(fileFrom, fileTo);

    int res = rename(fileFrom, fileTo);

    char str[100];
    sprintf(str, "RENAME::%s::%s", from, to);
    log_v2(str, INFO);

    if (res == -1) return -errno;
    return 0;
}

```

### 1c
Apabila direktori yang terenkripsi di-rename menjadi tidak ter-encode, maka isi direktori tersebut akan terdecode.
untuk sourcode perintah decode sebagai berikut :

```
void decode_atbash(char *str) {
    if (!strcmp(str, ".") || !strcmp(str, "..")) return;
    if (strstr(str, "/") == NULL) return;
    printf("==== before:%s\n", str);
    int str_length = strlen(str), s = 0, i;
    for (i = str_length; i >= 0; i--) {
        if (str[i] == '/') break;

        if (str[i] == '.') {
            str_length = i;
            break;
        }
    }
    for (i = 0; i < str_length; i++) {
        if (str[i] == '/') {
            s = i + 1;
            break;
        }
    }

    atbash(str, s, str_length);
    printf("==== dec:atb:%s\n", str);
}

```

### 1d 
Setiap pembuatan direktori ter-encode (mkdir atau rename) akan tercatat ke sebuah log. Format : /home/[USER]/Downloads/[Nama Direktori] → /home/[USER]/Downloads/AtoZ_[Nama Direktori]

Untuk pembuatan  log kita bisa membuat dengan funsi sebagai berikut dan membuat string penempatan
```
static char *dirpath = "/home/wisnupramoedya/Downloads";
static char *logpath = "/home/wisnupramoedya/SinSeiFS.log";
```

```
void log_v1(char *from, char *to) {
    int i;
    for (i = strlen(to); i >= 0; i--) {
        if (to[i] == '/') break;
    }
    if (strstr(to + i, ATOZ) == NULL) return;

    FILE *log_file = fopen(logpath, "a");
    fprintf(log_file, "%s -> %s\n", from, to);
}

void log_v2(char *str, int type) {
    FILE *log_file = fopen(logpath, "a");

    time_t current_time;
    time(&current_time);
    struct tm *time_info;
    time_info = localtime(&current_time);

    if (type == INFO) {
        fprintf(log_file, "INFO::%d%d%d-%d:%d:%d:%s\n", time_info->tm_mday,
                time_info->tm_mon, time_info->tm_year, time_info->tm_hour,
                time_info->tm_min, time_info->tm_sec, str);
    } else if (type == WARNING) {
        fprintf(log_file, "WARNING::%d%d%d-%d:%d:%d:%s\n", time_info->tm_mday,
                time_info->tm_mon, time_info->tm_year, time_info->tm_hour,
                time_info->tm_min, time_info->tm_sec, str);
    }
}
```

### 1e
Metode encode pada suatu direktori juga berlaku terhadap direktori yang ada di dalamnya.(rekursif)


### Output
Run program 
![alt text](https://i.postimg.cc/NfHXc8Jb/Capture.jpg)
![alt text](https://i.postimg.cc/mkQFQBFJ/2.jpg)

Muncul file sistem baru
![alt text](https://i.postimg.cc/QMH9fpTp/3.jpg)

File asli 

![alt text](https://i.postimg.cc/D0KSfHYg/4.jpg)

File yang terencode

![alt text](https://i.postimg.cc/q7PRQW0T/5.jpg)

hasil log yang di cetak

![alt text](https://i.postimg.cc/PrS50ChK/reload.jpg)


## Soal No 4
Untuk memudahkan dalam memonitor kegiatan pada filesystem mereka Sin dan Sei membuat sebuah log system dengan spesifikasi sebagai berikut.

### 4a 
Log system yang akan terbentuk bernama “SinSeiFS.log” pada direktori home pengguna (/home/[user]/SinSeiFS.log). Log system ini akan menyimpan daftar perintah system call yang telah dijalankan pada filesystem.

### jawab
Untuk soal ini, kita menentukan nama directory.
```
static char *logpath = "/home/wisnupramoedya/SinSeiFS.log";
```
Kita membuat log_file berikut.
```
FILE *log_file = fopen(logpath, "a");
```

### 4b
Karena Sin dan Sei suka kerapian maka log yang dibuat akan dibagi menjadi dua level, yaitu INFO dan WARNING.

### jawab
Kita membuat dua jenis kode log.
```
const int INFO = 1;
const int WARNING = 2;
```

### 4c
Untuk log level WARNING, digunakan untuk mencatat syscall rmdir dan unlink.

### jawab
Pada rmdir, kode yang ditulis.
```
char str[100];
sprintf(str, "RMDIR::%s", path);
log_v2(str, WARNING);

```
Sedangkan, unlink sebagai berikut.
```
char str[100];
sprintf(str, "REMOVE::%s", path);
log_v2(str, WARNING);

```

### 4d
Sisanya, akan dicatat pada level INFO
```
char str[100];
sprintf(str, "MKDIR::%s", path);
log_v2(str, INFO);
```

### 4e
Format untuk logging yaitu:

[Level]::[dd][mm][yyyy]-[HH]:[MM]:[SS]:[CMD]::[DESC :: DESC]

Level : Level logging, dd : 2 digit tanggal, mm : 2 digit bulan, yyyy : 4 digit tahun, HH : 2 digit jam (format 24 Jam),MM : 2 digit menit, SS : 2 digit detik, CMD : System Call yang terpanggil, DESC : informasi dan parameter tambahan

INFO::28052021-10:00:00:CREATE::/test.txt
INFO::28052021-10:01:00:RENAME::/test.txt::/rename.txt

### jawab
```
    time_t current_time;
    time(&current_time);
    struct tm *time_info;
    time_info = localtime(&current_time);

    if (type == INFO) {
        fprintf(log_file, "INFO::%d%d%d-%d:%d:%d:%s\n", time_info->tm_mday,
                time_info->tm_mon, time_info->tm_year, time_info->tm_hour,
                time_info->tm_min, time_info->tm_sec, str);
    } else if (type == WARNING) {
        fprintf(log_file, "WARNING::%d%d%d-%d:%d:%d:%s\n", time_info->tm_mday,
                time_info->tm_mon, time_info->tm_year, time_info->tm_hour,
                time_info->tm_min, time_info->tm_sec, str);
    }
```

## Kendala
- Ketika mencaro dokumentasi FUSE di internet masih sangat sedikit dan susah ditemukan
- Masih bingung debugging
