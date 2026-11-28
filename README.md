# Lu Ambriz

The website of Lu Ambriz, Mexico City artist: UTXO art on Bitcoin, Ordinals, ACME works and physical pieces paired with NFTs. English and Spanish.

- **Website:** https://luambriz.ar.io/
- **Direct (GitHub Pages):** https://luamb26.github.io/luambriz/

## How it is served

- This repository is the whole site. GitHub Pages serves it from the `main` branch (root folder).
- `index.html` is the full single-page site. `img/`, `vid/` and `fonts/` hold its media and fonts, and `js/three.module.min.js` is Three.js r160 for the 3D hero.
- Everything loads from this repository, except two live lookups: the latest Bitcoin block height from mempool.space, or blockstream.info as backup (header chip, confirmations) and open dispensers from the Counterparty API (Collect, work pages, Home). If either can't load, the page keeps the snapshot from October 25, 2026.
- `work/`, `story/`, `physical/`, `onchain/`, `collect/` and `es/` (with the same tabs inside) are entry pages for search engines: the same page, opening on that tab (in Spanish under `es/`), each with its own title and description. `sitemap.xml` lists them; to have Google list them sooner, add https://luamb26.github.io/luambriz/ in Google Search Console and submit the sitemap (optional).
- Spanish links: https://luambriz.ar.io/#es opens the site in Spanish, and so does any section with `es.` in front (`#es.work`, `#es.w-s17584`). Visitors whose browser is set to Spanish are offered Spanish once.
- `404.html` is the page-not-found page; it forwards short addresses such as `/luambriz/w-s17584` to that work.
- `img/w160`, `img/w360`, `img/w720` and `img/w1280` hold smaller WebP copies of large photos; the page picks one by screen size and shows the original in the viewer.
- https://luambriz.ar.io/ is an ArNS name on Arweave. It points to a small frame page (`arweave/site/`) that shows this GitHub Pages site full-window. Updates pushed here appear there with no re-publishing.
- `arweave/deploy-arweave.sh` publishes the frame page, following `arweave/GO_LIVE.md`. Re-publish only when files in `arweave/site/` change.

## Credits

Artwork, photographs and text © Lu Ambriz. All rights reserved.
Three.js © three.js authors (MIT). Fonts: Geist and Geist Mono, Anton (SIL Open Font License 1.1).
