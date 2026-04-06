# Parcial 3 - Guia STP/PVST paso a paso

## 1) Objetivo
Configurar STP por VLAN para cumplir estas condiciones:

- VLAN 10 y 20:
  - El switch de Core (SW1) debe ser Root Bridge.
  - Se debe forzar la topologia STP para que los roles de puertos queden como en el enunciado.
- VLAN 30 y 40:
  - El Root Bridge debe ser un switch de distribucion (se usara SW2).
  - Debe haber un respaldo (SW3) para mantener estabilidad ante reemplazos.

Adicionalmente, la configuracion se hace con prioridades explicitas para reducir dependencia del MAC address y preservar comportamiento si se reemplazan switches.

## 2) Topologia y enlaces usados

Se asume la misma topologia del punto 1-2:

- SW1 Fa0/1 <-> SW2 Fa0/1
- SW1 Fa0/2 <-> SW3 Fa0/1
- SW2 Fa0/2 <-> SW3 Fa0/2
- SW2 Fa0/3 <-> SW4 Fa0/1
- SW3 Fa0/3 <-> SW5 Fa0/1
- SW3 Fa0/4 <-> SW6 Fa0/1
- SW4 Fa0/2 <-> SW5 Fa0/2
- SW5 Fa0/3 <-> SW6 Fa0/2

## 3) Rol STP esperado para VLAN 10 y 20

- SW1: Fa0/1 Designado, Fa0/2 Designado
- SW2: Fa0/1 Raiz, Fa0/2 Designado, Fa0/3 Designado
- SW3: Fa0/1 Raiz, Fa0/2 Bloqueado, Fa0/3 Designado, Fa0/4 Designado
- SW4: Fa0/1 Raiz, Fa0/2 Designado
- SW5: Fa0/1 Raiz, Fa0/2 Bloqueado, Fa0/3 Bloqueado
- SW6: Fa0/1 Raiz, Fa0/2 Designado

## 4) Recomendacion de modo STP

En todos los switches:

```bash
enable
configure terminal
spanning-tree mode rapid-pvst
end
copy running-config startup-config
```

Si tu Packet Tracer no soporta rapid-pvst en ese modelo, usa el modo por defecto PVST y continua con las prioridades.

## 5) Configuracion de prioridades por switch

### SW1 (Core)

Objetivo:
- Root Bridge para VLAN 10 y 20.
- No ser root en VLAN 30 y 40.

```bash
enable
configure terminal
spanning-tree vlan 10,20 priority 0
spanning-tree vlan 30,40 priority 24576
end
copy running-config startup-config
```

### SW2 (Distribucion)

Objetivo:
- Secundario para VLAN 10 y 20.
- Root principal para VLAN 30 y 40.
- Ganar Designated sobre SW3 en el enlace SW2-SW3 para VLAN 10 y 20.

```bash
enable
configure terminal
spanning-tree vlan 10,20 priority 8192
spanning-tree vlan 30,40 priority 4096
end
copy running-config startup-config
```

### SW3 (Distribucion)

Objetivo:
- Debe quedar por debajo de SW2 en VLAN 10 y 20 para que Fa0/2 quede bloqueado en ese enlace.
- Secundario para VLAN 30 y 40.

```bash
enable
configure terminal
spanning-tree vlan 10,20 priority 12288
spanning-tree vlan 30,40 priority 8192
end
copy running-config startup-config
```

### SW4 (Acceso)

Objetivo:
- En VLAN 10 y 20, tener mejor BID que SW5 para que en SW4-SW5 el Designated sea SW4 y SW5 quede bloqueado.

```bash
enable
configure terminal
spanning-tree vlan 10,20 priority 16384
end
copy running-config startup-config
```

### SW5 (Acceso)

Objetivo:
- En VLAN 10 y 20, quedar con peor BID que SW4 y SW6 para que Fa0/2 y Fa0/3 queden bloqueados.

```bash
enable
configure terminal
spanning-tree vlan 10,20 priority 28672
end
copy running-config startup-config
```

### SW6 (Acceso)

Objetivo:
- En VLAN 10 y 20, tener mejor BID que SW5 para que en SW6-SW5 el Designated sea SW6.

```bash
enable
configure terminal
spanning-tree vlan 10,20 priority 20480
end
copy running-config startup-config
```

## 6) Comandos de verificacion

En cada switch valida:

```bash
show spanning-tree vlan 10
show spanning-tree vlan 20
show spanning-tree vlan 30
show spanning-tree vlan 40
show spanning-tree root
```

## 7) Checklist de cumplimiento

### VLAN 10 y 20

- Root Bridge: SW1.
- Roles por puerto:
  - SW1: Fa0/1 D, Fa0/2 D
  - SW2: Fa0/1 R, Fa0/2 D, Fa0/3 D
  - SW3: Fa0/1 R, Fa0/2 B, Fa0/3 D, Fa0/4 D
  - SW4: Fa0/1 R, Fa0/2 D
  - SW5: Fa0/1 R, Fa0/2 B, Fa0/3 B
  - SW6: Fa0/1 R, Fa0/2 D

### VLAN 30 y 40

- Root Bridge: SW2.
- Root secundario: SW3.
- SW1 no debe ser Root Bridge en estas VLAN.

## 8) Prueba de resiliencia (reemplazo de switches)

Para validar que se preserva el comportamiento esperado:

1. Simula reemplazo de un switch de acceso (por ejemplo SW5) en Packet Tracer.
2. Reaplica el nombre del switch y los comandos de prioridad STP del punto 5.
3. Ejecuta de nuevo verificaciones del punto 6.
4. Confirma que para VLAN 10 y 20 se recuperan los puertos bloqueados esperados y que en VLAN 30 y 40 el Root sigue en distribucion.

## 9) Notas importantes

- Las prioridades de STP deben ser multiplos de 4096.
- Si omites prioridades explicitas, STP puede elegir root por MAC y cambiar el resultado al reemplazar equipos.
- Si observas un puerto distinto al esperado, revisa primero:
  - Troncales activos y VLAN permitidas.
  - Coincidencia de Native VLAN en ambos extremos.
  - Que no existan costos STP manuales heredados en interfaces.
