# Proyectos STM32F4 - Grupo 6 TDII 2025

## 📋 Descripción General

Este repositorio contiene 4 aplicaciones desarrolladas para la placa STM32F413ZH Discovery, implementando diferentes funcionalidades utilizando drivers personalizados y la biblioteca HAL de STMicroelectronics.

## 🏗️ Estructura del Proyecto

```
AFP_2_GRUPO_6_TDII_2025/
├── App_1_1_GRUPO_6_2025/     # Aplicación 1: Control de LEDs
├── App_1_2_GRUPO_6_2025/     # Aplicación 2: [Descripción pendiente]
├── App_1_3_GRUPO_6_2025/     # Aplicación 3: [Descripción pendiente]
└── App_1_4_GRUPO_6_2025/     # Aplicación 4: [Descripción pendiente]
```

## 🔧 ¿Por qué se crean Drivers?

Los drivers se crean por las siguientes razones fundamentales:

### 1. **Abstracción de Hardware**
- **Problema**: El hardware específico del microcontrolador es complejo y requiere configuración detallada
- **Solución**: Los drivers encapsulan esta complejidad en funciones simples y fáciles de usar
- **Ejemplo**: En lugar de configurar registros directamente, usamos `writeLedOn_GPIO(LD1_Pin)`

### 2. **Portabilidad**
- **Problema**: El código específico del hardware no es reutilizable en otros microcontroladores
- **Solución**: Los drivers crean una capa de abstracción que permite migrar fácilmente entre plataformas
- **Beneficio**: El código de aplicación principal no cambia al cambiar de hardware

### 3. **Mantenibilidad**
- **Problema**: Los cambios en el hardware requieren modificar todo el código de aplicación
- **Solución**: Los cambios se centralizan en los drivers
- **Beneficio**: Modificaciones localizadas y código más limpio

### 4. **Reutilización**
- **Problema**: Funcionalidades comunes se repiten en múltiples proyectos
- **Solución**: Los drivers se pueden reutilizar en diferentes aplicaciones
- **Ejemplo**: `API_GPIO.c` se usa en todas las aplicaciones del proyecto

## 📁 Estructura de Archivos .h y .c

### **Archivo Header (.h)**
```c
// API_GPIO.h
typedef uint16_t led_t;
typedef bool buttonStatus_t;

void writeLedOn_GPIO(led_t LDx);
void writeLedOff_GPIO(led_t LDx);
void toggleLed_GPIO(led_t LDx);
buttonStatus_t readButton_GPIO(void);
```

**Propósito del .h:**
- **Declaraciones**: Define qué funciones están disponibles
- **Tipos de datos**: Define estructuras y tipos personalizados
- **Interfaz pública**: Especifica cómo usar el driver
- **Inclusión**: Permite que otros archivos usen las funciones

### **Archivo Implementación (.c)**
```c
// API_GPIO.c
void writeLedOn_GPIO(led_t LDx){
    HAL_GPIO_WritePin(GPIOB, LDx, GPIO_PIN_SET);
}

void writeLedOff_GPIO(led_t LDx){
    HAL_GPIO_WritePin(GPIOB, LDx, GPIO_PIN_RESET);
}
```

**Propósito del .c:**
- **Implementación**: Contiene el código real de las funciones
- **Lógica**: Define cómo se ejecutan las operaciones
- **Detalles**: Maneja la complejidad del hardware
- **Privacidad**: Los detalles internos no son visibles desde fuera

## 🎯 Funciones HAL (Hardware Abstraction Layer)

### **¿Qué es HAL?**
HAL es la biblioteca de abstracción de hardware de STMicroelectronics que proporciona una interfaz unificada para acceder a los periféricos del microcontrolador.

### **Funciones HAL Principales:**

#### **GPIO (Entrada/Salida de Propósito General)**
```c
// Configurar pin como salida
HAL_GPIO_WritePin(GPIOB, LD1_Pin, GPIO_PIN_SET);    // Encender LED
HAL_GPIO_WritePin(GPIOB, LD1_Pin, GPIO_PIN_RESET);  // Apagar LED
HAL_GPIO_TogglePin(GPIOB, LD1_Pin);                 // Alternar LED
HAL_GPIO_ReadPin(GPIOC, USER_Btn_Pin);              // Leer botón
```

#### **Sistema y Reloj**
```c
HAL_Init();                    // Inicializar HAL
HAL_Delay(200);               // Delay de 200ms
SystemClock_Config();         // Configurar reloj del sistema
```

#### **UART (Comunicación Serial)**
```c
HAL_UART_Init(&huart3);       // Inicializar UART
HAL_UART_Transmit();          // Transmitir datos
HAL_UART_Receive();           // Recibir datos
```

#### **USB**
```c
HAL_PCD_Init(&hpcd_USB_OTG_FS);  // Inicializar USB
```

## 🔄 Flujo del Código - ¿Qué archivo llama a qué?

### **Secuencia de Ejecución:**

```
1. startup_stm32f413zhtx.s
   ↓ (Punto de entrada)
2. main.c
   ↓ (Inicialización)
3. HAL_Init()
   ↓
4. SystemClock_Config() [en API_GPIO.c]
   ↓
5. MX_GPIO_Init() [en API_GPIO.c]
   ↓
6. MX_USART3_UART_Init() [en API_GPIO.c]
   ↓
7. MX_USB_OTG_FS_PCD_Init() [en API_GPIO.c]
   ↓
8. while(1) [Bucle principal en main.c]
   ↓
9. writeLedOff_GPIO() [llama a HAL_GPIO_WritePin()]
   ↓
10. toggleLed_GPIO() [llama a HAL_GPIO_TogglePin()]
    ↓
11. HAL_Delay() [función HAL]
```

### **Jerarquía de Inclusión:**
```
main.c
├── #include "main.h"
│   ├── #include "stm32f4xx_hal.h"
│   │   ├── stm32f4xx_hal_gpio.h
│   │   ├── stm32f4xx_hal_uart.h
│   │   └── stm32f4xx_hal_pcd.h
│   └── [definiciones de pines]
└── #include "API_GPIO.h"
    └── [declaraciones de funciones del driver]
```

## 🎮 Aplicación 1: Control de LEDs

### **Funcionalidad:**
- Parpadeo secuencial de 3 LEDs (LD1, LD2, LD3)
- Cada LED parpadea durante 200ms
- Secuencia: LD1 → LD2 → LD3 → Repetir

### **Código Principal:**
```c
while (1) {
    writeLedOff_GPIO(LED[2]);  // Apagar último LED
    for (int i = 0; i < 3; i++) {
        toggleLed_GPIO(LED[i]);  // Encender LED
        HAL_Delay(200);
        toggleLed_GPIO(LED[i]);  // Apagar LED
        HAL_Delay(200);
    }
}
```

### **Pines Utilizados:**
- **LD1_Pin**: GPIO_PIN_0 en GPIOB
- **LD2_Pin**: GPIO_PIN_7 en GPIOB  
- **LD3_Pin**: GPIO_PIN_14 en GPIOB
- **USER_Btn_Pin**: GPIO_PIN_13 en GPIOC

## 🛠️ Configuración del Proyecto

### **Requisitos:**
- STM32CubeIDE o STM32CubeMX
- Placa STM32F413ZH Discovery
- Compilador ARM GCC

### **Configuración de Pines:**
- **LEDs**: Configurados como salidas push-pull
- **Botón**: Configurado como entrada con pull-up
- **UART3**: 115200 baudios, 8 bits, sin paridad
- **USB**: Full-speed, 6 endpoints

### **Reloj del Sistema:**
- **Frecuencia**: 96 MHz
- **Fuente**: HSE externo (8 MHz)
- **PLL**: Configurado para multiplicar por 48

## 📚 Conceptos Clave

### **1. Capas de Abstracción:**
```
Aplicación (main.c)
    ↓
Driver Personalizado (API_GPIO.c)
    ↓
HAL (stm32f4xx_hal_*.c)
    ↓
Registros del Microcontrolador
```

### **2. Separación de Responsabilidades:**
- **main.c**: Lógica de aplicación
- **API_GPIO.c**: Funciones de alto nivel para GPIO
- **HAL**: Funciones de bajo nivel para hardware
- **main.h**: Definiciones y constantes

### **3. Modularidad:**
- Cada funcionalidad en su propio archivo
- Interfaces claras entre módulos
- Fácil mantenimiento y extensión

## 🚀 Compilación y Debug

### **Build:**
```bash
# En STM32CubeIDE
Project → Build All
# O en línea de comandos
make -f makefile
```

### **Debug:**
- Conectar placa via USB
- Configurar debugger en STM32CubeIDE
- Ejecutar en modo debug

## 📝 Notas Importantes

1. **Inicialización**: Siempre llamar `HAL_Init()` antes de usar cualquier función HAL
2. **Configuración de Reloj**: Esencial para el funcionamiento correcto
3. **Inicialización de Periféricos**: Debe hacerse antes del bucle principal
4. **Manejo de Errores**: Usar `Error_Handler()` para errores críticos

## 🔗 Enlaces Útiles

- [Documentación STM32F4](https://www.st.com/en/microcontrollers-microprocessors/stm32f4-series.html)
- [Manual HAL](https://www.st.com/resource/en/user_manual/dm00105879.pdf)
- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)

---

**Desarrollado por Grupo 6 - Técnicas Digitales II 2025**
