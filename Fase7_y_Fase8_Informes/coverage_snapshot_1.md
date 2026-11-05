> **SNAPSHOT #1 — HISTÓRICO:** Este archivo representa la cobertura de la base de datos en Snapshot #1. Sus valores (por ejemplo, 403.922 ocurrencias inventariadas y 235.326 cadenas únicas pendientes) corresponden a ese corte y fueron ampliados después por la auditoría de Fase 9. Para métricas vigentes del corte auditado más reciente, consulta [CURRENT_PROJECT_STATE.md](../documentacion/HISTORICO.md).

---

# Informe de Cobertura Real — Snapshot #1

## 1. Resumen Global y Política ES-419 (Con Deduplicación Transparente)
- **Cadenas totales inventariadas (ocurrencias)**: 403922
- **Cadenas traducibles de mods (ocurrencias)**: 314449 *(Cadenas únicas deduplicadas: **235326**)*
- **Localización oficial de Bethesda preexistente (Skyrim + DLCs + Creation Club)**: 57261 ocurrencias *(48029 cadenas únicas)*
- **Traducciones comunitarias reutilizadas**: **0** *(100% excluidas por política ES-419)*
- **Traducidas y aprobadas en plataforma (ES-419 propio)**: 0 ocurrencias *(0 cadenas únicas)*
- **Protegidas por Glosario Maestro**: 8667
- **Pendientes de traducción (ocurrencias)**: 314449
- **Cadenas únicas netas pendientes (`unique_translatable_total - unique_approved_es419`)**: **235326** *(5415 de ellas cuentan con referencia contextual en tablas oficiales de Bethesda)*
- **Necesitan revisión humana / investigación de lore**: 718
- **No traducibles (código/variables/identificadores/técnico)**: 23545
- **No compatibles (recursos como .swf / audio)**: 687 *(Archivos no compatibles: 687)*
- **Extracción fallida**: 0
- **Cobertura real efectiva (incluyendo penalización por no compatibles/fallos)**: **17.30 %**

## 2. Desglose por Dominio Funcional
| Dominio | Encontrados | Traducibles | Bethesda Oficial | Aprobados ES-419 | Pendientes | Glosario | Revisión | No Trad. | No Comp. | Fallos | Cobertura Real |
| :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `books` | 9639 | 8820 | 746 | 0 | 8820 | 0 | 139 | 73 | 0 | 0 | **7.8%** |
| `dialogues` | 190540 | 148192 | 41325 | 0 | 148192 | 3 | 0 | 1020 | 0 | 0 | **21.8%** |
| `interfaces` | 15176 | 12812 | 2163 | 0 | 12812 | 78 | 0 | 123 | 678 | 0 | **14.2%** |
| `mcm_menus` | 12696 | 9149 | 0 | 0 | 9149 | 5 | 0 | 3542 | 0 | 0 | **0.1%** |
| `other_resources` | 718 | 718 | 0 | 0 | 718 | 0 | 0 | 0 | 9 | 0 | **0.0%** |
| `plugins` | 120792 | 99874 | 10060 | 0 | 99874 | 8556 | 579 | 2302 | 0 | 0 | **15.7%** |
| `quests` | 17504 | 14370 | 2967 | 0 | 14370 | 4 | 0 | 163 | 0 | 0 | **17.1%** |
| `scripts` | 36857 | 20514 | 0 | 0 | 20514 | 21 | 0 | 16322 | 0 | 0 | **0.1%** |
| `strings_files` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | **100.0%** |