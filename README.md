# T09D15 — Modular Architecture, Static/Dynamic Libraries, Macros, Makefile  
School 21 — C Programming Project

Bu project C tilida modular architecture, static va dynamic library, macro-based I/O, va Makefile orqali build avtomatizatsiyasini o‘rgatadi. Loyihaning barcha funksional qismlari alohida fayllarga bo‘lingan va bosqichma-bosqich kengaytirilgan.

---

# 📁 1. Project Structure
```c
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

