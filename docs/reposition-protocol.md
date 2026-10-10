[reposition-protocol.md](https://github.com/user-attachments/files/33292168/reposition-protocol.md)
# Position-adjustment trace protocol — `reposition/1`

(Protocole de traçabilité du recalage. Les métadonnées sont en anglais, dates en ISO 8601 UTC, coordonnées en degrés décimaux WGS84 (EPSG:4326) ; l'interface de l'outil reste en français.)

## Principle
Each adjusted photo carries its own adjustment state in EXIF, so the change is reversible. One photo = one "version"; each write session = one "changeset" (as in OSM: id, timestamp, tool, list of edits). The same information is mirrored as Panoramax semantic tags for photos already online.

## Where it is written (EXIF GPS IFD)
- `GPSAreaInformation` (0x1C, UNDEFINED, `ASCII\0\0\0` prefix): `reposition/1 {JSON}`
- `GPSProcessingMethod` (0x1B): `MANUAL`
- `GPSLatitude/Longitude`, `GPSImgDirection`: adjusted values
- Untouched: XMP GPano (360° detection by Panoramax), DateTimeOriginal, make/model, thumbnail.
- Readable with ExifTool (`-GPSAreaInformation`). When a sequence is loaded from Panoramax, the tool reads `Exif.GPSInfo.GPSAreaInformation` from `properties.exif` if present (to confirm on the real instance).

## JSON keys (ASCII, ≤ 1500 chars)
| key | meaning |
|---|---|
| `cs` | changeset id (`cs-YYYYMMDDHHMMSS-xxxx`) |
| `ts` | modification timestamp, ISO 8601 UTC |
| `tool` | `seq360-adjust/1` |
| `state` | `adjusted` or `reverted` |
| `orig` | **original position [lat, lon]**, never overwritten by later adjustments |
| `dist_m` | displacement from the original position (metres) |
| `method` | `anchor`, `redistribution` or `manual` |
| `anchor.theme` / `anchor.tag` | when the photo is itself an anchor: landmark theme and its OSM-style tag |
| `prev`, `next` | previous / next anchors used: `[theme, file]` |
| `hist` | last 3 previous states: `{cs, ts, pos}` |

Example:
```
reposition/1 {"cs":"cs-20261010100000-ab12","ts":"2026-10-10T10:00:00Z","tool":"seq360-adjust/1","state":"adjusted","orig":[46.5,3.26],"dist_m":18.92,"method":"anchor","anchor":{"theme":"milestone","tag":"osm|highway=milestone"}}
```

## Anchor themes → OSM-style tags (Panoramax `osm|key=value` syntax)
| theme | tag |
|---|---|
| `crossing` | `osm|highway=crossing` |
| `stop_line` | `osm|road_marking=stop_line` (*) |
| `give_way` | `osm|highway=give_way` |
| `bridge` | `osm|bridge=yes` |
| `junction` | `osm|junction=yes` (*) |
| `pole` | `osm|man_made=utility_pole` |
| `milestone` | `osm|highway=milestone` |
| `traffic_island` | `osm|traffic_calming=island` |
| `median_kerb` | `osm|barrier=kerb` |
| `merge_arrow` | `osm|road_marking=arrow` (*) |
| `hatched_marking` | `osm|road_marking=hatched` (*) |
| `overtaking` | `reposition|exception=overtaking` (not a landmark) |
| `other` | `reposition|theme=other` |

(*) `road_marking=*` values and `junction=yes` on a point are not established OSM usage; to be checked on the OSM wiki before relying on them. The tag is stored as a qualified value (`reposition:anchor`), not as a bare `osm|…` tag on the photo, so it does not claim the photo is the object.

## Panoramax semantics (photos already uploaded)
Semantics are OSM-like `key=value` tags, free keys, unique key/value pair, `add`/`delete` actions on item PATCH (`properties.semantics`). On "send", the tool (option "poser les tags de traçabilité", on by default) writes for each moved photo:
- `reposition=adjusted`
- `reposition:changeset=cs-…`
- `reposition:timestamp=<ISO 8601>`
- `reposition:orig=<lat>,<lon>`
- `reposition:method=anchor|redistribution|manual`
- `reposition:anchor=osm|highway=milestone` (anchor photos only)

Older `reposition*` tags are removed (`delete`) when their value changes, so only the current state remains.

## Lifecycle
original (no block) → `adjusted` (orig set) → `adjusted` (hist grows, orig unchanged) → `reverted` (position = orig).
Revert is available from the photos themselves ("Restaurer les positions d'origine", then write).

## Changeset log (separate file)
`changeset_<id>.json`: id, timestamp, tool, source, destination, photo count, and per photo `from`, `to`, `orig`, `state`, `method`, `anchor`, `prev`, `next`.

## Backward compatibility
The reader still accepts an earlier French format `calage/1` (keys c, t, st, o, m, a, p, n, h) and converts it.

## Possible next version (reposition/2)
- Keep the original (wrong, e.g. 2016) capture datetime and the datetime resynchronised on a GPX trace.
- Reference anchors to OSM/Wikidata objects (`osm|node=…`, `wd|Q…`).
- Dedicated XMP namespace if the GPS field becomes too constrained.

## Checks done
EXIF write + read-back on a JPEG with GPano XMP: XMP and make preserved; `orig` stable across two adjustments; history stacked; legacy `calage/1` parsed. Semantic tag generation and add/delete diff tested offline.
Not tested: ExifTool, real Panoramax PATCH with `semantics`, Panoramax exposing `GPSAreaInformation`.
