# Entrega 1: Diseño de Red para Optimización de Latencia con QoS

## 1. Introducción

En el contexto de aplicaciones interactivas como videojuegos en línea, la latencia y la pérdida de paquetes son factores críticos que afectan directamente la experiencia del usuario. Las herramientas de optimización de red (como WTFast) actúan seleccionando rutas de menor retardo y priorizando el tráfico interactivo en redes congestionadas.

Este proyecto propone diseñar un laboratorio académico que simule estas técnicas mediante:
- La construcción de una red con múltiples rutas de diferentes características (latencia y ancho de banda)
- La aplicación de mecanismos de Calidad de Servicio (QoS) para priorizar tráfico sensible a latencia
- La comparación del comportamiento de la red antes y después de optimizaciones

El problema que se quiere resolver es: **¿Cómo reducir la latencia y mejorar la experiencia de aplicaciones interactivas en una red congestionada utilizando selección de rutas inteligente y QoS?**

---

## 2. Objetivos

### Objetivo General

Diseñar e implementar un laboratorio de red que demuestre técnicas de optimización de latencia mediante selección de rutas preferenciales y aplicación de políticas de QoS, reproduciendo el comportamiento de herramientas comerciales de optimización de juegos en línea.

### Objetivos Específicos

#### Objetivo 1: Diseñar una topología de red con múltiples rutas

**¿Qué?** Crear una topología de red que incluya al menos 3 routers intermedios, 2 caminos independientes entre cliente y servidor, y diferentes características de latencia y ancho de banda en cada enlace.

**¿Cómo?** Diseñar una red con dos rutas paralelas donde:
- Ruta A (preferida): mayor ancho de banda y menor retardo
- Ruta B (alternativa): menor ancho de banda y mayor retardo
Utilizando enlaces seriales entre routers y redes LAN en los extremos del cliente y servidor.

**¿Para qué?** Simular escenarios reales donde existen múltiples caminos hacia un destino, permitiendo que el protocolo de enrutamiento seleccione la ruta óptima basado en métricas.

---

#### Objetivo 2: Configurar el direccionamiento IP y protocolo de enrutamiento dinámico

**¿Qué?** Implementar un esquema de direccionamiento IP jerárquico y configurar un protocolo de enrutamiento dinámico (EIGRP) que influya en la selección de rutas.

**¿Cómo?** 
- Asignar direcciones IPv4 con subredes coherentes para las LANs de cliente (192.168.10.0/24), servidor (192.168.40.0/24) y enlaces WAN (subredes /30)
- Configurar EIGRP en todos los routers, manipulando el bandwidth y delay de los enlaces para que EIGRP prefiera la ruta A
- Validar la selección de rutas con comandos de diagnóstico

**¿Para qué?** Garantizar que el tráfico tome la ruta de menor latencia automáticamente mediante decisiones de enrutamiento basadas en métricas de distancia.

---

#### Objetivo 3: Simular congestión de red mediante tráfico diferenciado

**¿Qué?** Generar múltiples tipos de tráfico desde los equipos clientes (tráfico de juego, streaming y descargas) que congestion el enlace de forma realista.

**¿Cómo?**
- Crear tres PCs adicionales en la LAN del cliente, cada una generando un tipo de tráfico específico
- Simular tráfico UDP sensible a latencia (juego)
- Simular tráfico TCP continuo (streaming)
- Simular tráfico TCP pesado (descargas) que congestione la red
- Usar herramientas de simulación de tráfico como iperf3 (en GNS3) o configuraciones en Packet Tracer

**¿Para qué?** Crear un escenario realista donde la red esté congestionada y el tráfico interactivo sufra degradación sin QoS, para luego demostrar la mejora con optimizaciones.

---

#### Objetivo 4: Implementar QoS para priorizar tráfico interactivo

**¿Qué?** Configurar políticas de Quality of Service (QoS) que clasifiquen y prioricen el tráfico de juego sobre otros tipos de tráfico en routers congestionados.

**¿Cómo?**
- Crear class-maps para identificar tráfico interactivo (UDP en puertos específicos o tráfico con características de juego)
- Definir policy-maps con colas priorizadas usando LLQ (Low Latency Queuing) o CBWFQ
- Aplicar las políticas en las interfaces de salida de routers críticos
- Configurar parámetros como bandwidth, priority queue y shaping

**¿Para qué?** Asegurar que el tráfico de juego reciba prioridad de transmisión incluso cuando hay congestión, reduciendo su latencia y jitter comparado con otros tipos de tráfico.

---

#### Objetivo 5: Validar y comparar el comportamiento antes y después de optimizaciones

**¿Qué?** Realizar pruebas de conectividad, medir latencia y analizar el comportamiento de la red en dos escenarios: sin QoS y con QoS.

**¿Cómo?**
- Ejecutar comandos ping y traceroute para verificar conectividad y ruta
- Medir latencia (RTT) del tráfico de juego sin QoS
- Medir latencia del tráfico de juego con QoS activo
- Comparar métricas: latencia, jitter, pérdida de paquetes
- Documentar observaciones en un reporte de comparación

**¿Para qué?** Demostrar de forma cuantificable que las técnicas de optimización (selección de rutas + QoS) efectivamente reducen la latencia y mejoran la experiencia en aplicaciones interactivas.

---

## 3. Actividades

### Fase 1: Planificación y Diseño

| # | Actividad | Responsable | Entregable |
|---|-----------|-------------|-----------|
| 1.1 | Diseñar la topología de red (diagramas) | | Diagrama lógico de la red con todos los equipos |
| 1.2 | Definir el plan de direccionamiento IP | | Tabla de asignación de IPs y subredes |
| 1.3 | Especificar parámetros de latencia y ancho de banda | | Tabla de características de enlaces |

### Fase 2: Implementación en Packet Tracer

| # | Actividad | Responsable | Entregable |
|---|-----------|-------------|-----------|
| 2.1 | Crear la topología de red en Packet Tracer | | Archivo .pkt con topología básica |
| 2.2 | Configurar direccionamiento IP en todos los equipos | | Pruebas ping locales exitosas |
| 2.3 | Implementar EIGRP y configurar métricas | | Tabla de rutas con ruta A seleccionada |
| 2.4 | Configurar delay en los enlaces para simular latencia | | Medidas de latencia verificables con traceroute |

### Fase 3: Simulación de Tráfico

| # | Actividad | Responsable | Entregable |
|---|-----------|-------------|-----------|
| 3.1 | Crear equipos adicionales para diferentes tipos de tráfico | | 3 PCs con roles definidos (gamer, streaming, descargas) |
| 3.2 | Configurar generación de tráfico UDP (juego) | | Tráfico UDP fluyendo desde PC Gamer |
| 3.3 | Configurar generación de tráfico TCP (streaming y descargas) | | Tráfico TCP simultáneo generando congestión |
| 3.4 | Medir latencia sin QoS | | Tabla de mediciones de RTT y jitter |

### Fase 4: Configuración de QoS

| # | Actividad | Responsable | Entregable |
|---|-----------|-------------|-----------|
| 4.1 | Diseñar política de QoS (class-maps y policy-maps) | | Especificación detallada de clases y políticas |
| 4.2 | Configurar class-maps para identificar tráfico de juego | | Configuración de clasificadores en routers |
| 4.3 | Configurar policy-maps con colas priorizadas | | Configuración de políticas (LLQ/CBWFQ) |
| 4.4 | Aplicar políticas en interfaces de salida | | Confirmación de aplicación con show commands |

### Fase 5: Validación y Análisis

| # | Actividad | Responsable | Entregable |
|---|-----------|-------------|-----------|
| 5.1 | Ejecutar pruebas de conectividad (ping, traceroute) | | Resultados de pruebas de conectividad |
| 5.2 | Medir latencia con QoS activo | | Tabla de mediciones con QoS |
| 5.3 | Comparar resultados (antes vs después) | | Gráficos y análisis de mejora |
| 5.4 | Documentar hallazgos y conclusiones | | Reporte técnico con resultados |

### Fase 6: Documentación Final

| # | Actividad | Responsable | Entregable |
|---|-----------|-------------|-----------|
| 6.1 | Documentar configuración de routers | | Archivo con comandos de configuración |
| 6.2 | Crear guía paso a paso de implementación | | Manual técnico reproductible |
| 6.3 | Explicar conceptos teóricos relacionados | | Documento con fundamentos de QoS y enrutamiento |
| 6.4 | Generar reporte final de proyecto | | Documento ejecutivo con conclusiones |

---

## 4. Cronograma Tentativo

| Semana | Fases | Hitos |
|--------|-------|-------|
| 1-2 | Planificación y Diseño (Fase 1) | Topología y plan de direccionamiento aprobados |
| 3-4 | Implementación en Packet Tracer (Fase 2) | Red funcional con EIGRP |
| 5-6 | Simulación de Tráfico (Fase 3) | Tráfico diferenciado y mediciones iniciales |
| 7-8 | Configuración de QoS (Fase 4) | Políticas implementadas y validadas |
| 9-10 | Validación (Fase 5) | Comparativas antes/después completadas |
| 11-12 | Documentación Final (Fase 6) | Entrega final del proyecto |

---

## Notas Importantes

- Este proyecto es un trabajo académico enfocado en aprendizaje de conceptos de redes y QoS
- Se utilizará Cisco Packet Tracer como herramienta principal, con posibilidad de validación en GNS3
- Se espera documentación clara y demostraciones reproducibles
- La comparación de resultados debe ser cuantificable (latencia, jitter, pérdida de paquetes)
