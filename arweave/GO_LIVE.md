# Going live on an ar.io domain

How to publish any static website to Arweave at `https://<name>.ar.io/`, with working link previews and a favicon.
One script does the checking and publishing; you paste your wallet key at a hidden prompt when it publishes, and it is never saved.

**What you need:** a Mac or Linux Terminal · a static site folder · an ArNS name you own · a Solana wallet (e.g. Phantom) · a few dollars of Turbo credits.

---

## 1. One-time setup

1. **Node.js 18 or newer.** Check with `node -v`. On a Mac, install with `brew install node`.
2. **Know which wallet account owns the name.** ArNS names are controlled by a **Solana** wallet (older guides that use an Arweave JWK are out of date). In Phantom, note the account that lists your name under *Domains* at [console.ar.io](https://console.ar.io).
3. **Add Turbo credits to that same account.** console.ar.io → connect that exact account → **Top Up** (card or crypto). A few dollars covers many deploys.

> **Most common failure:** Phantom holds several accounts, and credits belong to one address only. The account that owns the name, the account with the credits, and the account whose key you paste must all be **the same one**. The script prints the address of the key you paste so you can confirm it.

---

## 2. Prepare the site folder

Everything to be published goes in **one folder** (the script calls it `SRC`), and nothing else does.

### Required
- [ ] `index.html` at the root of the folder
- [ ] **All local paths relative**: `assets/logo.png`, never `/assets/logo.png`
- [ ] **Nothing server-side**: no PHP, no API routes, no build step at runtime (build first, publish the output)
- [ ] **Preview image**: a 1200×630 PNG or JPG (under 5 MB), e.g. `og-image.png` at the root
- [ ] **Favicon**: e.g. `favicon.png` (32–64 px square) or `favicon.ico` at the root, or built into the page as a `data:` URI
- [ ] **Head tags** in `index.html`, using the full address (replace `<name>`):

```html
<title>Site Name</title>
<meta name="description" content="One sentence about the site.">
<link rel="canonical" href="https://<name>.ar.io/">
<link rel="icon" type="image/png" href="favicon.png">
<link rel="apple-touch-icon" href="apple-touch-icon.png">  <!-- optional: 180×180, iPhone home screen -->
<meta property="og:type" content="website">
<meta property="og:url" content="https://<name>.ar.io/">
<meta property="og:title" content="Site Name">
<meta property="og:description" content="One sentence about the site.">
<meta property="og:image" content="https://<name>.ar.io/og-image.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:image" content="https://<name>.ar.io/og-image.png">
```

The preview image address **must be absolute** (`https://…`). X, Discord, Telegram etc. can't resolve a relative one.

### Never in the folder
Everything in it becomes public and permanent: **no keys, `.env` files, wallet exports, scripts or notes**. Keep the deploy script *next to* the folder, not inside it. Check that your browser hasn't saved downloads into it.

### Good to know
- **Single-page apps:** hash routes (`#page`) work as-is. Path routes (`/page`) need the `--fallback-file index.html` flag on the deploy command.
- **Fewer outside dependencies:** fonts and scripts from other sites still work, but bundling them in the folder means the site keeps working if those hosts change.
- **Making the preview image from an HTML mock-up** (no design app needed):
  `npx -y playwright@1 install chromium && npx -y playwright@1 screenshot --viewport-size=1200,630 --wait-for-timeout=3000 "file://$PWD/og.html" og-image.png`
- **Favicon from any image** (Mac): `sips -Z 64 logo.png --out favicon.png`

---

## 3. Publish

Save `deploy-arweave.sh` (appendix below) next to your site folder. Set the two values at the top of the script once per project, or pass them each time:

```bash
ARNS_NAME="yourname"                      # <yourname>.ar.io
SRC="$HOME/Projects/my-site/site"         # the folder with index.html
```

Then, from the folder that holds the script:

| Step | Command | What happens |
|---|---|---|
| **1. Check** | `bash deploy-arweave.sh check` | Checks Node, the folder, relative paths, secrets, every head tag, **the preview image (exists, 1200×630, size)** and **the favicon**, and estimates the cost. Optionally paste the key (Enter to skip) to see which wallet address it belongs to. Fix any ✖ and run again. |
| **2. Staging** *(optional)* | `bash deploy-arweave.sh staging` | Asks for the key, publishes to `https://staging.<name>.ar.io/` to test on desktop and phone. |
| **3. Go live** | `bash deploy-arweave.sh deploy` | Asks for the key, publishes and points `https://<name>.ar.io/` at it. After staging, the files are already uploaded, so this costs almost nothing. |
| **4. Verify** | `bash deploy-arweave.sh verify` *(or `verify staging`)* | Run ~5 minutes after publishing. Confirms the live page loads and matches your local copy, **the preview image loads as an image**, and **the favicon loads**. |

**The key prompt:** paste the private key and press Enter. The prompt stays blank while you paste. Phantom: Settings → Manage Accounts → *the account that owns the name* → Show Private Key. Both Phantom's text key and the `[64 numbers]` format are accepted. The key is kept in memory for that one run and handed to the deploy tool; nothing is written to disk.

**Success looks like:**
```
✔ Turbo credits check passed
✔ Folder uploaded: <transaction id> (files cached for future uploads)
✔ ANT record updated
```
The Arweave URL printed with it works immediately. The `.ar.io` name follows within the TTL (5 minutes). If an old version shows, hard-refresh (Cmd+Shift+R).

---

## 4. Updating the site

Edit the folder, then run `check` → `deploy` → `verify` again. Only changed files are uploaded, so small edits cost cents. The script keeps its upload cache in a fixed working folder (`~/Sites/arweave-<name>`); leave it in place, or the next deploy re-uploads everything.

**Changed the preview image?** X and other apps cache a link's preview for days. Give the new image a new file name (e.g. `og-image-2.png`) and update the three tags that point to it, then redeploy.

---

## 5. Troubleshooting

| Problem | Fix |
|---|---|
| `Insufficient Turbo credits` although you bought some | The pasted key is from a different Phantom account than the one with credits. Compare the address the script prints with console.ar.io, switch accounts in Phantom, run again. |
| Name still shows the old site | Wait for the TTL (5 min), hard-refresh, then `verify`. The printed Arweave URL shows the new files right away. |
| Link preview shows no image | Run `verify`. The `og:image` must be an absolute `https://` URL to a 1200×630 image that loads. Then see the cache note in section 4. |
| Missing images or styles on the live site | A path starts with `/`. Make it relative (the `check` step flags these). |
| `check` lists extra files | Move them out of the folder (often a downloaded script or a `.env`). |
| Paths like `/about` give 404 | Single-page app with path routing: add `--fallback-file index.html` to the deploy command in the script. |

**Costs:** first upload ≈ size × the live Turbo rate (see `https://payment.ardrive.io/v1/rates`; roughly $0.05–0.10 per MB at recent rates). Updates pay only for changed files.

**Under the hood:** the script runs the official ar.io tool,
`npx -y -p @ar.io/deploy ario-deploy deploy --use-arns --arns-name <name> --deploy-folder ./site --sig-type solana --ttl-seconds 300`
with the key in the `DEPLOY_KEY` (pays for the upload) and `ARNS_KEY` (updates the name) environment variables for that one command. Other flags: `--undername staging`, `--fallback-file index.html`, `--no-dedupe` (force a full re-upload).

---

## Appendix: deploy-arweave.sh

Save this as `deploy-arweave.sh` next to your site folder (not inside it). If pasting into Terminal is unreliable, copy it into a text editor and save the file from there.

<details>
<summary>Show the script</summary>

```bash
#!/bin/bash
# deploy-arweave.sh — publish a static site folder to an ar.io name (ArNS) with @ar.io/deploy (Turbo upload, Solana wallet)
#
#   bash deploy-arweave.sh check             # pre-flight: tools, files, preview image, favicon, share tags, cost
#   bash deploy-arweave.sh staging           # publish to https://staging.<name>.ar.io/
#   bash deploy-arweave.sh deploy            # publish to https://<name>.ar.io/
#   bash deploy-arweave.sh verify [staging]  # after publishing: page, preview image and favicon load from the live address
#
# Settings: edit the two lines below once per project, or pass them in:
#   ARNS_NAME=myname SRC=~/myproject/site bash deploy-arweave.sh check
# The wallet private key is never saved: it is asked for at a hidden prompt on each publish, kept in memory,
# handed to the deploy tool for that run only, and gone when the script ends.
set -euo pipefail

ARNS_NAME="${ARNS_NAME:-CHANGE-ME}"          # the ArNS name you own: <name>.ar.io
SRC="${SRC:-$PWD/site}"                      # the folder to publish, with index.html at its root
ARW="${ARW:-$HOME/Sites/arweave-$ARNS_NAME}" # stable working folder: keeps the upload cache (no keys)
TTL="${TTL:-300}"                            # seconds gateways cache the name
MODE="${1:-check}"
SITE_URL="https://$ARNS_NAME.ar.io"
EXCLUDES=(--exclude .git --exclude .DS_Store --exclude Thumbs.db --exclude node_modules --exclude .ario-deploy)

case "$MODE" in check|staging|deploy|verify) ;; *) echo "usage: bash deploy-arweave.sh check|staging|deploy|verify [staging]"; exit 1 ;; esac
[ "$ARNS_NAME" = CHANGE-ME ] && { echo "Set ARNS_NAME first (edit the script or run: ARNS_NAME=yourname bash deploy-arweave.sh $MODE)"; exit 1; }

ok=0; bad=0; warn=0
pass() { printf '  \342\234\224 %s\n' "$1"; ok=$((ok + 1)); }
fail() { printf '  \342\234\226 %s\n' "$1"; bad=$((bad + 1)); }
note() { printf '  ! %s\n' "$1"; warn=$((warn + 1)); }

# reads <title>, <meta> and <link> tags from an HTML file (any attribute order) -> "key<TAB>value" lines
head_tags() {
  HTMLFILE="$1" node -e '
    const h = require("fs").readFileSync(process.env.HTMLFILE, "utf8");
    const attr = (t, n) => { const m = t.match(new RegExp("\\s" + n + "\\s*=\\s*(\"([^\"]*)\"|\x27([^\x27]*)\x27)", "i")); return m ? (m[2] !== undefined ? m[2] : m[3]) : null; };
    const out = [];
    const title = h.match(/<title[^>]*>([^<]*)<\/title>/i); if (title) out.push(["title", title[1].trim()]);
    for (const t of h.match(/<meta\b[^>]*>/gi) || []) { const k = attr(t, "property") || attr(t, "name"); const v = attr(t, "content"); if (k && v !== null) out.push([k.toLowerCase(), v]); }
    for (const t of h.match(/<link\b[^>]*>/gi) || []) { const r = (attr(t, "rel") || "").toLowerCase(); const v = attr(t, "href"); if (v === null) continue;
      if (r === "canonical") out.push(["canonical", v]); else if (r === "icon" || r === "shortcut icon") out.push(["icon", v]); else if (r === "apple-touch-icon") out.push(["apple-touch-icon", v]); }
    for (const [k, v] of out) console.log(k + "\t" + v.replace(/[\t\n]/g, " "));'
}
tag() { printf '%s\n' "$TAGS" | awk -F'\t' -v k="$1" '$1 == k { print substr($0, length(k) + 2); exit }'; }
# width and height of a PNG / JPEG / GIF / WebP file
img_size() {
  IMG="$1" node -e '
    const b = require("fs").readFileSync(process.env.IMG); let w = 0, h = 0;
    if (b.slice(0, 8).toString("hex") === "89504e470d0a1a0a") { w = b.readUInt32BE(16); h = b.readUInt32BE(20); }
    else if (b.slice(0, 3).toString("ascii") === "GIF") { w = b.readUInt16LE(6); h = b.readUInt16LE(8); }
    else if (b.slice(0, 4).toString("ascii") === "RIFF" && b.slice(8, 12).toString("ascii") === "WEBP") { const t = b.slice(12, 16).toString("ascii");
      if (t === "VP8 ") { w = b.readUInt16LE(26) & 0x3fff; h = b.readUInt16LE(28) & 0x3fff; }
      else if (t === "VP8L") { const v = b.readUInt32LE(21); w = (v & 0x3fff) + 1; h = ((v >> 14) & 0x3fff) + 1; }
      else if (t === "VP8X") { w = 1 + b.readUIntLE(24, 3); h = 1 + b.readUIntLE(27, 3); } }
    else if (b[0] === 0xff && b[1] === 0xd8) { let i = 2; while (i + 9 < b.length) { if (b[i] !== 0xff) { i++; continue; } const m = b[i + 1];
      if (m >= 0xc0 && m <= 0xcf && m !== 0xc4 && m !== 0xc8 && m !== 0xcc) { h = b.readUInt16BE(i + 5); w = b.readUInt16BE(i + 7); break; } i += 2 + b.readUInt16BE(i + 2); } }
    console.log(w + " " + h);' 2>/dev/null || echo "0 0"
}
# a same-site URL or relative path -> the file inside SRC ("" if it points elsewhere)
local_file() {
  local u="${1%%[?#]*}"
  case "$u" in
    "$SITE_URL"/*) u="${u#"$SITE_URL"/}" ;;
    https://*|http://*|//*|data:*) echo ""; return ;;
    ./*) u="${u#./}" ;;
    /*) u="${u#/}" ;;
  esac
  [ -z "$u" ] && u="index.html"
  echo "$SRC/$u"
}

# =============================================================== verify (after publishing)
if [ "$MODE" = verify ]; then
  BASE="$SITE_URL"; [ "${2:-}" = staging ] && BASE="https://staging.$ARNS_NAME.ar.io"
  BASE="${VERIFY_BASE:-$BASE}"
  echo "== verify $BASE/ =="
  TMP=$(mktemp -d "${TMPDIR:-/tmp}/arv.XXXXXX")
  CODE=$(curl -s -L --max-time 30 -o "$TMP/page.html" -w '%{http_code}' "$BASE/" 2>/dev/null || true)
  if [ "$CODE" = 200 ]; then pass "page loads (HTTP 200)"; else fail "page returned HTTP $CODE (just published? names update within ~$TTL s)"; fi
  if [ "$CODE" = 200 ]; then
    if [ -s "$SRC/index.html" ] && cmp -s "$TMP/page.html" "$SRC/index.html"; then pass "live page matches your local index.html"; else note "live page differs from your local index.html (still updating? wait a few minutes, then verify again)"; fi
    TAGS=$(head_tags "$TMP/page.html")
    IMGURL=$(tag og:image)
    # on staging, the tags still name the main address: check the staging copy of the same file
    case "$IMGURL" in "$SITE_URL"/*) if [ "$BASE" != "$SITE_URL" ]; then IMGURL="$BASE/${IMGURL#"$SITE_URL"/}"; fi ;; esac
    if [ -n "$IMGURL" ]; then
      R=$(curl -s -L --max-time 30 -o "$TMP/og" -w '%{http_code} %{content_type}' "$IMGURL" 2>/dev/null || true)
      case "$R" in "200 image/"*) pass "preview image loads: $IMGURL (${R#* })" ;; *) fail "preview image did not load: $IMGURL ($R)" ;; esac
    else fail "no og:image tag on the live page"; fi
    ICON=$(tag icon)
    case "$ICON" in
      "") R=$(curl -s -L --max-time 20 -o /dev/null -w '%{http_code}' "$BASE/favicon.ico" 2>/dev/null || true); if [ "$R" = 200 ]; then pass "favicon.ico loads"; else fail "no favicon (no icon link, /favicon.ico returned $R)"; fi ;;
      data:*) pass "favicon built into the page" ;;
      *) case "$ICON" in https://*|http://*) IU="$ICON" ;; /*) IU="$BASE$ICON" ;; *) IU="$BASE/${ICON#./}" ;; esac
         R=$(curl -s -L --max-time 20 -o /dev/null -w '%{http_code}' "$IU" 2>/dev/null || true); if [ "$R" = 200 ]; then pass "favicon loads: $IU"; else fail "favicon did not load: $IU (HTTP $R)"; fi ;;
    esac
  fi
  rm -rf "$TMP"
  echo; if [ "$bad" -gt 0 ]; then echo "$bad problem(s)."; exit 1; else echo "live and complete: $BASE/"; fi
  exit 0
fi

echo "== $SRC -> $SITE_URL ($MODE) =="
# =============================================================== tools
if command -v node >/dev/null && [ "$(node -v | sed 's/^v//' | cut -d. -f1)" -ge 18 ]; then pass "Node $(node -v)"; else fail "Node 18+ needed (macOS: brew install node)"; fi
if command -v rsync >/dev/null; then pass "rsync"; else fail "rsync not found"; fi
command -v node >/dev/null || { echo; echo "Install Node first."; exit 1; }

# =============================================================== files
[ -d "$SRC" ] || { fail "site folder not found: $SRC (set SRC)"; echo; exit 1; }
if [ -s "$SRC/index.html" ]; then pass "index.html at the folder root"; else fail "no index.html at the root of $SRC"; fi
SECRETS=$(find "$SRC" -type f \( -name '.env*' -o -name '*.key' -o -name '*.pem' -o -name 'id.json' -o -name '*keypair*' -o -name '*.sh' \) -not -path '*/node_modules/*' -not -path '*/.git/*' | sed "s#^$SRC/##" | tr '\n' ' ')
if [ -n "$SECRETS" ]; then fail "these would be published publicly — move them out of the folder: $SECRETS"; else pass "no keys, env files or scripts in the folder"; fi
ABS=$(grep -rlE '(href|src)="/[^/]' --include='*.html' "$SRC" 2>/dev/null | sed "s#^$SRC/##" | tr '\n' ' ' || true)
if [ -n "$ABS" ]; then fail "absolute local paths (href=\"/...\") in: $ABS — make them relative (assets/x, not /assets/x)"; else pass "all local paths are relative"; fi
if [ -s "$SRC/index.html" ]; then
  TAGS=$(head_tags "$SRC/index.html")
  T=$(tag title); if [ -n "$T" ]; then pass "title: $T"; else note "no <title>"; fi
  # ---- share tags
  V=$(tag canonical); case "$V" in "$SITE_URL"*) pass "canonical: $V" ;; "") fail "missing <link rel=\"canonical\" href=\"$SITE_URL/\">" ;; *) fail "canonical points elsewhere: $V (expected $SITE_URL/...)" ;; esac
  V=$(tag og:url); case "$V" in "$SITE_URL"*) pass "og:url: $V" ;; "") fail "missing <meta property=\"og:url\" content=\"$SITE_URL/\">" ;; *) fail "og:url points elsewhere: $V" ;; esac
  if [ -n "$(tag og:title)" ]; then pass "og:title"; else fail "missing og:title"; fi
  if [ -n "$(tag og:description)" ]; then pass "og:description"; else fail "missing og:description"; fi
  V=$(tag twitter:card); case "$V" in summary_large_image) pass "twitter:card summary_large_image" ;; "") fail "missing <meta name=\"twitter:card\" content=\"summary_large_image\">" ;; *) note "twitter:card is \"$V\" (summary_large_image shows the big preview)" ;; esac
  # ---- preview image
  IMGURL=$(tag og:image)
  if [ -z "$IMGURL" ]; then
    fail "missing og:image (the preview image): <meta property=\"og:image\" content=\"$SITE_URL/og-image.png\">"
  else
    case "$IMGURL" in https://*) ;; *) fail "og:image must be a full https:// address, got: $IMGURL" ;; esac
    F=$(local_file "$IMGURL")
    if [ -z "$F" ]; then
      note "og:image is on another site ($IMGURL); verify checks it after publishing"
    elif [ ! -s "$F" ]; then
      fail "og:image file not in the folder: ${F#"$SRC"/} (expected because og:image is $IMGURL)"
    else
      read -r W H <<< "$(img_size "$F")"
      BYTES=$(wc -c < "$F" | tr -d ' ')
      if [ "$W" = 1200 ] && [ "$H" = 630 ]; then pass "preview image ${F#"$SRC"/}: 1200x630, $((BYTES / 1024)) KB"
      elif [ "$W" -ge 600 ] && awk -v w="$W" -v h="$H" 'BEGIN { r = w / h; exit !(r > 1.85 && r < 1.97) }'; then note "preview image is ${W}x${H} (works; 1200x630 is the standard)"
      else fail "preview image ${F#"$SRC"/} is ${W}x${H} — make it 1200x630 (PNG/JPG/WebP)"; fi
      if [ "$BYTES" -gt 5000000 ]; then fail "preview image is over 5 MB (X won't show it)"; fi
    fi
    TI=$(tag twitter:image); if [ -n "$TI" ]; then pass "twitter:image"; else note "no twitter:image (X falls back to og:image)"; fi
  fi
  # ---- favicon
  ICON=$(tag icon)
  case "$ICON" in
    "") if [ -s "$SRC/favicon.ico" ]; then pass "favicon.ico at the root"; else fail "no favicon: add <link rel=\"icon\" href=\"favicon.png\"> (or a data: URI) or a favicon.ico at the root"; fi ;;
    data:*) pass "favicon built into the page" ;;
    *) F=$(local_file "$ICON"); if [ -z "$F" ]; then note "favicon is on another site: $ICON"; elif [ -s "$F" ]; then pass "favicon: ${F#"$SRC"/}"; else fail "favicon file not in the folder: $ICON"; fi ;;
  esac
  if [ -z "$(tag apple-touch-icon)" ]; then note "no apple-touch-icon (optional: home-screen icon on iPhone)"; fi
fi
# ---- size and cost
BYTES=$(find "$SRC" -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/.ario-deploy/*' -not -name .DS_Store -not -name Thumbs.db -exec cat {} + | wc -c | tr -d ' ')
FILES=$(find "$SRC" -type f -not -path '*/.git/*' -not -path '*/node_modules/*' -not -path '*/.ario-deploy/*' -not -name .DS_Store -not -name Thumbs.db | wc -l | tr -d ' ')
USD_GIB=$(curl -s --max-time 15 https://payment.ardrive.io/v1/rates 2>/dev/null | node -e 'let s="";process.stdin.on("data",d=>s+=d).on("end",()=>{try{console.log(JSON.parse(s).fiat.usd)}catch(e){}})' 2>/dev/null || true)
if [ -n "$USD_GIB" ]; then
  printf '    %s files, %s bytes: about $%s for a first upload ($%s per GiB); later updates only pay for changed files\n' "$FILES" "$BYTES" "$(awk -v b="$BYTES" -v u="$USD_GIB" 'BEGIN { printf "%.4f", b / 1073741824 * u }')" "$USD_GIB"
else
  printf '    %s files, %s bytes\n' "$FILES" "$BYTES"
fi

if [ "$bad" -gt 0 ]; then echo; echo "$bad problem(s) above — fix them, then run the check again."; exit 1; fi

# =============================================================== the key: hidden prompt, memory only
echo
if [ "$MODE" = check ]; then PROMPT="Paste the wallet's private key to see its address (hidden; Enter to skip): "
else PROMPT="Paste the private key of the wallet that owns $ARNS_NAME (hidden), then press Enter: "; fi
RAW=""; IFS= read -r -s -p "$PROMPT" RAW || true; echo
RAW="$(printf '%s' "$RAW" | tr -d '[:space:]')"
KEY58=""
if [ -n "$RAW" ]; then
  OUT=$(KEYVAL="$RAW" node -e '
    const A = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz";
    const dec = (s) => { let n = 0n; for (const ch of s) { const i = A.indexOf(ch); if (i < 0) process.exit(1); n = n * 58n + BigInt(i); }
      const o = []; while (n > 0n) { o.unshift(Number(n % 256n)); n /= 256n; } for (const ch of s) { if (ch === "1") o.unshift(0); else break; } return Uint8Array.from(o); };
    const enc = (b) => { let n = 0n; for (const x of b) n = n * 256n + BigInt(x); let s = ""; while (n > 0n) { s = A[Number(n % 58n)] + s; n /= 58n; } for (const x of b) { if (x === 0) s = "1" + s; else break; } return s; };
    const k = process.env.KEYVAL; let secret;
    if (k.startsWith("[")) { const a = JSON.parse(k); if (!a.every((x) => Number.isInteger(x) && x >= 0 && x < 256)) process.exit(1); secret = Uint8Array.from(a); }
    else secret = dec(k);
    if (secret.length !== 64) process.exit(1);
    console.log(enc(secret)); console.log(enc(secret.slice(32)));' 2>/dev/null || true)
  RAW=""; KEY58=$(printf '%s\n' "$OUT" | sed -n 1p); ADDR=$(printf '%s\n' "$OUT" | sed -n 2p); OUT=""
  if [ -n "$ADDR" ]; then
    echo "  wallet address: $ADDR"
    echo "  in console.ar.io this address must own $ARNS_NAME (Domains) and hold Turbo credits"
  else
    echo "  that is not a Solana private key (expected a base58 string or [64 numbers])"; KEY58=""
    [ "$MODE" = check ] || exit 1
  fi
elif [ "$MODE" != check ]; then
  echo "  no key entered — nothing was published"; exit 1
fi
if [ "$MODE" = check ]; then KEY58=""; echo; echo "pre-flight passed. next: bash deploy-arweave.sh staging   (or: deploy)"; exit 0; fi

# =============================================================== publish
mkdir -p "$ARW/site"
rsync -a --delete "${EXCLUDES[@]}" "$SRC/" "$ARW/site/"
cd "$ARW"
if [ "$MODE" = staging ]; then UNDER=(--undername staging); TARGET="https://staging.$ARNS_NAME.ar.io/"; VER="verify staging"; else UNDER=(); TARGET="$SITE_URL/"; VER="verify"; fi
echo; echo "== publishing $FILES files -> $TARGET =="
DEPLOY_KEY="$KEY58" ARNS_KEY="$KEY58" npx -y -p @ar.io/deploy ario-deploy deploy --use-arns --arns-name "$ARNS_NAME" \
  --deploy-folder ./site --sig-type solana --ttl-seconds "$TTL" ${UNDER[@]+"${UNDER[@]}"}
KEY58=""
echo
echo "Published. The name updates within ~$TTL seconds; then run:  bash deploy-arweave.sh $VER"
echo "The Arweave URL printed above serves the files right away."
```

</details>
