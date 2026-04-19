---
paths:
  - "**/*.set"
  - "**/*.mdl"
  - "**/*.wok"
---

# NWN EE .SET File Format and Model Files

Reference for the .set tileset definition, .mdl model files, and .wok walkmesh files.

## Prerequisites: NWN Nim Tools

The **neverwinter.nim** toolset (nwn_gff, nwn_erf, etc.) is required for working with GFF binary files (like .itp). Must be on the system PATH. Download from https://github.com/niv/neverwinter.nim/releases. Restart terminal sessions after adding to PATH.

The .set file is plain text and needs no special tools.

## File Relationships

| File | Format | Purpose |
|------|--------|---------|
| `{tileset}.set` | INI-style text | Tileset definition — terrains, crossers, tiles, doors, groups |
| `{tileset}palstd.itp` | GFF binary | Toolset palette (see nwn-itp-palette-format rule) |
| `{model}.mdl` | ASCII text | 3D tile model |
| `{model}.wok` | ASCII text | Walkmesh for pathfinding |

Every tile's `Model=` value must have a matching .mdl and .wok file. No orphans in either direction.

## .SET File Sections (in order)

### [GENERAL]

```ini
Name=mytileset
Type=SET
Version=V1.0
Interior=1                ; 1 = interior, 0 = exterior
HasHeightTransition=0     ; 1 = supports height transitions
EnvMap=mytileset_refl
Transition=1
DisplayName=-1
UnlocalizedName=My Tileset
Border=Wall           ; terrain at area edges — MUST be shared if using multiple families
Default=Wall          ; default terrain for unpainted areas
Floor=Floor           ; default floor terrain
SelectorHeight=5      ; optional — not present in all tilesets
```

### [TERRAIN TYPES]

Defines terrain names for tile corners. Zero-indexed, `Count=` must match. Names are arbitrary — chosen by the tileset author. With multiple families, non-border terrains get a family suffix (e.g., `FloorPlain`, `FloorPainted`).

```ini
[TERRAIN TYPES]
Count=5

[TERRAIN0]
Name=Wall
UnlocalizedName=Wall
```

### [CROSSER TYPES]

Defines crosser names for tile edges. Zero-indexed, `Count=` must match. Names are arbitrary. With multiple families, crossers get family suffixes.

```ini
[CROSSER TYPES]
Count=4

[CROSSER0]
Name=Corridor
UnlocalizedName=Corridor
```

### [PRIMARY RULES] / [SECONDARY RULES]

Not used in practice for custom content. Difficult to implement and generally should be ignored. Set `Count=0`.

### [TILES]

```ini
[TILES]
Count=77

[TILE0]
Model=mytileset_a01_01
WalkMesh=msb01
TopLeft=Wall          ; terrain type (must match a TERRAIN name)
TopLeftHeight=0
TopRight=Wall
TopRightHeight=0
BottomLeft=Wall
BottomLeftHeight=0
BottomRight=Wall
BottomRightHeight=0
Top=                  ; crosser type (empty = none, must match a CROSSER name)
Right=
Bottom=
Left=
MainLight1=0
MainLight2=0
SourceLight1=0
SourceLight2=0
AnimLoop1=0
AnimLoop2=0
AnimLoop3=0
Doors=0
Sounds=0
PathNode=P
Orientation=0
ImageMap2D=mi_mytileset_a01_01   ; always "mi_" + model name
```

Visibility properties:
- `VisibilityNode`, `VisibilityOrientation` — present on most tiles
- `DoorVisibilityNode`, `DoorVisibilityOrientation` — required when `Doors > 0`. **If a tile has doors but is missing these fields, flag it to the user** — absent door visibility nodes can cause visual culling issues in-game.

### Door Sub-Sections

If `Doors=N` (N > 0), exactly N sections follow named `[TILExxDOORn]`:

```ini
[TILE5DOOR0]
Type=6003
X=0.00
Y=5.00
Z=0.00
Orientation=0.00
```

Door header index must match parent tile index. The `Type` value is an index into the game's `doortypes.2da` file, which defines door models and properties.

### [GROUPS]

```ini
[GROUPS]
Count=3

[GROUP0]
Name=Double Door
Rows=1
Columns=1
Tile0=5               ; tile INDEX — must be remapped when tiles are reordered
```

## Model Naming Convention

Pattern: `{tileset}_{series}{number}_{variant}` (e.g., `mst06_b03_01`)

- Series letter identifies the tile's role (wall, corridor, door, etc.)
- Number identifies the tile within the series
- Variant `_02`, `_03` for visual/functional alternatives
- `ImageMap2D` is always `mi_` + model name

## MDL/WOK Internal References

Both files reference the model name internally. When copying/renaming, `content.replace(old_name, new_name)` covers all occurrences:

**MDL:** `newmodel`, `setsupermodel`, `beginmodelgeom`/`endmodelgeom`, `node dummy`, `parent` (many), `node light {name}ml1`/`ml2`, `donemodel`

**WOK:** `beginwalkmeshgeom`/`endwalkmeshgeom`, `parent`
