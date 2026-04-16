# 🧠 PROYECTO PROFESIONAL: Integración Python + C + ARM64 (Termux)

Este proyecto demuestra la integración de lenguajes de alto y bajo nivel para desarrollar librerías de alto rendimiento en arquitectura **ARM64 (AArch64)**. Se utiliza Python como interfaz de usuario mediante `ctypes`, conectándose a un puente en C que ejecuta rutinas optimizadas en Assembly.

## 🎯 Objetivo
Desarrollar una librería de alto rendimiento accesible desde Python, incluyendo depuración avanzada con GDB y análisis comparativo de rendimiento (Profiling).

---

## ⚙️ 1. Instalación desde cero (Termux)

Para preparar el entorno de desarrollo en un dispositivo Android con Termux, ejecuta los siguientes comandos:

```bash
pkg update && pkg upgrade -y
pkg install clang make python git gdb asciinema -y
```

**Verificación de herramientas:**
```bash
python --version
clang --version
make --version
```

---

## 📁 2. Estructura del Proyecto

La organización de archivos sigue los estándares de desarrollo profesional:

```text
Proyecto_ARM64_Python/
│
├── src/               # Código fuente (App, Bridge, Assembly)
│   ├── app.py
│   ├── bridge.c
│   ├── ops.s
│
├── build/             # Binarios y objetos compilados
│
├── Makefile           # Automatización de compilación
├── README.md          # Documentación principal
├── reporte.pdf        # Documentación técnica extendida
└── evidencias/        # Capturas y archivos de performance
```

**Comandos para crear la estructura:**
```bash
mkdir -p Proyecto_ARM64_Python/src Proyecto_ARM64_Python/build
cd Proyecto_ARM64_Python
touch src/app.py src/bridge.c src/ops.s Makefile README.md
```

---

## 🛠️ 3. Implementación del Código

### 📑 ARM64 Assembly (`src/ops.s`)
Implementación de operaciones aritméticas básicas directamente en registros AArch64.

```asm
// Autor: Noyola Rivera Carlos Ernesto
// Arquitectura: ARM64 / AArch64

.global suma
.global resta
.global multiplicacion
.global maximo
.global minimo

suma:
    add x0, x0, x1
    ret

resta:
    sub x0, x0, x1
    ret

multiplicacion:
    mul x0, x0, x1
    ret

maximo:
    cmp x0, x1
    csel x0, x0, x1, gt
    ret

minimo:
    cmp x0, x1
    csel x0, x0, x1, lt
    ret
```

### 📑 Puente en C (`src/bridge.c`)
Actúa como interfaz entre el código de bajo nivel y la librería dinámica compatible con Python.

```c
/*
Autor: Noyola Rivera Carlos Ernesto
Descripción: Puente C hacia ASM
*/

long suma(long, long);
long resta(long, long);
long multiplicacion(long, long);
long maximo(long, long);
long minimo(long, long);

long c_suma(long a, long b){ return suma(a, b); }
long c_resta(long a, long b){ return resta(a, b); }
long c_mul(long a, long b){ return multiplicacion(a, b); }
long c_max(long a, long b){ return maximo(a, b); }
long c_min(long a, long b){ return minimo(a, b); }
```

### 📑 Makefile Profesional
Automatiza el proceso de ensamblado, compilación y generación de la librería compartida `.so`.

```makefile
CC = clang
CFLAGS = -fPIC -g
LDFLAGS = -shared

all:
	$(CC) $(CFLAGS) -c src/ops.s -o build/ops.o
	$(CC) $(CFLAGS) -c src/bridge.c -o build/bridge.o
	$(CC) $(LDFLAGS) build/ops.o build/bridge.o -o build/libops.so

clean:
	rm -f build/*.o build/*.so
```

---

## 🐍 4. Interfaz Python (`src/app.py`)

Este script carga la librería compilada y realiza una prueba de rendimiento comparando la implementación nativa de Python contra la rutina en Assembly.

```python
import ctypes
import time
import os

# Carga de la librería dinámica
lib_path = os.path.abspath('../build/libops.so')
lib = ctypes.CDLL(lib_path)

# Definición de tipos para c_suma
lib.c_suma.argtypes = [ctypes.c_long, ctypes.c_long]
lib.c_suma.restype = ctypes.c_long

print(f"--- Resultado de Prueba ---")
print("Suma (ASM):", lib.c_suma(10, 5))

N = 1000000

def py_suma(a, b):
    return a + b

# Benchmarking Python
t_start = time.time()
for _ in range(N):
    py_suma(10, 5)
t_end = time.time()
print(f"Tiempo Python: {t_end - t_start:.4f}s")

# Benchmarking Assembly (via C Bridge)
t_start = time.time()
for _ in range(N):
    lib.c_suma(10, 5)
t_end = time.time()
print(f"Tiempo ASM:    {t_end - t_start:.4f}s")
```

---

## 🚀 5. Flujo de Trabajo

### Compilación
```bash
make
```
*Resultado esperado: Generación de `build/libops.so`.*

### Ejecución
```bash
cd src
python app.py
```

### Depuración con GDB
Para realizar un análisis paso a paso de los registros de la CPU:
```bash
gdb python
```
Dentro de GDB:
```gdb
break suma
run src/app.py
info registers
stepi
```

---

## 📊 6. Análisis Técnico y Resultados

### Comparativa de Rendimiento
| Método | Tiempo (1M iteraciones) | Ventaja |
| :--- | :--- | :--- |
| **Python** | X.XXX s | Facilidad de uso |
| **Assembly** | Y.YYY s | Control de hardware |

**Conclusiones:**
1. **Velocidad:** Assembly es significativamente más rápido al eliminar el "overhead" del intérprete de Python.
2. **Eficiencia:** Control directo de registros y memoria.
3. **Costo de Llamada:** Python introduce una pequeña latencia al llamar funciones externas vía `ctypes`.

---

## 🚨 7. Resolución de Problemas (Troubleshooting)

* **`undefined symbol`**: Asegúrate de que los nombres en el Makefile y el código C coincidan exactamente. Ejecuta `make clean && make`.
* **`segmentation fault`**: Verifica que los tipos de datos en `ctypes` (`c_long`) coincidan con los de C/Assembly.
* **Error de Arquitectura**: Este código está diseñado específicamente para **ARM64**. No funcionará en emuladores x86 sin traducción.

---

## 📈 8. Retos Avanzados y Escalamiento
* **Optimización SIMD:** Implementación de instrucciones **NEON** para procesamiento paralelo de datos.
* **Sistemas Embebidos:** Portabilidad hacia Raspberry Pi o NVIDIA Jetson.
* **Integración Continua:** Uso de CMake para proyectos de mayor escala.

---

## 🎥 Evidencias de Ejecución
Para grabar una sesión de terminal profesional:
```bash
https://asciinema.org/a/jcTMixVOmP7CFojB
```

**Autor:** Noyola Rivera Carlos Ernesto  
**Institución:** Instituto Tecnológico de Tijuana  
**Carrera:** Ingeniería en Sistemas Computacionales
