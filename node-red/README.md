# Node-RED Dashboard – Linux CPU mérés

Ez a mappa egy **importálható Node-RED flow JSON** fájlt tartalmaz, amely Linux alatt a processzorhasználatot jeleníti meg a Node-RED Dashboard felületén.

## Előfeltételek

1. **Node-RED** telepítve és fut
2. **Dashboard csomag** telepítve (klasszikus UI):

```bash
cd ~/.node-red
npm install node-red-dashboard
```

3. Node-RED újraindítása a telepítés után

> **Megjegyzés:** A FlowFuse új dashboard csomagja (`@flowfuse/node-red-dashboard`) más node-típusokat használ. Ez a flow a széles körben elterjedt `node-red-dashboard` (ui_* node-ok) csomaghoz készült.

## Flow importálása

1. Nyisd meg a Node-RED szerkesztőt: `http://<gép-címe>:1880`
2. Menü (☰) → **Import**
3. Válaszd a **select a file to import** lehetőséget, vagy másold be a JSON tartalmát
4. Tallózd be a fájlt: `flows/cpu-dashboard-linux.json`
5. Kattints az **Import** gombra
6. **Deploy** (telepítés) a jobb felső sarokban

## Dashboard megnyitása

A telepítés után a dashboard elérhető:

```
http://<gép-címe>:1880/ui
```

## Mit tartalmaz a flow?

| Elem | Leírás |
|------|--------|
| **Inject** | 5 másodpercenként indít egy mérést |
| **Function** | A `/proc/stat` fájlból számolja a CPU %-ot |
| **Gauge** | Óra-szerű mérő 0–100% tartományban |
| **Chart** | Idősor grafikon (utolsó 1 óra adatai) |
| **Text** | Szöveges aktuális érték |

### Hogyan számolja a CPU-t?

A function node két egymást követő mintát olvas a `/proc/stat` első sorából (`cpu` összesített sor), majd ebből számítja:

```
CPU% = (1 - Δidle / Δtotal) × 100
```

Az **első mérés** gyakran `0%`, mert még nincs előző minta az összehasonlításhoz. A **második méréstől** az érték pontos.

## Jogosultságok

A Node-RED folyamatának olvasnia kell a `/proc/stat` fájlt. Normál Linux felhasználóval ez általában működik, mert a fájl world-readable.

Ha Node-RED konténerben fut, győződj meg róla, hogy a host `/proc` elérhető (pl. Docker: `-v /proc:/host/proc:ro` és a function node-ban `/host/proc/stat` útvonal).

## Testreszabás

- **Frissítési időköz:** az Inject node `repeat` mezője (alapértelmezés: 5 mp)
- **Színzónák a mérőn:** `seg1` = 50%, `seg2` = 80% (zöld / sárga / piros)
- **Grafikon időtartama:** Chart node `removeOlder` = 1 óra

## Hibaelhárítás

| Probléma | Megoldás |
|----------|----------|
| Nincs `ui_gauge` node | Telepítsd: `npm install node-red-dashboard`, majd restart |
| Dashboard 404 | Deploy után ellenőrizd: `http://host:1880/ui` |
| Mindig 0% | Várj legalább egy frissítési ciklust (5 mp) |
| Permission denied | Ellenőrizd a `/proc/stat` olvasási jogát |

## Fájlok

- `flows/cpu-dashboard-linux.json` – importálható Node-RED flow
