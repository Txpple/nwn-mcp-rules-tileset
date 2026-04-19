---
paths:
  - "**/*.set"
  - "**/*.itp"
  - "**/*.mdl"
  - "**/*.wok"
---

# NWN EE Tileset Operations

Procedures and troubleshooting for common tileset tasks. See companion rules for file format details: nwn-set-file-format, nwn-itp-palette-format.

## How the Tile Solver Works

The toolset tile solver automatically places tiles when terrain is painted:
- Matches tile **corner terrains** (TopLeft, TopRight, BottomLeft, BottomRight)
- Matches tile **edge crossers** (Top, Right, Bottom, Left)
- Respects `Border=` and `Default=` from `[GENERAL]` for area edges

## Terrain Families

A tileset can support multiple visual families by giving each family its own terrain/crosser names. The solver treats them independently — painting `FloorStone` only uses tiles with `FloorStone` corners.

**Critical rule: the border terrain must be shared across all families.** `Border=` and `Default=` can only reference one name. Splitting the border terrain per family silently breaks painting for some families.

### Split Rules

| Type | Shared or Split? | Reason |
|------|------------------|--------|
| Border terrain | **Shared** | Required by Border=/Default=. Splitting breaks solver. |
| Non-border terrains | Split per family | Family-specific visuals |
| All crossers | Split per family | Family-specific appearance |

## Reordering Tiles

Sort all tiles alphabetically by `Model=`. Update:
1. `[TILExx]` headers — renumber sequentially from 0
2. `[TILExxDOORn]` headers — match new parent tile index
3. `TileN=` in groups — remap old index to new index
4. Door sections stay immediately after their parent tile

Do NOT change: tile properties, door contents, group metadata, Count (unless tiles added/removed).

## Duplicating a Tile Family

Creates new tiles sharing geometry but with different terrain/crosser names for independent painting.

1. Choose series letters for the new family
2. Add new terrain types with family suffix — keep border terrain shared
3. Add new crosser types with family suffix
4. Update terrain/crosser counts
5. For each source tile, create duplicate with:
   - Model name with new series letter
   - ImageMap2D = `mi_` + new model name
   - Corner terrains with new family suffix (border terrain stays as-is)
   - Edge crossers with new family suffix
   - Door sections duplicated
6. Duplicate groups pointing to new tile indices
7. Copy .mdl and .wok files, find/replace model name inside each
8. Sort all tiles by model name and renumber
9. Update ITP: add family subfolder with `ID=2`, add terrain/crosser paint entries

## Adding a Single Tile

1. Add tile entry in .set with correct terrain/crosser references
2. Place .mdl and .wok files matching `Model=`
3. Increment `[TILES] Count=`
4. Re-sort and renumber if maintaining order
5. Update group references if indices shifted
6. Add to ITP palette if needed

## Troubleshooting

**Terrain painting shows entries but solver finds no solution:**
- Check ITP subfolder ID values — must be `ID=2`, not `ID=3` (#1 cause)
- Check border terrain is shared — not split per family
- Check RESREF matches .set terrain/crosser Name (case-insensitive)

**One family paints fine, another doesn't:**
- Both families must share the border terrain
- Check the non-working family's tiles reference correct suffixed names
- Check ITP subfolder for that family uses `ID=2`

**Tiles don't appear in toolset:**
- Verify .set `Count=` matches actual tile count
- Verify .mdl and .wok files exist with matching names
- Check ITP palette has entries with correct RESREFs

**Door doesn't spawn on a tile:**
- `Doors=N` must match count of `[TILExxDOORn]` sections
- Door section header index must match tile index

## Verification Checklist

Run after any modification:

- [ ] Tile headers sequential 0 to N-1
- [ ] Models in sorted order
- [ ] `[TILES] Count=` matches actual count
- [ ] Door sections match parent tiles
- [ ] Tiles with `Doors > 0` have `DoorVisibilityNode` and `DoorVisibilityOrientation` (flag to user if missing)
- [ ] Group references point to valid tile indices
- [ ] `[TERRAIN TYPES] Count=` and `[CROSSER TYPES] Count=` match actual
- [ ] Every defined terrain/crosser used by at least one tile (and vice versa)
- [ ] Border terrain is shared (not split per family)
- [ ] `Border=` and `Default=` reference the shared border terrain
- [ ] Every .set model has matching .mdl and .wok (no orphans)
- [ ] ITP paint subfolders use `ID=2`
- [ ] ITP RESREFs match .set names
