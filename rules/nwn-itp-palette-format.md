---
paths:
  - "**/*.itp"
  - "**/*palstd*"
---

# NWN EE .ITP Palette File Format

Reference for the .itp toolset palette file used by NWN tilesets. Requires `nwn_gff` from neverwinter.nim tools on the system PATH.

## Converting

```bash
nwn_gff -i tileset_palstd.itp -o tileset_palstd.json   # binary -> editable JSON
nwn_gff -i tileset_palstd.json -o tileset_palstd.itp    # JSON -> binary
```

## Branch Structure

The ITP `MAIN` list has three branches identified by `ID`:

| ID | Branch | STRREF | Purpose |
|----|--------|--------|---------|
| 0 | Features | 63261 | Individual tile/feature placement |
| 1 | Groups | 63262 | Group placement |
| 2 | Terrain Paint | 8282 | Paintable terrains/crossers — drives the tile solver |

## Subfolder ID Values — Critical

Within branches, entries and subfolders have an `ID` field that controls behavior. Getting this wrong causes **silent failures**:

| ID Value | Meaning | Solver behavior |
|----------|---------|-----------------|
| `0` | Tile picker subfolder | Visual grouping in the tile picker (branch 0) |
| `2` | **Terrain paint subfolder** | Solver CAN resolve — painting works |
| `3` | **Tile placement subfolder** | Solver IGNORES — specific model placement only |

**If terrain painting shows entries but the solver finds no solution, check subfolder ID values first.** Paint subfolders MUST use `ID=2`. Using `ID=3` fails silently — entries appear in the palette but painting does nothing.

## Terrain Paint Branch Layout

Example with two visual families sharing geometry:

```
Branch ID=2 (Terrain Paint)
+-- Wall (top-level, RESREF="wall")              shared border terrain
+-- Eraser (RESREF="eraser", STRREF=63291)
+-- Family A (subfolder, ID=2)                    ID=2 = solver resolves
|   +-- Floor (RESREF="FloorFamilyA")
|   +-- Corridor (RESREF="CorridorFamilyA")
|   +-- Doorway, Fence, Pit, Bridge, etc.
|   +-- Terrain Transitions (subfolder, ID=3)     ID=3 = specific tile placement
|       +-- Stairs (RESREF="mytileset_h02_02")    references a specific model
+-- Family B (subfolder, ID=2)
    +-- Floor (RESREF="FloorFamilyB")
    +-- ... same structure
    +-- Terrain Transitions (subfolder, ID=3)
        +-- Stairs (RESREF="mytileset_s02_02")
```

Key rules:
- Shared terrains (Wall) go at the **top level**, outside family subfolders
- Family paint subfolders use **ID=2**
- Terrain Transition subfolders use **ID=3** (they place specific models, not paintable terrains)
- RESREF for paint entries must match terrain/crosser `Name` in the .set file (case-insensitive)
- RESREF for placement entries must match a model name

## JSON Entry Formats

Leaf entry:
```json
{
  "__struct_id": 0,
  "NAME": {"type": "cexostring", "value": "Floor"},
  "RESREF": {"type": "resref", "value": "FloorFamilyA"}
}
```

Subfolder:
```json
{
  "__struct_id": 0,
  "DELETE_ME": {"type": "cexostring", "value": "Family A"},
  "ID": {"type": "byte", "value": 2},
  "LIST": {"type": "list", "value": [ ...children... ]}
}
```

Field reference:
- `DELETE_ME` = folder display name in the toolset
- `NAME` = leaf entry display name
- `RESREF` = terrain/crosser name (paint) or model name (placement)
- `ID` = behavior (0=tile folder, 2=paint folder, 3=placement folder)
