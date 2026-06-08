# Node-RED Dashboard – Linux CPU mérés

Importálható Node-RED flow JSON fájlok Linux CPU-használat megjelenítéséhez.

## „Flows stopped due to missing node types” – gyors megoldás

Ha ezt látod:

```
Flows stopped due to missing node types.
    ui_base
    ui_tab
    ui_group
```

akkor a **klasszikus Dashboard 1.0** flow van betöltve (`cpu-dashboard-linux-v1.json`), de a `node-red-dashboard` csomag **nincs telepítve**.

### Ajánlott megoldás (FlowFuse Dashboard 2.0)

1. **Töröld** a hibás flow tabot a Node-RED szerkesztőben (jobb klikk a „CPU Dashboard” fülön → Delete)
2. Telepítsd a csomagot:

```bash
cd ~/.node-red
npm install @flowfuse/node-red-dashboard
```

3. **Indítsd újra** a Node-RED-et
4. Importáld: `flows/cpu-dashboard-linux.json`
5. **Deploy**
6. Nyisd meg: `http://<gép-címe>:1880/dashboard`

### Alternatíva (klasszikus Dashboard 1.0 megtartása)

Ha a régi dashboardot szeretnéd:

```bash
cd ~/.node-red
npm install node-red-dashboard
```

Node-RED **újraindítása**, majd Deploy. A meglévő v1 flow ekkor működni fog.

Dashboard URL: `http://<gép-címe>:1880/ui`

## „unknown: ui_gauge” hiba – gyors megoldás

Ez a hiba azt jelenti, hogy **nincs telepítve** a megfelelő Dashboard csomag. Két út van:

### A) FlowFuse Dashboard 2.0 (ajánlott, új telepítésekhez)

```bash
cd ~/.node-red
npm install @flowfuse/node-red-dashboard
```

Node-RED **újraindítása**, majd importáld:

```
flows/cpu-dashboard-linux.json
```

Dashboard URL: `http://<gép-címe>:1880/dashboard`

### B) Klasszikus Dashboard 1.0 (régi `node-red-dashboard`)

```bash
cd ~/.node-red
npm install node-red-dashboard
```

Node-RED **újraindítása**, majd importáld:

```
flows/cpu-dashboard-linux-v1.json
```

Dashboard URL: `http://<gép-címe>:1880/ui`

> **Fontos:** A csomag telepítése után mindig indítsd újra a Node-RED-et! Palette Managerből történő telepítés is működik, de restart szükséges.

## Melyik flow melyik csomaghoz?

| Fájl | Dashboard csomag | Node típusok | URL |
|------|------------------|--------------|-----|
| `cpu-dashboard-linux.json` | `@flowfuse/node-red-dashboard` | `ui-gauge`, `ui-chart`, `ui-text` | `/dashboard` |
| `cpu-dashboard-linux-flowfuse.json` | ugyanaz (azonos tartalom) | ugyanaz | `/dashboard` |
| `cpu-dashboard-linux-v1.json` | `node-red-dashboard` | `ui_gauge`, `ui_chart`, `ui_text` | `/ui` |

## Flow importálása

1. Nyisd meg: `http://<gép-címe>:1880`
2. Menü (☰) → **Import**
3. Válaszd ki a megfelelő JSON fájlt (lásd fenti táblázat)
4. **Deploy**

## Mit tartalmaz a flow?

| Elem | Leírás |
|------|--------|
| **Inject** | 5 másodpercenként indít egy mérést |
| **Function** | A `/proc/stat` fájlból számolja a CPU %-ot |
| **Gauge** | Mérő 0–100% tartományban (zöld / sárga / piros zónák) |
| **Chart** | Idősor grafikon |
| **Text** | Szöveges aktuális érték |

### CPU számítás

```
CPU% = (1 - Δidle / Δtotal) × 100
```

Az **első mérés** gyakran `0%`, mert még nincs előző minta. A **második méréstől** pontos.

## Jogosultságok

A Node-RED folyamatának olvasnia kell a `/proc/stat` fájlt. Normál Linux felhasználóval ez általában működik.

Dockerben: `-v /proc:/host/proc:ro` és a function node-ban `/host/proc/stat` útvonal.

## Hibaelhárítás

| Probléma | Megoldás |
|----------|----------|
| `missing: ui_base, ui_tab, ui_group` | v1 flow van deployolva csomag nélkül → telepítsd `node-red-dashboard`-ot VAGY töröld a flow-t és használd a fő `.json` fájlt FlowFuse-szal |
| `unknown: ui_gauge` | Telepítsd: `npm install node-red-dashboard`, restart, használd a `-v1.json` fájlt |
| `unknown: ui-gauge` | Telepítsd: `npm install @flowfuse/node-red-dashboard`, restart, használd a fő `.json` fájlt |
| Dashboard 404 | Ellenőrizd az URL-t: `/dashboard` (v2) vagy `/ui` (v1) |
| Mindig 0% | Várj legalább egy frissítési ciklust (5 mp) |
| Permission denied | Ellenőrizd a `/proc/stat` olvasási jogát |

## Testreszabás

- **Frissítési időköz:** Inject node → `repeat` (alapértelmezés: 5 mp)
- **Színzónák:** Gauge node → segments (0% zöld, 50% sárga, 80% piros)
