# 🗺️ Orte-Katalog des Saar LiDAR Explorers

Der Katalog verbindet alle Orte des [Saar LiDAR Explorers](https://open-kirrberg.github.io/saar-lidar-explorer/):
welche Gemeinde welche 1-km-Kacheln zeigt, wo ihre Daten liegen, die grobe Übersicht für das ganze Land und die
Rangliste „Orte im Vergleich“. Veröffentlicht auf GitHub Pages:
`https://open-kirrberg.github.io/orte-katalog/catalog.json` (Format `sle-catalog-v1`, Datenvertrag des Explorers
„Many places (v3, S2)“ und „Catalog site (S3)“).

**Schauen, nicht graben.** Der Explorer zeigt Hypothesen, keine Funde.

## Was die Seite enthält

| Pfad | Inhalt | Quelle |
|---|---|---|
| `catalog.json` | alle 54 Gemeinden (52 Saarland + Zweibrücken, Bechhofen): Status, `data_url`, eigene Kacheln | gebaut aus `orte.toml` + VG25-Grenzen |
| `overview/` | zentrale Übersichtsebenen 2–6 (`sle-overview-v1`, mit `coverage`) | Daten-Bundle |
| `saarland/` | Rangliste Ebene „Gemeinden“ (stats-only Manifest + `stats/places.{json,geojson}`) | Daten-Bundle |
| `ortsteile/` | Rangliste Ebene „Ortsteile“ | Daten-Bundle |

## Grundsätze

- **Zuordnung der Kacheln** (`largest-share-v1`): eine Kachel gehört der Gemeinde mit dem größten Flächenanteil
  nach den amtlichen Grenzen (BKG VG25). Sie wird hier gerechnet, nie in einem Ort-Repository.
- **Ranglisten-Zahlen** werden zentral aus öffentlichen Quellen gerechnet (R3) und nie aus Ort-Repositories
  übernommen. Gemeinden ohne Repository erscheinen mit „Ortspate gesucht“.
- **Keine Last auf amtlichen Diensten:** der Workflow lädt nur das Daten-Bundle aus diesem Repository.

## Einen Ort eintragen

Pull Request auf [`orte.toml`](orte.toml): `[places.<id>]` mit `status`, `data_url`
(`https://open-kirrberg.github.io/ort-<id>/`), `repo`, `maintainers`. Die Prüfung im Pull Request baut den
Katalog probeweise; nach dem Übernehmen wird er veröffentlicht. Erst danach findet die Explorer-App den Ort.

## Daten-Bundle erneuern (Maintainer)

Die landesweiten Produkte passen nicht auf einen GitHub-Runner (Übersicht: ≈ 2 800 DGM1-Kacheln ≈ 11 GB;
Ranglisten-Bäume: DGM1 + DOM1 ≈ 22 GB, ≈ 90 min Download) und nicht in `actions/cache` (10 GB je Repository,
nach 7 Tagen ohne Zugriff gelöscht). Darum rechnet sie eine Maintainerin oder ein Maintainer lokal mit der
Explorer-Pipeline – nur, wenn es einen Grund gibt (neue Befliegung, neue VG25-Ausgabe, neue Disziplin):

```bash
cd saar-lidar-explorer/pipeline
./run.sh saarland                        # Rangliste Gemeinden (lange Läufe gecacht)
./run.sh orte                            # Rangliste Ortsteile
./run.sh orte-katalog                    # Zuordnung + Übersichtsebenen
uv run python katalog.py bundle --out ../../_katalog-release --name sle-katalog-daten-<datum>.tar
gh release create daten-<datum> ../../_katalog-release/sle-katalog-daten-<datum>.tar --repo Open-Kirrberg/orte-katalog \
  --title "Daten <datum>" --notes "Pipeline <version>; Quellen siehe ATTRIBUTION.txt"
```

Danach `daten.toml` (Tag, Asset, SHA-256 aus der `.sha256`-Datei) per Pull Request aktualisieren. Das Bundle
enthält nur Aggregate je Gemeinde / Ortsteil, Übersichtshöhen und die VG25-Grenzen – keine Flurstücke, Gebäude,
Personen oder Kampfmittel-Daten; `katalog.py bundle` und `katalog.py check` prüfen das.

## Lizenzen

Code und Workflows: MIT ([LICENSE](LICENSE)). Veröffentlichte Daten: unter den Lizenzen ihrer Quellen
(Namensnennung in `ATTRIBUTION.txt` der Seite), u. a.:

- © BKG (2026) CC BY 4.0 (Daten verändert), Datenquellen: https://sgx.geodatenzentrum.de/web_public/gdz/datenquellen/datenquellen_vg25.pdf (VG25)
- © GeoBasis-DE/LVGL-SL (2026), dl-de/by-2-0, Daten bearbeitet
- © GeoBasis-DE/LVermGeoRP (2026), dl-de/by-2-0, www.lvermgeo.rlp.de, Daten bearbeitet
- Zensus 2022, © Statistische Ämter des Bundes und der Länder, dl-de/by-2-0, Daten bearbeitet
- Marktstammdatenregister – © Bundesnetzagentur, dl-de/by-2-0, Daten bearbeitet
- Baumarten: DLR, Tree Species Germany 2022 (Wegler et al. 2025), CC BY 4.0, Daten bearbeitet (umprojiziert)
- GBIF.org Occurrence Search (nur Anzahlen), © OpenStreetMap-Mitwirkende, ODbL

Die vollständige, maßgebliche Liste erzeugt der Build aus den Manifesten (`ATTRIBUTION.txt`).
