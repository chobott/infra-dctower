# Architektura Datoveho Centra

## Prehled

Tato infrastruktura je navrzena jako **redundantni a vysoce dostupna** s ohledem na omezeny pocet serveru (2 ks). Klicovym prvkem je oddeleni sitovych VLAN pro management, VM traffic a storage, pricemz kazda cesta je vedena pres samostatny Fabric Interconnect pro maximalizaci odolnosti proti vypadkum.

## Hardwarove komponenty

### Servery (2x Cisco UCS)

- **Model**: Cisco UCS B-Series blade (napr. B200 M5/M6) v chassis UCS 5108 nebo UCS C-Series rack servery (napr. C220/C240 M5/M6)
- **Konfigurace**:
  - 2x CPU (Intel Xeon Scalable)
  - 256-512 GB RAM (pro konsolidaci vice VM)
  - 2x SSD pro lokalni boot ESXi (RAID1) nebo boot ze SAN (iSCSI z QNAP)
  - VIC 1340/1455 adapter pro 10/25 GbE konektivitu
- **Boot ze SAN**: Pokud QNAP podporuje iSCSI, doporucujeme boot ze SAN pro rychlejsi obnovu serveru pri vymene HW.

### Sitova infrastruktura (Cisco)

- **Top-of-Rack switche**: 2x Cisco Nexus 9300 nebo Cisco Catalyst 9300 (pro 10/25 GbE)
  - Kazdy UCS server je pripojen k obema switchum (dual-homed)
  - VPC (Virtual Port Channel) mezi switche pro redundantni uplink
- **Fabric Interconnects**: 2x Cisco UCS 6300/6400 Series (pokud pouzivate B-Series blades)
  - Kazdy FI pripojen k obema ToR switchum
  - Umoznuje oddeleni Fabric A a Fabric B pro redundantni cesty

### Uloziste (QNAP NAS)

- **Model**: QNAP Enterprise NAS (napr. TES-3085U, TVS-h1288X) s podporou iSCSI a NFS
- **Konfigurace**:
  - RAID6 nebo RAID10 pro data
  - 2x 10 GbE SFP+ porty (pripojene k obema Cisco switchum)
  - iSCSI LUNs pro:
    - Boot LUNs (volitelne)
    - VM datastores
    - vMotion traffic (volitelne oddeleny)
  - NFS pro backup nebo ISO repository
- **Best practice**: Kazdy QNAP controller (pokud dual-controller) pripojit k obema fabric (A/B).

## Sitovy design

### VLAN schema

| VLAN ID | Nazev | Ucel |
|---------|-------|------|
| 10 | MGMT | ESXi management, vCenter, UCS Manager |
| 20 | VM_DATA | Produkcní´´ VM traffic |
| 30 | STORAGE | iSCSI/NFS traffic (isolated) |
| 40 | vMOTION | vMotion traffic (volitelne oddeleno) |
| 50 | VCSA | vCenter Server Appliance |

### vSphere networking

- **vSwitch0**: Management + vMotion (2x vNIC, kazda z jineho Fabric)
- **vSwitch1**: VM traffic (2x vNIC, teaming pres obe Fabric)
- **VMkernel porty**:
  - vmk0: Management
  - vmk1: vMotion
  - vmk2: iSCSI (pokud boot ze SAN)
- **Distributed Switch (VDS)**: Doporuceno pro lepsi spravu a NSX readiness

## VMware vSphere design

### Hosts a Clusters

- **2x ESXi 8.0 hosts** v jednom clusteru
- **vCenter Server Appliance (VCSA)**: Nasazen jako VM na jednom z hostu (s HA restartem na druhem)
- **HA (High Availability)**: Povoleno s 1 host failure tolerance
- **DRS (Distributed Resource Scheduler)**: Povoleno pro automatickou zatezovou bilanci

### Storage

- **Datastore**: iSCSI LUN z QNAP (VMFS6)
- **Multipathing**: Round-robin nebo MRU (Most Recently Used)
- **Jumbo frames**: Povoleno na iSCSI VLAN (MTU 9000) pro lepsi vykon

## Redundance a vysoka dostupnost

- **Sit**: Kazdy server ma 2x uplink k ruznym ToR switchum
- **Storage**: QNAP dual-controller (pokud available) pripojeny k obema fabric
- **Power**: Kazdy server napajen z ruznych PDU/zdroju
- **HA**: vSphere HA automaticky restartuje VM na druhem hostu pri vypadku

## Schema zapojeni

```
[QNAP NAS] --10GbE-- [Cisco Nexus 1] --10/25GbE-- [UCS FI A] -- [UCS Server 1]
     |                    |                           |
     |                    |                           |
[QNAP NAS] --10GbE-- [Cisco Nexus 2] --10/25GbE-- [UCS FI B] -- [UCS Server 2]
```

## Doporuceni pro implementaci

### Pred nasazenim

1. Nakonfigurujte UCS Manager service profiles s oddelenymi vNIC pro kazdy typ traffic
2. Vytvorte VLAN na Cisco switchech a povolte trunking na portech smerem k UCS FI

### Behem instalace

1. Nainstalujte ESXi na kazdy UCS server
2. Nakonfigurujte vSwitch a VMkernel porty podle vyse uvedeneho schematu
3. Pripojte iSCSI storage z QNAP a vytvorte datastore

### Po instalaci

1. Nasaďte vCenter Server Appliance
2. Vytvorte cluster s HA a DRS
3. Nakonfigurujte backup (napr. Veeam na samostatnou VM nebo externalni backup target)

## Zaver

Tato architektura poskytuje solidni zaklad pro produkcní´´ prostredi s moznosti budouciho rozsireni (pridani dalsich UCS serveru, upgrade na vSAN nebo NSX).
