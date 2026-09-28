# Solución — Examen Práctico (Práctica)

Esta guía resuelve el archivo `Examen_Practico--Practica.pkt` paso a paso. Primero va el subneteo, después la tabla de direccionamiento y por último los comandos de cada equipo, en el orden en que conviene configurarlos.

---

## 1. Lo que pide el enunciado

1. Con la red **10.0.0.0/8**, hacer un subneteo de **300 hosts para la LAN 1**.
2. Direccionar correctamente cada **punto a punto** (PP1, PP2, PP3 y PP4).
3. Usar **rutas estáticas**.
4. Que desde **Router0** se llegue a **VLAN 1 y VLAN 2** con una **ruta sumarizada**.
5. Hacer una **ruta flotante** del **Switch Capa 3 hacia Router2**.
6. Poner una **ruta por defecto** de **Router1 hacia el Switch Capa 3**.

## 2. Cómo está conectada la topología

| Enlace | Extremo A | Extremo B |
|---|---|---|
| PP1 | Router1 G0/0 | Switch Capa 3 F0/4 |
| PP2 (ruta principal) | Switch Capa 3 F0/1 | Router2 G0/0 |
| PP3 (ruta flotante) | Switch Capa 3 F0/2 | Router2 G0/1 |
| PP4 | Router0 G0/0 | Switch Capa 3 F0/3 |
| LAN 1 | Router0 G0/1 | Switch1 F0/1 (PC2 en F0/2, PC3 en F0/3) |
| VLAN 1 y VLAN 2 | Router1 G0/1 | Switch0 F0/1 (PC0 F0/2, PC1 F0/3, PC4 F0/4, PC5 F0/5) |

Las VLAN cuelgan de un solo cable entre Router1 y Switch0, así que Router1 se configura como **router-on-a-stick** (subinterfaces) y el puerto F0/1 de Switch0 va como **trunk**.

---

## 3. Subneteo

### LAN 1 con 300 hosts

Se busca la menor cantidad de bits de host que alcance para 300 equipos.

- 2⁸ − 2 = 254 → no alcanza
- 2⁹ − 2 = **510** → sí alcanza

Entonces se ocupan **9 bits de host**, o sea 32 − 9 = **/23**, que es la máscara **255.255.254.0**.

| Dato | Valor |
|---|---|
| Red | 10.0.0.0/23 |
| Máscara | 255.255.254.0 |
| Primer host | 10.0.0.1 (gateway, Router0 G0/1) |
| Último host | 10.0.1.254 |
| Broadcast | 10.0.1.255 |
| Hosts útiles | 510 |

### Punto a punto con /30

Cada enlace punto a punto solo necesita 2 IPs, entonces se usa **/30 (255.255.255.252)**, que da exactamente 2 hosts útiles. Se toman justo después de la LAN 1, a partir de 10.0.2.0.

| Enlace | Red | IP extremo A | IP extremo B | Broadcast |
|---|---|---|---|---|
| PP1 | 10.0.2.0/30 | Router1 G0/0 → 10.0.2.1 | SW Capa 3 F0/4 → 10.0.2.2 | 10.0.2.3 |
| PP2 | 10.0.2.4/30 | SW Capa 3 F0/1 → 10.0.2.5 | Router2 G0/0 → 10.0.2.6 | 10.0.2.7 |
| PP3 | 10.0.2.8/30 | SW Capa 3 F0/2 → 10.0.2.9 | Router2 G0/1 → 10.0.2.10 | 10.0.2.11 |
| PP4 | 10.0.2.12/30 | Router0 G0/0 → 10.0.2.13 | SW Capa 3 F0/3 → 10.0.2.14 | 10.0.2.15 |

### Sumarización de VLAN 1 y VLAN 2

Se pasan a binario los terceros octetos, que es donde cambian las redes.

```
192.168.25.0  →  192.168.000110 01.0
192.168.26.0  →  192.168.000110 10.0
                         ^^^^^^ 6 bits en común
```

Los primeros 16 bits (192.168) más 6 bits en común dan **/22**. Con los bits sobrantes en cero queda la red **192.168.24.0/22**, máscara **255.255.252.0**.

Esa ruta cubre de 192.168.24.0 a 192.168.27.255, así que incluye las dos VLAN en una sola línea.

---

## 4. Tabla de direccionamiento completa

| Equipo | Interfaz | IP | Máscara | Gateway |
|---|---|---|---|---|
| Router0 | G0/0 | 10.0.2.13 | 255.255.255.252 | — |
| Router0 | G0/1 | 10.0.0.1 | 255.255.254.0 | — |
| Router1 | G0/0 | 10.0.2.1 | 255.255.255.252 | — |
| Router1 | G0/1.1 (VLAN 1, nativa) | 192.168.25.1 | 255.255.255.0 | — |
| Router1 | G0/1.2 (VLAN 2) | 192.168.26.1 | 255.255.255.0 | — |
| Router2 | G0/0 | 10.0.2.6 | 255.255.255.252 | — |
| Router2 | G0/1 | 10.0.2.10 | 255.255.255.252 | — |
| SW Capa 3 | F0/1 | 10.0.2.5 | 255.255.255.252 | — |
| SW Capa 3 | F0/2 | 10.0.2.9 | 255.255.255.252 | — |
| SW Capa 3 | F0/3 | 10.0.2.14 | 255.255.255.252 | — |
| SW Capa 3 | F0/4 | 10.0.2.2 | 255.255.255.252 | — |
| PC0 | Fa0 | 192.168.25.10 | 255.255.255.0 | 192.168.25.1 |
| PC1 | Fa0 | 192.168.25.11 | 255.255.255.0 | 192.168.25.1 |
| PC4 | Fa0 | 192.168.26.10 | 255.255.255.0 | 192.168.26.1 |
| PC5 | Fa0 | 192.168.26.11 | 255.255.255.0 | 192.168.26.1 |
| PC2 | Fa0 | 10.0.0.10 | 255.255.254.0 | 10.0.0.1 |
| PC3 | Fa0 | 10.0.0.11 | 255.255.254.0 | 10.0.0.1 |

> Ojo con PC2 y PC3. La máscara es **255.255.254.0**, no 255.255.255.0. Es el error más común en este ejercicio.

---

## 5. Configuración de cada equipo

Los bloques se pueden copiar y pegar en la pestaña **CLI** de cada equipo. Si el router pregunta por el diálogo de configuración inicial, respondé `no`.

### 5.1 Switch0 (VLAN 1 y VLAN 2)

PC0 y PC1 se quedan en VLAN 1 (viene así por defecto). PC4 y PC5 pasan a VLAN 2. El puerto hacia Router1 va en trunk.

```
enable
configure terminal
hostname SW0
vlan 2
 name VLAN2
 exit
interface fastEthernet 0/1
 switchport mode trunk
 exit
interface range fastEthernet 0/4 - 5
 switchport mode access
 switchport access vlan 2
 exit
end
copy running-config startup-config
```

### 5.2 Switch1 (LAN 1)

No necesita configuración. Todos los puertos están en VLAN 1 por defecto y eso basta. Si querés, solo cambiale el nombre.

```
enable
configure terminal
hostname SW1
end
copy running-config startup-config
```

### 5.3 Router1 (router-on-a-stick y ruta por defecto)

```
enable
configure terminal
hostname R1

interface gigabitEthernet 0/0
 description PP1 hacia Switch Capa 3
 ip address 10.0.2.1 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet 0/1
 no shutdown
 exit

interface gigabitEthernet 0/1.1
 description VLAN 1
 encapsulation dot1Q 1 native
 ip address 192.168.25.1 255.255.255.0
 exit

interface gigabitEthernet 0/1.2
 description VLAN 2
 encapsulation dot1Q 2
 ip address 192.168.26.1 255.255.255.0
 exit

! Ruta por defecto hacia el Switch Capa 3 (punto 6 del enunciado)
ip route 0.0.0.0 0.0.0.0 10.0.2.2

end
copy running-config startup-config
```

Router1 ya conoce sus VLAN y el PP1 porque están conectadas directamente. Todo lo demás lo manda al Switch Capa 3 con la ruta por defecto.

### 5.4 Router0 (LAN 1 y ruta sumarizada)

```
enable
configure terminal
hostname R0

interface gigabitEthernet 0/0
 description PP4 hacia Switch Capa 3
 ip address 10.0.2.13 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet 0/1
 description LAN 1 (300 hosts)
 ip address 10.0.0.1 255.255.254.0
 no shutdown
 exit

! Ruta sumarizada hacia VLAN 1 y VLAN 2 (punto 4 del enunciado)
ip route 192.168.24.0 255.255.252.0 10.0.2.14

! Rutas hacia los demás punto a punto
ip route 10.0.2.0 255.255.255.252 10.0.2.14
ip route 10.0.2.4 255.255.255.252 10.0.2.14
ip route 10.0.2.8 255.255.255.252 10.0.2.14

end
copy running-config startup-config
```

### 5.5 Switch Capa 3 (Multilayer Switch0, 3560)

Acá hay dos detalles clave.

- Hay que activar **`ip routing`**, porque sin eso el switch no enruta aunque tenga IPs.
- Los puertos que van a routers se vuelven puertos enrutados con **`no switchport`**, así se les puede poner IP directamente.

```
enable
configure terminal
hostname SW-L3
ip routing

interface fastEthernet 0/1
 description PP2 Ruta principal hacia Router2
 no switchport
 ip address 10.0.2.5 255.255.255.252
 no shutdown
 exit

interface fastEthernet 0/2
 description PP3 Ruta flotante hacia Router2
 no switchport
 ip address 10.0.2.9 255.255.255.252
 no shutdown
 exit

interface fastEthernet 0/3
 description PP4 hacia Router0
 no switchport
 ip address 10.0.2.14 255.255.255.252
 no shutdown
 exit

interface fastEthernet 0/4
 description PP1 hacia Router1
 no switchport
 ip address 10.0.2.2 255.255.255.252
 no shutdown
 exit

! Hacia la LAN 1 (por Router0)
ip route 10.0.0.0 255.255.254.0 10.0.2.13

! Hacia VLAN 1 y VLAN 2 (por Router1)
ip route 192.168.25.0 255.255.255.0 10.0.2.1
ip route 192.168.26.0 255.255.255.0 10.0.2.1

! Hacia Router2, ruta principal por PP2 (distancia administrativa 1)
ip route 0.0.0.0 0.0.0.0 10.0.2.6
! Hacia Router2, ruta flotante por PP3 (distancia administrativa 5)
ip route 0.0.0.0 0.0.0.0 10.0.2.10 5

end
copy running-config startup-config
```

**¿Por qué funciona la ruta flotante?** Las dos rutas van al mismo destino, pero la principal tiene distancia administrativa 1 (el valor por defecto) y la flotante tiene 5. El switch siempre prefiere el número más bajo, así que la flotante queda "escondida" y solo entra a la tabla de enrutamiento si se cae el enlace PP2. Cualquier número mayor que 1 sirve; se usa 5 como ejemplo.

### 5.6 Router2 (rutas de regreso)

Router2 también necesita saber cómo volver a las demás redes, si no los pings llegan pero la respuesta se pierde. Se le pone lo mismo, una ruta principal por PP2 y una flotante por PP3.

La red **10.0.0.0/22** agrupa la LAN 1 (10.0.0.0/23) y todos los punto a punto (10.0.2.0 a 10.0.2.15), y **192.168.24.0/22** agrupa las dos VLAN.

```
enable
configure terminal
hostname R2

interface gigabitEthernet 0/0
 description PP2 Ruta principal hacia Switch Capa 3
 ip address 10.0.2.6 255.255.255.252
 no shutdown
 exit

interface gigabitEthernet 0/1
 description PP3 Ruta flotante hacia Switch Capa 3
 ip address 10.0.2.10 255.255.255.252
 no shutdown
 exit

! Rutas principales por PP2
ip route 10.0.0.0 255.255.252.0 10.0.2.5
ip route 192.168.24.0 255.255.252.0 10.0.2.5

! Rutas flotantes por PP3
ip route 10.0.0.0 255.255.252.0 10.0.2.9 5
ip route 192.168.24.0 255.255.252.0 10.0.2.9 5

end
copy running-config startup-config
```

### 5.7 PCs

En cada PC entrá a **Desktop → IP Configuration**, dejalo en **Static** y poné los datos de la tabla.

| PC | IP | Máscara | Gateway |
|---|---|---|---|
| PC0 | 192.168.25.10 | 255.255.255.0 | 192.168.25.1 |
| PC1 | 192.168.25.11 | 255.255.255.0 | 192.168.25.1 |
| PC4 | 192.168.26.10 | 255.255.255.0 | 192.168.26.1 |
| PC5 | 192.168.26.11 | 255.255.255.0 | 192.168.26.1 |
| PC2 | 10.0.0.10 | 255.255.254.0 | 10.0.0.1 |
| PC3 | 10.0.0.11 | 255.255.254.0 | 10.0.0.1 |

---

## 6. Resumen de todas las rutas estáticas

| Equipo | Destino | Máscara | Siguiente salto | AD | Para qué |
|---|---|---|---|---|---|
| Router0 | 192.168.24.0 | 255.255.252.0 | 10.0.2.14 | 1 | **Sumarizada** a VLAN 1 y 2 |
| Router0 | 10.0.2.0 | 255.255.255.252 | 10.0.2.14 | 1 | PP1 |
| Router0 | 10.0.2.4 | 255.255.255.252 | 10.0.2.14 | 1 | PP2 |
| Router0 | 10.0.2.8 | 255.255.255.252 | 10.0.2.14 | 1 | PP3 |
| Router1 | 0.0.0.0 | 0.0.0.0 | 10.0.2.2 | 1 | **Por defecto** al SW Capa 3 |
| SW Capa 3 | 10.0.0.0 | 255.255.254.0 | 10.0.2.13 | 1 | LAN 1 |
| SW Capa 3 | 192.168.25.0 | 255.255.255.0 | 10.0.2.1 | 1 | VLAN 1 |
| SW Capa 3 | 192.168.26.0 | 255.255.255.0 | 10.0.2.1 | 1 | VLAN 2 |
| SW Capa 3 | 0.0.0.0 | 0.0.0.0 | 10.0.2.6 | 1 | **Principal** a Router2 |
| SW Capa 3 | 0.0.0.0 | 0.0.0.0 | 10.0.2.10 | 5 | **Flotante** a Router2 |
| Router2 | 10.0.0.0 | 255.255.252.0 | 10.0.2.5 | 1 | Regreso principal |
| Router2 | 192.168.24.0 | 255.255.252.0 | 10.0.2.5 | 1 | Regreso principal |
| Router2 | 10.0.0.0 | 255.255.252.0 | 10.0.2.9 | 5 | Regreso flotante |
| Router2 | 192.168.24.0 | 255.255.252.0 | 10.0.2.9 | 5 | Regreso flotante |

---

## 7. Cómo comprobar que todo funciona

### Pings

Desde el **Command Prompt** de las PCs.

| Desde | Hacia | Qué demuestra |
|---|---|---|
| PC0 | 192.168.26.10 (PC4) | Router-on-a-stick entre VLAN |
| PC0 | 10.0.0.10 (PC2) | VLAN 1 llega a LAN 1 |
| PC4 | 10.0.0.11 (PC3) | VLAN 2 llega a LAN 1 |
| PC2 | 10.0.2.6 (Router2) | LAN 1 llega a Router2 |
| PC5 | 10.0.2.10 (Router2) | VLAN 2 llega a Router2 |

El primer ping a veces pierde uno o dos paquetes mientras se resuelve ARP. Es normal, repetilo.

### Revisar las tablas de enrutamiento

- En **Router0**, `show ip route` tiene que mostrar `S 192.168.24.0/22 [1/0] via 10.0.2.14`.
- En **Router1**, tiene que aparecer `S* 0.0.0.0/0 [1/0] via 10.0.2.2`.
- En el **Switch Capa 3**, tiene que aparecer `S* 0.0.0.0/0 [1/0] via 10.0.2.6`. La flotante **no** se ve todavía, y eso está bien.

### Probar la ruta flotante

1. En el Switch Capa 3 apagá el enlace principal.
   ```
   configure terminal
   interface fastEthernet 0/1
    shutdown
   end
   show ip route
   ```
2. Ahora la ruta tiene que cambiar a `S* 0.0.0.0/0 [5/0] via 10.0.2.10`. Eso confirma que la flotante entró.
3. Hacé ping desde PC0 a 10.0.2.10 y tiene que responder por el camino de respaldo.
4. Volvé a encender el enlace con `no shutdown` en F0/1 y la ruta principal regresa sola con `[1/0]`.

---

## 8. Errores comunes que conviene revisar

- **Olvidar `ip routing`** en el 3560. Las interfaces tienen IP, pero nada pasa entre redes.
- **Olvidar `no switchport`** en los puertos del 3560 hacia routers. Sin eso no deja poner la IP.
- **No encender la interfaz física G0/1 de Router1**. Las subinterfaces no funcionan si la física está en `shutdown`.
- **Máscara incorrecta en la LAN 1**. Tiene que ser 255.255.254.0 en Router0 y en PC2 y PC3.
- **Router2 sin rutas de regreso**. Los pings hacia Router2 fallan aunque el Switch Capa 3 sí tenga la ruta.
- **Poner la ruta flotante con la misma AD que la principal**. Si ambas tienen AD 1, el equipo balancea carga entre las dos en lugar de dejar una de respaldo.
- **No guardar** con `copy running-config startup-config` antes de cerrar el archivo.

---

## 9. Nota sobre Router2

En el archivo, Router2 no tiene ninguna LAN propia, solo los dos enlaces al Switch Capa 3. Por eso la ruta principal y la flotante se hicieron como **rutas por defecto** hacia Router2. Si el profesor pide que Router2 tenga una red de destino específica, se le puede crear una loopback (por ejemplo `interface loopback 0` con `ip address 10.0.3.1 255.255.255.0`) y cambiar las dos rutas del Switch Capa 3 para que apunten a esa red en lugar de 0.0.0.0. La lógica de principal con AD 1 y flotante con AD 5 queda exactamente igual.
