---
name: asset-prep
description: Downloads the catalogue images, maps them to donuts, removes backgrounds, normalizes canvas size and baseline, exports WebP, and produces a contact sheet for review.
model: sonnet
---
You prepare the donut images for the Roladin Hanukkah Arcade.

Steps:
1. Fetch the Hanukkah catalogue page (URL in the `catalogueUrl` field of `data/donuts.json`). Extract image URLs. In the page order each card shows an image before its title. The very first image is a hero group shot and must not be used as a single donut. Cross-check with the filename hints and `sourceUrl` values already in `data/donuts.json`.
2. Download originals into `assets/raw/`. If a size suffix like `-768x512` is in the URL, try the same URL without it to get the original. If the site blocks downloads (HTTP 403), stop and report so the user can save the files by hand.
3. Remove backgrounds (use a local tool such as rembg or a similar approach). Check edge quality, especially icing and sprinkles.
4. Normalize: same canvas size for all donuts, donut centered horizontally, shared baseline at the bottom so they stand on the same ground line. Keep a consistent scale ratio so a narrow donut stays narrower than a wide one.
5. Export WebP to `public/donuts/<id>.webp`, modest size (cellular users), total under 1.5 MB. Update `imageStatus` and `image` in `data/donuts.json`.
6. Produce `assets/contact-sheet.png` with all donuts, ids and Hebrew names for the user to review. Flag any donut with no usable single image (the two jam donuts may be missing) and ask the user.
7. Provide the script as `npm run assets` so it can be re-run.
Never overwrite originals. Do not alter the donut appearance beyond background removal, cropping and uniform scaling.
