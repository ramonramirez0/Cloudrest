# Cloudrest
# 🌙 Cloudrest: Sistema IoT de Monitorización de Apnea del Sueño

![Status](https://img.shields.io/badge/Status-Prototype-orange)
![Platform](https://img.shields.io/badge/Platform-Arduino%20Nano%2033%20IoT-blue)
![License](https://img.shields.io/badge/License-Open%20Source-green)

> *"Tecnología accesible para un descanso seguro."*
>
> **Proyecto Final: Electrónica Digital y Microcontroladores**
> **Universidad Loyola Andalucía**

---

## 📖 Descripción

**Cloudrest** es un dispositivo wearable de bajo coste (~53€) diseñado para la detección y monitorización domiciliaria de la Apnea del Sueño. A diferencia de los smartwatches convencionales, este sistema es capaz de diferenciar entre **Apnea Obstructiva** y **Apnea Central** mediante la fusión de múltiples parámetros fisiológicos.

El sistema se conecta a **Arduino IoT Cloud** para ofrecer visualización en tiempo real y alertas automáticas ante eventos de hipoxia o cese de la respiración.

---

## 🚀 Características Técnicas

* **🧠 Fusión de Sensores:** Integración simultánea de Oximetría (SpO2), Frecuencia Cardíaca, Audio (Ronquidos) y Esfuerzo Respiratorio.
* **☁️ Arquitectura Cloud Native:** Eliminación del almacenamiento local (SD) en favor de una transmisión segura y en tiempo real a la nube.
* **📉 Algoritmos de Filtrado:**
    * *Max Peak Latch* para detección precisa de ronquidos.
    * Suavizado exponencial para lecturas estables de BPM/SpO2.
    * Lógica de umbrales dinámicos para detección de esfuerzo torácico.
* **🚨 Detección de Eventos:** Algoritmo capaz de identificar ventanas de apnea (>10s) y generar alertas de anomalía.

---

## 🛠️ Hardware (BOM)

| Componente | Función | Precio Aprox. |
| :--- | :--- | :--- |
| **Arduino Nano 33 IoT** | Unidad de control (Cortex-M0+) y conectividad WiFi/BLE. | ~27,98 € |
| **MAX30102** | Sensor de Oximetría de pulso y ritmo cardíaco (I2C). | ~8,00 € |
| **ZD10-100** | Sensor resistivo de película delgada (Expansión torácica). | ~4,00 € |
| **MAX9814** | Micrófono con Control Automático de Ganancia (AGC). | ~7,99 € |
| **Materiales Varios** | Cables, resistencias, banda elástica, carcasa. | ~5,00 € |
| **TOTAL** | **Solución Low-Cost** | **~52,97 €** |

---

## 🧠 Lógica y Algoritmos

El firmware procesa las señales en el borde (*Edge Processing*) antes de enviarlas a la nube:

### 1. Detección de Apnea (Máquina de Estados)
El sistema evalúa continuamente si se cumplen tres condiciones simultáneas durante un intervalo de **10 segundos**:
1.  **Silencio:** Nivel de audio por debajo del `MIC_UMBRAL_RUIDO`.
2.  **Inmovilidad:** Variación de resistencia en el sensor ZD10-100 por debajo del umbral de esfuerzo.
3.  **Desaturación:** Caída del valor de SpO2.

Si el contador `local_contador_apnea` supera el límite médico, se activa la variable `cloud_diagnostico`.

### 2. Procesamiento de Audio
Se utiliza el módulo **MAX9814** con una rutina de lectura rápida. Se implementó un algoritmo de retención de picos para diferenciar entre ruido de fondo y ronquidos fuertes (`> 780` en lectura analógica).

### 3. Esfuerzo Respiratorio
El sensor **ZD10-100** actúa como un divisor de tensión variable. El código calibra dinámicamente la "línea base" de la respiración para detectar la expansión del tórax, permitiendo saber si el paciente intenta respirar (Obstructiva) o no (Central).

---

## 🔌 Esquema de Conexión

| Sensor | Pin Arduino | Tipo de Señal |
| :--- | :--- | :--- |
| **MAX9814 (Micrófono)** | A0 | Analógica |
| **ZD10-100 (Banda)** | A1 | Analógica (Divisor de tensión) |
| **MAX30102 (SpO2)** | A4 (SDA) / A5 (SCL) | Digital I2C |

---

## 🚧 Desafíos Superados (Engineering Logs)

Durante el desarrollo del prototipo nos enfrentamos a varios retos que definieron la arquitectura final:

1.  **Fallo del Sensor Resistivo:** Los contactos del ZD10-100 son frágiles. Se tuvo que implementar una solución mecánica robusta y un filtrado por software para ignorar lecturas erráticas.
2.  **Pivotaje a Cloud:** Inicialmente se planeó usar una tarjeta microSD para datalogging. Tras problemas de integración y corrupción de librerías, se optó por una arquitectura 100% IoT, aprovechando la conectividad del Arduino Nano 33.
3.  **Sincronización:** Se optimizó el envío de variables a Arduino Cloud para evitar el bloqueo del bucle principal (`loop`), permitiendo que el muestreo de sensores no se viera interrumpido por la latencia de la red.

---

## 👨‍💻 Autor

**Proyecto realizado en la Universidad Loyola Andalucía**
* **Asignatura:** Electrónica Digital y Microcontroladores

---
*Este proyecto es de código abierto con fines educativos.*
