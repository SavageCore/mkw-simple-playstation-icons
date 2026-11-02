# Mario Kart World: PlayStation Icon Swap Guide

This guide covers how to replace the Nintendo Switch button icons (A/B/X/Y) with PlayStation shapes (○/✕/△/□) in matching positions.

Game dump used here: `/home/savagecore/Roms/Switch 2/Mario Kart World/base/romfs/`

## 0. Target file (confirmed)

The face-button glyphs live in the font, **not** in a layout texture:

- `romfs/Font/PicFont_00.Nin_Ounce_NVN2.bffnt.zs`
  (zstd-compressed BFFNT, `Ounce` = Switch 2 codename, `NVN2` = API)

Sheet: one 1024x1024 RGBA8 texture at BFFNT file offset `0x2000`
(BNTX at `0x1000`, blocklinear-tiled Ounce/NVN2, `TileMode=0`,
`BlockHeightLog2=4`). CMAP covers `U+E0A0`-`U+E0CF`.

The positional mapping (same physical buttons, new art):

| Codepoint | Nintendo | PlayStation | Cell x (px, display orientation) |
|-----------|----------|-------------|----------------------------------|
| U+E0CE    | A        | ○ Circle    | 1-86                             |
| U+E0CD    | B        | ✕ Cross     | 106-191                          |
| U+E0CC    | X        | △ Triangle  | 211-296                          |
| U+E0CB    | Y        | □ Square    | 316-401                          |

The KFL PlayStation mod ships white glyph shapes on a black sheet, which drop straight into MKW's white badges (drawn in the badge-letter gray). Color comes from the game, not the texture, so no recoloring is needed.

## 1. Extracting the Game Data

For a fresh dump from Eglantine:

1. Open Eglantine, right-click **Mario Kart World** in the game list.
2. Extract RomFS (Eden-family equivalent of Yuzu's Dump RomFS / Ryujinx's Extract Data).
3. Navigate to `romfs/Font/` in the dumped folder.

Copy out `PicFont_00.Nin_Ounce_NVN2.bffnt.zs`. The face-button glyphs live in this file; the 218 `.blarc` archives in `romfs/Layout/` contain no ABXY textures (`Layout/BtnIconFooter_00` is only the shared footer frame).

## 2. Decoding the Glyph Sheet (headless, no GUI needed)

Wine exposes no AT-SPI, so Toolbox's GUI texture preview cannot be driven. The working path is `TextureData.GetImageData` (pure-CPU Ounce deswizzle, no OpenGL context required). `SaveBitMap`/`GetBitmap` fail headless (they need a GL context).

With the bottle `csc.exe`, compile a small exporter against Toolbox's own DLLs (`FirstPlugin.Plg.dll`, `Toolbox.Library.dll`, `Syroot.NintenTools.NSW.Bntx.dll`):

- Slice the BNTX out of the BFFNT (file offset `0x1000`, length `0x402068 - 0x1000`), load it with `BNTX.Load`.
- Call `texture.GetImageData(0, 0, 0)` for the 4 MiB linear RGBA sheet.
- The output rows are bottom-up (OpenGL origin): flip vertically for viewing with Pillow.

The glyph row reads **A B X Y L R ZL ZR SL** in display orientation (105 px pitch, glyph band y 0-88, baseline bars at y 106-112 stay put).

## 3. Replacing the Glyphs

Keep the MKW badge discs and baselines; only the letter art changes:

1. **Export** the KFL PlayStation sheet the same way (`Ext-Operation.bffnt`, BNTX at `0x1000`, BC4 format — decode the blocks to linear first) and take the outline ○ ✕ △ □ set, dropping each cell's outer ring.
2. In the MKW sheet, erase each letter (opaque pixels darker than the badge white become badge white, keeping alpha).
3. Draw each PlayStation shape centered on its cell in the badge-letter gray, scaled to the original letter cap height.
4. **Verify** the row reads ○ ✕ △ □ L R ZL ZR SL before proceeding.

## 4. Saving and Compressing

1. Re-swizzle the modified linear sheet with Toolbox's own inverse: `TegraX1Swizzle.swizzle(1024, 1024, 1, 1, 1, 1, 0, 4, tileMode: 0, blockHeightLog2: 4, data, size, isOrin: true)`.
2. Patch it back into the decompressed BFFNT at file offset `0x2000` (verified byte-identical to the swizzled texture data).
3. Recompress with plain zstd framing to match the original:

```sh
zstd -f PicFont_swapped.bffnt -o PicFont_00.Nin_Ounce_NVN2.bffnt.zs
zstd -d -c PicFont_00.Nin_Ounce_NVN2.bffnt.zs | cmp - PicFont_swapped.bffnt  # must print nothing (clean round-trip)
```

## 5. Mod Deployment (Eglantine)

Title ID: `0400C3F00006E000`

Place the modified file under:

```
Simple PlayStation Icons/romfs/Font/PicFont_00.Nin_Ounce_NVN2.bffnt.zs
```

Full emulator path:

```
load/0400C3F00006E000/Simple PlayStation Icons/romfs/Font/PicFont_00.Nin_Ounce_NVN2.bffnt.zs
```
