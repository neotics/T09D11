# T09D15 — Modular Architecture, Static/Dynamic Libraries, Macros, Makefile  
School 21 — C Programming Project

Bu project C tilida modular architecture, static va dynamic library, macro-based I/O, va Makefile orqali build avtomatizatsiyasini o‘rgatadi. Loyihaning barcha funksional qismlari alohida fayllarga bo‘lingan va bosqichma-bosqich kengaytirilgan.

---

# 📁 1. Project Structure
```text
src/
│
├── data_libs/
│ ├── data_io.c
│ ├── data_io.h
│ ├── data_stat.c
│ ├── data_stat.h
│ ├── data_io_macro.h
│
├── data_module/
│ ├── data_process.c
│ ├── data_process.h
│ ├── data_module_entry.c
│
├── yet_another_decision_module/
│ ├── decision.c
│ ├── decision.h
│ ├── yet_another_decision_module_entry.c
│
└── main_executable_module/
├── main_executable_module.c
├── Makefile

```
---

# 🧩 2. Modules

## 2.1 data_io — input/output

### data_io.h
```c
#ifndef DATA_IO_H
#define DATA_IO_H

int read_data(double *data, int n);
int print_data(double *data, int n);

#endif

data_io.c
#include <stdio.h>
#include "data_io.h"

int read_data(double *data, int n) {
    for (int i = 0; i < n; i++)
        if (scanf("%lf", &data[i]) != 1)
            return 1;
    return 0;
}

int print_data(double *data, int n) {
    for (int i = 0; i < n; i++)
        printf("%.2lf ", data[i]);
    return 0;
}
```

data_io.c
```c
#include <stdio.h>
#include "data_io.h"

int read_data(double *data, int n) {
    for (int i = 0; i < n; i++)
        if (scanf("%lf", &data[i]) != 1)
            return 1;
    return 0;
}

int print_data(double *data, int n) {
    for (int i = 0; i < n; i++)
        printf("%.2lf ", data[i]);
    return 0;
}
```

2.2 data_stat — statistika funksiyalari
data_stat.h
```c
#ifndef DATA_STAT_H
#define DATA_STAT_H

double max(double* data, int n);
double min(double* data, int n);
double mean(double* data, int n);
double variance(double* data, int n);

#endif
```

data_stat.c
```c
#include "data_stat.h"

double max(double *data, int n) {
    double m = data[0];
    for (int i = 1; i < n; i++)
        if (data[i] > m) m = data[i];
    return m;
}

double min(double *data, int n) {
    double m = data[0];
    for (int i = 1; i < n; i++)
        if (data[i] < m) m = data[i];
    return m;
}

double mean(double *data, int n) {
    double s = 0;
    for (int i = 0; i < n; i++)
        s += data[i];
    return s / n;
}

double variance(double *data, int n) {
    double m = mean(data,n), s = 0;
    for (int i = 0; i < n; i++)
        s += (data[i] - m) * (data[i] - m);
    return s / n;
}
```

2.3 data_io_macro — parametrik makrolar (Quest 4)
```c
#ifndef DATA_IO_MACRO_H
#define DATA_IO_MACRO_H

#include <stdio.h>

#define DEFINE_INPUT(TYPE) \
void input_##TYPE(TYPE *data, int n) { \
    for (int i = 0; i < n; i++) \
        scanf("%lf", &data[i]); \
}

#define DEFINE_OUTPUT(TYPE, FORMAT) \
void output_##TYPE(TYPE *data, int n) { \
    for (int i = 0; i < n; i++) \
        printf(FORMAT " ", data[i]); \
}

#endif
```

🔧 3. data_process — Normalization
data_process.h
```c
#ifndef PROCESSING_H
#define PROCESSING_H
#define EPS 1E-6
int normalization(double *data, int n);
#endif
```

data_process.c
```c
#include "data_process.h"
#include <math.h>
#include "../data_libs/data_stat.h"

int normalization(double *data, int n) {
    double max_value = max(data, n);
    double min_value = min(data, n);
    double size = max_value - min_value;

    if (fabs(size) < EPS)
        return 1;

    for (int i = 0; i < n; i++)
        data[i] = (data[i] - min_value) / size;

    return 0;
}
```

🔌 4. data_module_entry — kirish/chiqish va normalization
```c
#include <stdio.h>
#include <stdlib.h>
#include "../data_libs/data_stat.h"
#include "../data_module/data_process.h"

#ifndef USE_MACRO_IO
#include "../data_libs/data_io.h"
#else
void input_double(double*, int);
void output_double(double*, int);
#endif

int data_module_entry(int (*norm)(double*, int)) {
    int n;

    if (scanf("%d", &n) != 1 || n <= 0) {
        printf("n/a");
        return 1;
    }

    double *data = malloc(n * sizeof(double));
    if (!data) {
        printf("n/a");
        return 1;
    }

#ifndef USE_MACRO_IO
    if (read_data(data, n) != 0) {
        free(data);
        printf("n/a");
        return 1;
    }
#else
    input_double(data, n);
#endif

    if (norm(data, n) != 0) {
        free(data);
        printf("n/a");
        return 1;
    }

#ifndef USE_MACRO_IO
    print_data(data, n);
#else
    output_double(data, n);
#endif

    free(data);
    return 0;
}
```

🧠 5. Dynamic Library (Quest 6)
main_executable_module.c
```c
#include <stdio.h>
#include <dlfcn.h>

#ifndef USE_MACRO_IO
#include "../data_libs/data_io.h"
#else
#include "../data_libs/data_io_macro.h"
DEFINE_INPUT(double)
DEFINE_OUTPUT(double, "%.2lf")
#endif

typedef int (*norm_func)(double*, int);

int data_module_entry(int (*norm)(double*, int));

int main() {
    void *handle = dlopen("./data_process.so", RTLD_LAZY);
    if (!handle) {
        printf("n/a");
        return 1;
    }

    norm_func normalization = (norm_func)dlsym(handle, "normalization");
    if (!normalization) {
        printf("n/a");
        dlclose(handle);
        return 1;
    }

    int result = data_module_entry(normalization);

    dlclose(handle);
    return result;
}
```

🏗 6. Makefile (Full Working Version)
```makefile
CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c11

all:
	$(CC) $(CFLAGS) main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_module/data_process.c \
	../data_libs/data_io.c \
	../data_libs/data_stat.c \
	-lm -o app

clean:
	rm -f app app_static app_dynamic app_macro data_stat.a data_process.so *.o

rebuild: clean all

### STATIC LIBRARY ###
data_stat.a:
	$(CC) $(CFLAGS) -c ../data_libs/data_stat.c -o data_stat.o
	ar rcs data_stat.a data_stat.o
	rm -f data_stat.o

build_with_static: data_stat.a
	$(CC) $(CFLAGS) main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_module/data_process.c \
	../data_libs/data_io.c \
	data_stat.a -lm -o app_static


### MACRO BUILD ###
build_with_macro:
	$(CC) $(CFLAGS) -DUSE_MACRO_IO main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_module/data_process.c \
	../data_libs/data_stat.c \
	-lm -o app_macro


### DYNAMIC LIBRARY ###
data_process.so:
	$(CC) $(CFLAGS) -fPIC -c ../data_module/data_process.c -o data_process.o
	$(CC) $(CFLAGS) -fPIC -c ../data_libs/data_stat.c -o data_stat_dyn.o
	$(CC) -shared -o data_process.so data_process.o data_stat_dyn.o -lm
	rm -f data_process.o data_stat_dyn.o

build_with_dynamic: data_process.so
	$(CC) $(CFLAGS) main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_libs/data_io.c \
	-lm -ldl -o app_dynamic
```
