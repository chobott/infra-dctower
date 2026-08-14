# Infrastruktura Datoveho Centra - DCTower

Tento repozitar obsahuje dokumentaci a konfiguraci pro infrastrukturu datoveho centra postavenou na:

- **VMware vSphere** - Virtualizacni platforma
- **Cisco UCS** - Serverova infrastruktura (2x UCS servery)
- **Cisco Nexus/Catalyst** - Sitova infrastruktura
- **QNAP NAS** - Diskove pole pro ulozeni dat

## Struktura repozitare

```
infra-dctower/
├── README.md              # Tento soubor
├── docs/
│   └── architecture.md    # Architekturni dokumentace
├── config/
│   ├── vmware/            # VMware konfigurace
│   ├── cisco-ucs/         # Cisco UCS konfigurace
│   ├── cisco-switch/      # Cisco switch konfigurace
│   └── qnap/              # QNAP NAS konfigurace
└── diagrams/              # Sitove diagramy (volitelne)
```

## Prehled architektury

Viz [docs/architecture.md](docs/architecture.md) pro podrobnou dokumentaci.

## Poznamka

Vsechny soubory jsou ulozeny v UTF-8 kodovani pro spravnou podporu ceske diakritiky.
