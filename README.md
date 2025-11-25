# T09D15 — Modular Architecture, Static/Dynamic Libraries, Macros, Makefile  
School 21 — C Programming Project

Bu project C tilida modular architecture, static va dynamic library, macro-based I/O, va Makefile orqali build avtomatizatsiyasini o‘rgatadi. Loyihaning barcha funksional qismlari alohida fayllarga bo‘lingan va bosqichma-bosqich kengaytirilgan.

---

# 📁 1. Project Structure
```markdown
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

```

data_io.c
```c
#include "data_io.h"

#include <stdio.h>

int read_data(double *data, int n) {
    for (int i = 0; i < n; i++) {
        if (scanf("%lf", &data[i]) != 1) {
            return 1;
        }
    }
    return 0;
}

void print_data(double *data, int n) {
    for (int i = 0; i < n; i++) {
        printf("%.2lf", data[i]);
        if (i < n - 1) {
            printf(" ");
        }
    }
}

```

2.2 data_stat — statistika funksiyalari
data_stat.h
```c
#ifndef DATA_STAT_H
#define DATA_STAT_H

double max(double *data, int n);
double min(double *data, int n);
double mean(double *data, int n);
double variance(double *data, int n);

#endif

```

data_stat.c
```c
#include "data_stat.h"

double max(double *data, int n) {
    double m = data[0];
    for (int i = 1; i < n; i++) {
        if (data[i] > m) m = data[i];
    }
    return m;
}

double min(double *data, int n) {
    double m = data[0];
    for (int i = 1; i < n; i++) {
        if (data[i] < m) m = data[i];
    }
    return m;
}

double mean(double *data, int n) {
    double sum = 0;
    for (int i = 0; i < n; i++) {
        sum += data[i];
    }
    return sum / n;
}

double variance(double *data, int n) {
    double m = mean(data, n);
    double v = 0;
    for (int i = 0; i < n; i++) {
        double diff = data[i] - m;
        v += diff * diff;
    }
    return v / n;
}

```

2.3 data_io_macro — parametrik makrolar (Quest 4)
Bu file repositoryda yo'q. Yaratish:
```bash
touch data_io_macro.h
```
```c
#ifndef DATA_IO_MACRO_H
#define DATA_IO_MACRO_H

#include <stdio.h>

#define DEFINE_INPUT(TYPE)                     \
    void input_##TYPE(TYPE *data, int n) {     \
        for (int i = 0; i < n; i++) {          \
            if (scanf("%lf", &data[i]) != 1) { \
                return;                        \
            }                                  \
        }                                      \
    }

#define DEFINE_OUTPUT(TYPE, FORMAT)         \
    void output_##TYPE(TYPE *data, int n) { \
        for (int i = 0; i < n; i++) {       \
            printf(FORMAT, data[i]);        \
            if (i < n - 1) {                \
                printf(" ");                \
            }                               \
        }                                   \
    }

#endif

```

🔧 3. data_process — Normalization
data_process.h
```c
#ifndef DATA_PROCESS_H
#define DATA_PROCESS_H

#define EPS 1e-6

int normalization(double *data, int n);

#endif

```

data_process.c
```c
#include "data_process.h"

#include <math.h>

#include "../data_libs/data_stat.h"  // max/min shu faylda

int normalization(double *data, int n) {
    double max_value = max(data, n);
    double min_value = min(data, n);
    double size = max_value - min_value;

    if (fabs(size) < EPS) {
        return 1;  // error: normalization impossible
    }

    for (int i = 0; i < n; i++) {
        data[i] = (data[i] - min_value) / size;
    }

    return 0;
}

```

🔌 4. data_module_entry.c — kirish/chiqish va normalization
```c
#include <stdio.h>
#include <stdlib.h>

#include "../data_libs/data_stat.h"
#include "../data_module/data_process.h"

#ifdef USE_MACRO_IO
void input_double(double*, int);
void output_double(double*, int);
#else
#include "../data_libs/data_io.h"
#endif

int data_module_entry(int (*norm)(double*, int)) {
    int n;

    if (scanf("%d", &n) != 1 || n <= 0) {
        printf("n/a");
        return 1;
    }

    double* data = malloc(n * sizeof(double));
    if (!data) {
        printf("n/a");
        return 1;
    }

#ifdef USE_MACRO_IO
    input_double(data, n);
#else
    if (read_data(data, n) != 0) {
        free(data);
        printf("n/a");
        return 1;
    }
#endif

    if (norm(data, n) != 0) {
        free(data);
        printf("n/a");
        return 1;
    }

#ifdef USE_MACRO_IO
    output_double(data, n);
#else
    print_data(data, n);
#endif

    free(data);
    return 0;
}

```

data_module_entry.h
```c
#ifndef DATA_MODULE_ENTRY_H
#define DATA_MODULE_ENTRY_H

int data_module_entry();

#endif

```

⭐ yet_another_decision_module

decision.c

```c
#include "decision.h"

#include <math.h>

#include "../data_libs/data_stat.h"

int make_decision(double *data, int n) {
    double m = mean(data, n);
    double sigma = sqrt(variance(data, n));
    double max_value = max(data, n);

    int ok = 1;

    ok &= (max_value <= m + 3 * sigma);
    ok &= (max_value >= m - 3 * sigma);
    ok &= (m >= GOLDEN_RATIO);

    return ok;
}

```

decision.h

```c
#ifndef DECISION_H
#define DECISION_H

#define GOLDEN_RATIO 0.666

int make_decision(double *data, int n);

#endif
```

yet_another_decision_module_entry.c

```c
#include <stdio.h>
#include <stdlib.h>

#include "../data_libs/data_stat.h"
#include "../data_module/data_process.h"
#include "decision.h"

#ifdef USE_MACRO_IO
void input_double(double*, int);
#else
#include "../data_libs/data_io.h"
#endif

int yet_another_decision_module_entry() {
    int n;

    if (scanf("%d", &n) != 1 || n <= 0) {
        printf("n/a");
        return 1;
    }

    double* data = malloc(n * sizeof(double));
    if (!data) {
        printf("n/a");
        return 1;
    }

#ifdef USE_MACRO_IO
    input_double(data, n);
#else
    if (read_data(data, n) != 0) {
        free(data);
        printf("n/a");
        return 1;
    }
#endif

    if (normalization(data, n) != 0) {
        free(data);
        printf("n/a");
        return 1;
    }

    if (make_decision(data, n))
        printf("YES");
    else
        printf("NO");

    free(data);
    return 0;
}

```


🧠 5. Dynamic Library (Quest 6)
main_executable_module.c
```c
#include <dlfcn.h>
#include <stdio.h>

#ifdef USE_MACRO_IO
#include "../data_libs/data_io_macro.h"
DEFINE_INPUT(double)
DEFINE_OUTPUT(double, "%.2lf")
#else
#include "../data_libs/data_io.h"
#endif

typedef int (*norm_func)(double *, int);

int data_module_entry(int (*norm)(double *, int));

int main() {
    void *handle = dlopen("./data_process.so", RTLD_LAZY);
    if (!handle) {
        printf("n/a");
        return 1;
    }

    norm_func normalization = (norm_func)dlsym(handle, "normalization");
    if (!normalization) {
        dlclose(handle);
        printf("n/a");
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
	../yet_another_decision_module/decision.c \
	../yet_another_decision_module/yet_another_decision_module_entry.c \
	-lm -o app

data_stat.a:
	$(CC) $(CFLAGS) -c ../data_libs/data_stat.c -o data_stat.o
	ar rcs data_stat.a data_stat.o
	rm -f data_stat.o

build_with_static: data_stat.a
	$(CC) $(CFLAGS) main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_module/data_process.c \
	../data_libs/data_io.c \
	../yet_another_decision_module/decision.c \
	../yet_another_decision_module/yet_another_decision_module_entry.c \
	data_stat.a -lm -o app_static

build_with_macro:
	$(CC) $(CFLAGS) -DUSE_MACRO_IO main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_module/data_process.c \
	../data_libs/data_stat.c \
	../yet_another_decision_module/decision.c \
	../yet_another_decision_module/yet_another_decision_module_entry.c \
	-lm -o app_macro

data_process.so:
	$(CC) $(CFLAGS) -fPIC -c ../data_module/data_process.c -o data_process.o
	$(CC) -shared -o data_process.so data_process.o
	rm -f data_process.o

build_with_dynamic: data_process.so
	$(CC) $(CFLAGS) main_executable_module.c \
	../data_module/data_module_entry.c \
	../data_libs/data_io.c \
	../data_libs/data_stat.c \
	-ldl -lm -o app_dynamic


clean:
	rm -f app app_macro app_static app_dynamic data_stat.a data_process.so *.o


```

🧪 7. Testing

Hamma commandlar main_executable_module folder ichida yoziladi:
```bash
cd src/main_executable_module
```

Normal:
```bash
make
echo "3 1 2 3" | ./app
```

Static:
```bash
make build_with_static
echo "3 1 2 3" | ./app_static
```

Macro:
```bash
make build_with_macro
echo "3 1 2 3" | ./app_macro
```

Dynamic:
```bash
make build_with_dynamic
echo "3 1 2 3" | ./app_dynamic
```

🛡 8. Memory Check (Valgrind)
```bash
valgrind --tool=memcheck --leak-check=yes ./app
valgrind --tool=memcheck --leak-check=yes ./app_static
valgrind --tool=memcheck --leak-check=yes ./app_dynamic
```


Kutilgan natija:
```bash
All heap blocks were freed -- no leaks are possible
ERROR SUMMARY: 0 errors
```

🧹 9. Clang-format
Clang-format ni src filega o'tqazish:
```bash
cd materials/linters/.clang-format src/
```

Tekshirish:
```bash
clang-format -n */*.c
clang-format -n */*.h
```

📌 10. Quest Completion Files

Har bitta Quest uchun Quest_X file ochiladi, ichiga hechnarsa yozilmaydi va build folderga git push qilinadi:

```bash
cd build
touch Quest_1
touch Quest_2
touch Quest_3
touch Quest_4
touch Quest_5
touch Quest_6
```
Keyin, GitLabga push qilinadi:

```bash
git add build
git commit -m "All Quest_X added"
git push origin develop
```

Quyidagi ko'rinishida push bo'lishi kerak:

```markdown
build/
    Quest_1
    Quest_2
    Quest_3
    Quest_4
    Quest_5
    Quest_6
```

📌 11. .gitignore file

Asosiy *rootda .gitignore yaratiladi:

*project boshlang'ich joyida

```bash
touch .gitignore
```

Ichiga quyidagilar yoziladi:

```nginx
# Executable files
app
app_macro
app_static
app_dynamic

# Static library
data_stat.a

# Dynamic library
data_process.so

# Object files
*.o

# Backup files
*~
*.swp
*.tmp
*.log
```

Project push qilinishidan oldin GitLabga push qilinadi:

```bash
git add .gitignore
git commit -m ".gitignore added"
git push origin develop
```

Yakuniy Qism

To'liq project GitLabga push qilinadi:

```bash
git add src
git add build
git add .gitignore
git commit -m "Full project completed"
git push origin develop
```
