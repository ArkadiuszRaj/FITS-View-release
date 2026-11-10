# FITS View (fits-viewer)

FITS viewer for the files of a station (and any other FITS): 16-bit images and cubes
scaled to the 8 bits of the screen, the planes of a cube played, the header keywords and
the binary tables listed. React + TypeScript in the browser, a small Node.js server
(fastify) on localhost, packaged as a `.deb`, an AppImage and a Windows installer and
zip, each with a bundled Node.js, the same way as `cad-viewer` in CAD-View.

```sh
fits-viewer image.fits              # opens the browser on the file
fits-viewer --app image.fits        # in a window of its own (Edge, Chrome, Chromium; else the browser)
fits-viewer 2026-10-21/             # a folder: its FITS files, with a list to switch between them
fits-viewer                         # opens the browser; files are opened from the page (or dropped on it)
fits-viewer a.fits.gz --no-open     # only the server: open the printed address yourself
```

The server binds to 127.0.0.1 on a free port and stops 15 s after the last page is
closed (with `--no-open`: on Ctrl+C), so the menu entry, `fits-viewer.exe` and "Open
with" on `.fits` files need no terminal. The menu entry (Linux) and `fits-viewer.exe`
(Windows) use `--app`; the window has a browser profile of its own in
`~/.cache/fits-viewer/app-profile` (Windows: `%LOCALAPPDATA%\fits-viewer\app-profile`), one for all the windows.

## Using it

The window has the same zones as the other viewers of the family (GED View's
`docs/belka.md`):

- the **toolbar**, one line that never changes with the file: **File ▾** (**Open…**, **Save
  PNG…**), the view as icons (**Fit** (F), **1:1** (1), zoom in (+), zoom out (−); the key
  is in each tooltip), the name of the file (its full path in the tooltip) and, at the right
  end, **⋯** with the light / dark theme and the language (English / Polski);
- the **track row** under the toolbar, only while the track is shown (see **Track**);
- the **rail** at the right edge: the tabs of the panels, **Header**, **Table** (with the
  number of rows), **Plot**, **Files** (with the file shown, e.g. 2/5), then **Scale**,
  **Profile** and **Track**. One panel at a time; a click on the tab of the panel open
  closes it. The panel chosen and the widths are kept in the browser;
- the **status bar**: the image HDU shown (a click: another image of the file, when it has
  several), the size and type of the image (or of the track), the range, the pixel under
  the cursor and the zoom.

- wheel: zoom at the cursor; a drag with the right button: pan; double click or **Fit**:
  whole image; **1:1**: a pixel of the image per pixel of the screen. Zoomed in, the
  pixels are drawn as squares (no smoothing). The rows are shown as stored, the first
  row of the file at the top; **first row at the bottom** in **Scale** (V) turns them
  over, as FITS defines it (the choice is kept). The status bar shows the pixel under
  the cursor: `x, y` of the screen from the top left corner (0-based), the column and
  row of the file (1-based) and the value (BZERO and BSCALE applied: a 16-bit image
  reads 0–65535).
- **Scale**: the **stretch** (linear, square root, square, log, asinh, histogram
  equalisation), the **range** shown as black and white (min–max, a cut at 99.9 / 99.5 /
  99 / 95 % of the pixels, zscale as in DS9, or by hand), the **colour map** (grey, heat,
  cool, viridis, magma, rainbow), **invert** (I) and the direction of the rows (V); under
  them the histogram of the plane (counts on a log scale) with the two ends of the range
  as handles to drag, the range as numbers, and min, max, mean and standard deviation.
  For a cube the range is that of the plane shown, or of the whole cube (a checkbox), so
  that the planes play without flicker.
- a cube (NAXIS = 3): the bar at the bottom of the image plays its planes (2–25 per
  second) and steps through them (← →, Home, End, space).
- **Header**: the keywords of the HDU, with a filter (key, value or comment), the HDU
  chosen from a list when the file has several; **csv** saves the keywords listed
  (keyword, value, comment).
- **Table**: the binary table (BINTABLE), as the **row** that goes with the plane shown
  (the firmware writes a row per plane: the fields listed down the panel, filtered by
  a regular expression on their names and values, e.g. `^roi_|_az$`) or as the whole
  **grid**; a click on a row of the grid shows its plane, a double click opens the
  fields of the row in a window of its own, with the same filter and previous / next
  row; **csv** saves the whole table (a line per row, the column names first, arrays
  with their elements separated by semicolons). Columns of every FITS type are read
  (L, B, I, J, K, E, D, A, X, arrays, variable-length arrays).
- **Plot**: the columns of the table plotted against the row (the plane), the time from
  the first row (from `observation_jd1` + `observation_jd2`) or another column: every
  numeric column as a sparkline with its range (the constant ones left out unless asked
  for, the filter a regular expression as in **Table**); a click on one plots it in full
  above, with axes, the value under the cursor and the row shown marked; a click on a
  plot shows that row's plane. The columns plotted are kept across files.
- **window…** (in **Plot**): the plots of the table in a big window, the columns ticked on
  the left, the plots sharing the height of the window (one: all of it, two: half each, …;
  scrolled when they are too many); a click on a plot shows that row (plane).
- **compare N files…** (in **Plot**, when several files are opened): the tables of all
  the files in a window, read without their pixels (quick even with the previews in the
  list): the columns ticked on the left are plotted on the right, each with a line per file,
  the file shown thick, the others grey (the one under the cursor raised and named), against
  the row, the time from each file's first row or the time of day (UTC); the plots share the
  height of the window as in **window…**. A click on a line opens its file at that row (plane).
- **Profile** (P): a line on the image, as in ImageJ (**Draw** in the head of the panel:
  the left button draws the line, on when the panel opens; off, it picks and pans again); the values along it are plotted (a
  sample per pixel, interpolated between the pixels, the mean over a width of 1–9 pixels
  across; **csv** saves them). It follows the plane shown (and works on the track).
  - A drag from a star: the star is measured once, in 2-D. Its centre is found as the
    station firmware finds it (`src/shared/starfit.ts`, from ON-Firmware's
    `roi-filter.hpp` and `shape-fitting.cpp` `fit_point`: the centroid of the pixels above
    half maximum around the peak, then a circular Gaussian with a background fitted to
    the pixels by Levenberg–Marquardt, saturated ones left out); its FWHM is of that
    Gaussian held at the centre, its amplitude, width and background fitted within 12 px
    (the firmware's own window, ~4–5 px, holds only the core: its background takes the
    star's wings and the FWHM comes out 10–20 % narrower). The star is circled with its
    FWHM, marked saturated when it has a flat top or two pixels or more at the top of the
    frame, and the line goes through its centre, turned about it by the cursor (the
    distance from the centre: half its length). Along the line, the section of the
    Gaussian is drawn over the profile and the half widths are taken about the same
    centre, before and after it (unequal for a star with coma or a trail).
  - A click and a click: a line between the two points (Esc after the first: given up),
    measured on its own: the background (the lower fifth of the samples), the half
    maximum, a Gaussian fitted to the profile, the FWHM both ways, the half widths about
    the peak, the centre. Interpolation widens an undersampled star (FWHM of 2–3 px) by
    some 10–15 %, and a clipped peak is pointed out.
  - Shift: horizontal, vertical or at 45°; the right button pans.
- **Track** (T: shown or not; the tab: its panel, with **Show** in its head, the ROIs, the
  sensor and the part of it covered): the pass reconstructed from a cube of ROIs: every plane laid at its place
  on the sensor, on one image (the part of the sensor the ROIs cover, the brightest where
  they overlap, transparent where none is), scaled like a plane. The ROI of the plane
  shown is framed; **up to the plane** (in the bar of the planes and in the panel) shows only the planes
  up to it, so that played the track builds up; a click on the track shows the plane of
  the ROI there; the status bar gives the sensor pixel. While the track is shown, a row
  under the toolbar holds its options: a **background** (one of the files opened, or a
  file from the disk) lays the ROIs on a whole frame of the sensor (a preview, `PRV-*`); it
  must be the size of the sensor and is kept while the files are switched. Its rows are
  taken as stored the way the planes are; the button next to the list turns them over;
  **✕ close** goes back to the plane. The places come from the table (a row per plane): the
  corner of a cutout is `position − center_of_mass` (`roi_location_x/y` is that corner in
  the TL files but the centroid in the newer AST ones, so it is only the fallback), the
  size `roi_size_x/y`, the sensor `camera_sensor_width/height`; whether the planes are
  stored turned over (the firmware) or not (TL) is found from where their light is.
- **Files**: the files of the command line or dropped onto the page (PageUp / PageDown
  switch between them).
- **File ▾ → Save PNG…**: the plane (or the track) as shown, 8-bit, with the stretch and
  colour map.
- the panels on the right are resized by dragging their left edge (a double click on it:
  the default width); **⋯** switches between the dark and the light theme (the system's at
  first) and between English and Polish (the browser's language at first: Polish when it is
  Polish, else English). The panel, widths, theme, language and orientation are kept in the
  browser. The keywords of the header, the names of the columns, the values and the messages
  of the FITS reader stay as they are.

Files read: plain FITS, gzipped files (`.fits.gz`, inflated in the browser), and
tile-compressed images (`.fits.fz` of fpack / cfitsio with RICE_1, GZIP_1 or GZIP_2,
integer pixels), i.e. everything the station firmware writes: 8- and 16-bit images and
cubes with `BZERO = 32768`, a BINTABLE of doubles with a row per plane, and the RICE
previews. Also 32-bit integers and 32/64-bit floats (uncompressed). Not read:
compressed floating-point images, ASCII tables, random groups, images of 1 or more
than 3 dimensions: their headers are still listed.

## How it works

| step | where |
|------|-------|
| file → HDUs | `src/shared/fits.ts`: header cards, images (big endian to typed arrays), binary tables, tile-compressed images; in a Web Worker (`src/client/parse.worker.ts`), gzip through `DecompressionStream` |
| RICE_1 | `src/shared/rice.ts`: the decoder of cfitsio (`ricecomp.c`) for 1, 2 and 4 bytes per pixel |
| plane → screen | `src/shared/scale.ts`: statistics (a sorted sample for the percentiles, a histogram for the panel), zscale, the stretch as a 4096-step table, colour maps, RGBA |
| the pass | `src/shared/track.ts`: the places of the ROIs from the table, the planes laid on the sensor |
| plots | `src/client/Plot.tsx`: sparklines and plots of the columns, on a canvas |
| compare, profile | `src/client/Compare.tsx`, `src/client/Profile.tsx` on `src/client/Chart.tsx` (lines on a canvas); `src/shared/profile.ts`: the samples along a line, the half-maximum width, a Levenberg–Marquardt Gaussian |
| screen | `src/client/ImageView.tsx`: the plane on an offscreen canvas, drawn with the view's transform |
| words | `src/client/i18n.ts`: the strings of the page in English and Polish, as in GED View |

### What is in `release`

`<version>` comes from `git describe`: `1.0` built at the tag `v1.0`, `1.0+4.gf3f7e03` four
commits after it (`.dirty` with uncommitted changes); without a tag, `package.json` plus the
commit. Every package carries its own Node.js, so none of them needs Node.js installed.

| file | for | how it is used |
|------|-----|----------------|
| `fits-viewer_<version>_amd64.deb` | Debian, Ubuntu and the like (x86-64): installed for all users; a newer `.deb` installed the same way upgrades it | `sudo apt install ./fits-viewer_<version>_amd64.deb`; then `fits-viewer` on the command line, the menu entry, "Open with" on FITS files |
| `fits-viewer-<version>-x86_64.AppImage` | any Linux (x86-64) without installing: one file to copy, e.g. to a station or a laptop without root | `chmod +x` and run it, with files or a folder as arguments |
| `fits-viewer-<version>-win64-setup.exe` | Windows 10/11 x64: installed per user, no administrator rights | run it; Start menu entry, optional desktop shortcut, "Open with" on `.fits`, `.fit`, `.fts`; uninstalled from Windows' settings |
| `fits-viewer-<version>-win64.zip` | Windows, portable: nothing installed (a USB stick, a locked-down PC) | unzip anywhere, run `fits-viewer.exe` in the unzipped folder (or `bin\fits-viewer.cmd`, with a console) |
| `*latest*` (e.g. `fits-viewer_latest_amd64.deb`) | a fixed name for scripts, links and downloads | a relative symbolic link to the package of that kind built last |

The details of each:

- `.deb`: `sudo apt install ./fits-viewer_<version>_amd64.deb` installs into
  `/opt/fits-viewer`, links `fits-viewer` into `/usr/bin` and registers the menu entry
  for FITS files. Needs glibc 2.28 (the bundled Node.js 22) and `xdg-utils`;
  Chromium, Chrome or Edge recommended for the app window.
- AppImage: `chmod +x fits-viewer-*.AppImage; ./fits-viewer-*.AppImage [--app] image.fits`.
  Without FUSE: `--appimage-extract-and-run` as the first argument or
  `APPIMAGE_EXTRACT_AND_RUN=1`.
- Windows 10/11 x64 (`packaging/windows`): `fits-viewer.exe` is a small launcher
  (C, MinGW-w64, static: only the DLLs of Windows) that runs the bundled `node.exe`
  with `--app` in an Edge window, without a console, the log in
  `%LOCALAPPDATA%\fits-viewer\fits-viewer.log`; `bin\fits-viewer.cmd` is the same with a
  console. The installer (NSIS) installs per user without administrator rights into
  `%LOCALAPPDATA%\Programs\fits-viewer`, adds the Start menu entry, optionally a desktop
  shortcut and FITS View in "Open with" of `.fits`, `.fit` and `.fts` files (the default
  program of these files is not changed); the zip is the portable version. See
  `packaging/windows/README-Windows.txt` (shipped in both).

The macOS package is built here, on Linux, from the Node.js of nodejs.org for macOS (the
modules are pure JS: nothing of Apple is needed); its `install.sh` puts it in `/opt/fits-viewer`
(or `~/.local/opt/fits-viewer` with `--user`) and `fits-viewer` on PATH; nothing is signed by Apple:
packaging/macos/README-macOS.txt.

## Licence

Copyright © 2026 Arkadiusz Raj (hijax_pl). MIT (`LICENSE`).
