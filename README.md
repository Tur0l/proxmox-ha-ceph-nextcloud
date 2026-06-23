# Clúster Proxmox VE con Alta Disponibilidad, ZFS, Ceph y Nextcloud

> Infraestructura hiperconvergente (HCI) completa construida íntegramente con software libre sobre un único servidor, capaz de competir con soluciones comerciales como VMware vSphere a una fracción del coste.

**Autor:** Alberto Arturo Morales García · **Trabajo Fin de Ciclo — ASIR** · 2026
I.E.S. Julián Marías (Valladolid) · Tutor: Ángel Tomás Domínguez Pequeño

---

## Resumen

Despliegue de un clúster **Proxmox VE 9** de tres nodos en **alta disponibilidad** sobre un servidor HP ProLiant DL360p Gen8 mediante **virtualización anidada**. Los nodos forman un clúster real con quorum y HA gestionada por `ha-manager`, almacenamiento **ZFS local con replicación** periódica y almacenamiento distribuido **Ceph** (RBD + CephFS). Sobre esa base se despliega **Nextcloud** como nube privada (instancias sobre ZFS y sobre Ceph), un LXC con **Caddy** como CA interna y reverse proxy TLS, y un modelo de **seguridad por capas** (firewall, Fail2Ban, 2FA TOTP, HTTPS con PKI interna).

> **English** — Complete open-source hyper-converged infrastructure: a 3-node high-availability Proxmox VE cluster (nested virtualization) with replicated ZFS, distributed Ceph and Nextcloud as a private cloud, on a single second-hand enterprise server. Verified in lab: HA failover in **140 s**, OSD recovery to `HEALTH_OK` in **24 s**, and **1,711 IOPS** randwrite 4K over Ceph RBD.

---

## Arquitectura

```
┌──────────────────────────────────────────────────────────────┐
│  Anfitrión físico — HP ProLiant DL360p Gen8 (nested-virt)      │
│                                                                │
│   ┌────────────┐    ┌────────────┐    ┌────────────┐           │
│   │ pve-nodo1  │    │ pve-nodo2  │    │ pve-nodo3  │           │
│   │ 24 GB RAM  │    │ 24 GB RAM  │    │ 24 GB RAM  │           │
│   └────────────┘    └────────────┘    └────────────┘           │
│        Clúster Proxmox VE · quorum · ha-manager                │
│   ── ZFS local + replicación (pvesr) ──────────────────        │
│   ══ Ceph distribuido (RBD + CephFS) — compartido ════         │
└──────────────────────────────────────────────────────────────┘
```

### Almacenamiento en tres capas

| Capa | Tecnología | Ámbito | Función |
|---|---|---|---|
| 1 — Sistema | ext4 sobre LVM | Local a cada nodo | SO Proxmox y configuración local del nodo |
| 2 — Datos rápidos | ZFS local + replicación | Local, sincronizado por `pvesr` | VMs con bajo overhead, snapshots instantáneos, RPO ajustable |
| 3 — Datos distribuidos | Ceph (RBD + CephFS) | Compartido entre los 3 nodos | VMs en HA y recursos compartidos (ISOs, backups, plantillas) |

---

## Pruebas ejecutadas en laboratorio

Telemetría a 1 Hz conservada junto a los scripts de prueba.

| Prueba | Resultado |
|---|---|
| **Failover HA (Ceph)** | VM rearrancada en otro nodo en **140 s** tras *kill* abrupto. Cumple RNF.5 (< 180 s). |
| **Pérdida de OSD** | Detección en **2 s**, servicio operativo durante toda la caída, recovery a `HEALTH_OK` en **24 s**. |
| **`pveperf` (3 nodos)** | READS ~310–333 MB/s · FSYNCS 554–703/s · DNS interno < 5 ms. |
| **`fio` sobre RBD Ceph** | randwrite 4 KiB → **1.711 IOPS** · randread 4 KiB → 929 IOPS · seqwrite 1 MiB → 116 MiB/s. |

---

## Funcionalidades

- Gestión de VMs (crear, clonar, arrancar, parar, migrar) desde interfaz web centralizada.
- Alta disponibilidad con failover automático ante caída de nodo.
- Snapshots y rollback de cualquier VM.
- Replicación automática de datos entre nodos mediante ZFS.
- Almacenamiento Ceph distribuido accesible simultáneamente desde todos los nodos.
- Migración en vivo (live migration) sin interrupción del servicio.
- Nextcloud como nube privada corporativa accesible vía web.
- Endurecimiento por capas: firewall, Fail2Ban, 2FA TOTP y HTTPS con PKI interna.

---

## Pila tecnológica

`Proxmox VE 9.1` · `Ceph Reef (RBD + CephFS)` · `ZFS` · `Nextcloud` · `Caddy (CA interna + reverse proxy)` · `Pi-hole (DNS local)` · `Fail2Ban` · `TOTP` · `Debian 12/13`

## Hardware del laboratorio

- **HP ProLiant DL360p Gen8** (rack 1U) — 2 × Intel Xeon E5-2630 v1 (12c/24t), **96 GB DDR3 ECC**.
- 4 × HDD SAS 2 TB en **RAID 10** (4 TB útiles), 4 × NIC 1 GbE + iLO 4 dedicado.
- Proxmox VE 9.1 sobre Debian; tres VMs-nodo de 24 GB RAM con virtualización anidada.

---

## Viabilidad económica

Coste de hardware de partida: **~200 €** (servidor de segunda mano). **0 € en licencias** (100 % software libre). Ahorro estimado de **4.800 €/año** frente a soluciones propietarias y **6.400 €/año** frente a cloud público; coste operativo de **378 €/año** en producción 24/7.

---

## Documentación

- **Memoria principal** (~60 págs): introducción, análisis, recursos, metodología, diseño detallado, implementación, fase de pruebas, trabajo futuro, conclusiones, anexo de comandos y bibliografía APA.
- **Anexo I — Manual de implementación** (~155 págs): 19 fases independientes y reproducibles con comandos, opciones de GUI y capturas.

---

## Conclusiones

El proyecto demuestra que se puede desplegar una infraestructura hiperconvergente completa usando solo software libre, con todas las funcionalidades implementadas **y verificadas con pruebas reales**. La virtualización anidada permite simular un entorno de producción con un único servidor físico, y el manual paso a paso sirve como guía reproducible para futuros despliegues en pymes y entornos formativos.

---

<sub>Repositorio de portfolio. Desarrollado como Trabajo Fin de Ciclo del Grado Superior en Administración de Sistemas Informáticos en Red (ASIR).</sub>
