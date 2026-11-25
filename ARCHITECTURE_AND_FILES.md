# Architecture & Complete File Inventory — `Qattous SM`

**Project Directory:** `C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP`

---

## 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     LOCAL ADMIN PC (Windows)                            │
│  5 Excel Files (.xlsx) ──► db_engine.py (calamine) ──► qattous_warehouse.db
│                                                            │            │
│  http://localhost:8900 ◄── server.py (REST API + UI) ──────┘            │
│                                │                                        │
│                         cloud_sync.py (JSON builder + git push)         │
└────────────────────────────────┼────────────────────────────────────────┘
                                 │ git push origin main
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                GITHUB REPO (ahmadalfasfous89-hue/qattous-wh-app)        │
│  • index.html + manifest.json + xlsx.full.min.js + qattous_logo.png     │
│  • data_bundle.json (~6.2 MB raw / ~600 KB gzip)                        │
│  • users_bundle.json (~2.4 KB)                                          │
│  • version.json + Qattous_Stock_Management_App.apk                      │
└────────────────────────────────┼────────────────────────────────────────┘
                                 │ HTTPS (4G / Wi-Fi)
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│              CLIENTS (Android APK / PDA / Mobile / Web Browser)         │
│  • Loads UI from GitHub Pages (LOAD_NO_CACHE)                           │
│  • Fetches data_bundle.json, users_bundle.json, version.json directly   │
│    from raw.githubusercontent.com for ~2s instant propagation           │
│  • AndroidBridge (@JavascriptInterface) saves .xlsx to Android Downloads│
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Complete File Inventory

### 2.1 Core Application Files (Active)

| File | Size (approx) | Purpose | Tracked on GitHub? |
| :--- | :--- | :--- | :---: |
| `index.html` | ~177 KB | Complete single-file frontend (Desktop + Mobile/PDA dual UI, Login Gate, Scanner, Camera, Reports, Data Sync Drag & Drop, Users Manager, Update Checker, Client-side Cloud Search Engine). | ✅ Yes |
| `server.py` | ~33 KB | Local Python `ThreadingHTTPServer` on port `8900`. Serves UI, static files, and all `/api/*` endpoints. Automatically triggers `cloud_sync` + background `git push` when users or sheets are saved. | Local PC |
| `db_engine.py` | ~63 KB | Fast Excel indexer (`python-calamine`), SQLite manager, lookup engine (`lookup_code`), Series Distribution & Store-to-Store transfer calculator, and Excel report generator (`openpyxl`). | Local PC |
| `cloud_sync.py` | ~9.4 KB | Exports active items (~5,384+ items across Bins, Location Inv, and Transfers) into compact `data_bundle.json` and `users_bundle.json`, and runs `git add/commit/push`. | Local PC |
| `build_apk.py` | ~16 KB | Standalone native Android APK builder (no Gradle required). Generates `AndroidManifest.xml`, `MainActivity.java` (WebView + Camera + FileChooser + `AndroidBridge`), launcher icons from `qattous_logo.png`, compiles via `aapt2`/`javac`/`d8`, aligns with `zipalign`, and signs with `apksigner`. | Local PC |
| `manifest.json` | ~0.3 KB | PWA Web App Manifest (`name: "Qattous Stock Management App"`, `short_name: "Qattous SM"`). | ✅ Yes |
| `version.json` | ~0.2 KB | Remote APK version manifest checked by the in-app "Check App Update" button. | ✅ Yes |
| `xlsx.full.min.js` | ~930 KB | Local copy of SheetJS `v0.20.3` for 100% reliable `.xlsx` export & browser Excel parsing with zero third-party CDN dependency. | ✅ Yes |
| `qattous_logo.png` | ~19 KB | Official Qattous Group logo used in header, login gate, favicon, and APK launcher icon. | ✅ Yes |

### 2.2 Generated Data, Database & Signing Files

| File | Purpose | Notes |
| :--- | :--- | :--- |
| `qattous_warehouse.db` | Local SQLite database (~71 MB) containing all 5 indexed Excel sheets, `sheet_meta`, and `users`. | **Local only** (gitignored) |
| `data_bundle.json` | Compact 4G cloud JSON bundle (~6.2 MB) of active inventory, bins, and transfers. | ✅ Pushed to GitHub |
| `users_bundle.json` | Synced user accounts & permission matrices for 4G cloud login. | ✅ Pushed to GitHub |
| `qattous_release.jks` | Release Android signing keystore (alias: `qattous`). **KEEP SAFE & BACKED UP** — required to sign future APK updates seamlessly over existing installs. | **Local only** (never commit) |

### 2.3 Launchers, APK Builds & QR Code

| File | Purpose |
| :--- | :--- |
| `Run_Qattous_App.bat` | 1-click launcher to start `server.py` and open `http://localhost:8900`. |
| `Update_Cloud_GitHub.bat` | 1-click script to re-index updated Excel files, rebuild JSON bundles, and push to GitHub. |
| `Qattous_Stock_Management_App.apk` | **Current Production Signed APK (`v2.0.0`, `24.8 KB`)** — pushed to GitHub. |
| `Qattous_SM_APK_QR.png` | Printable/shareable QR code pointing directly to `Qattous_Stock_Management_App.apk` on GitHub Pages. |
| `Brands_Stock_Stores_WH.apk` / `Qattous_Warehouse_v1.apk` | Legacy APK builds from earlier iterations (safe to archive/delete locally). |
| `android_apk_build/` | Temporary build output folder generated by `build_apk.py`. |

### 2.4 Source Excel Files (Local PC)
- `Bin Contents (10).xlsx` *(latest)* / `Bin Contents (8).xlsx`
- `Item Variant Inv. By Location (15).xlsx` *(latest)* / `Item Variant Inv. By Location (5).xlsx`
- `Al Najma Transfer Line (9).xlsx`
- `Retail Item List (47).xlsx`
- `AL Najma Item Variant Registration (2).xlsx`

---

## 3. SQLite Database Schema (`qattous_warehouse.db`)

| Table | Key Columns | Indexes |
| :--- | :--- | :--- |
| `bin_contents` | `bin_code`, `item_no`, `barcode`, `variant`, `description`, `qty`, `avail_qty`, `uom` | `barcode`, `item_no`, `bin_code` |
| `location_inv` | `item_no`, `barcode`, `variant`, `variant_desc`, `description`, `location`, `qty` | `barcode`, `item_no`, `location` |
| `transfer_lines` | `doc_no`, `line_no`, `item_no`, `barcode`, `variant`, `qty`, `from_code`, `to_code`, `status`, `ship_date` | `barcode`, `item_no`, `doc_no` |
| `retail_items` | `item_no`, `description`, `vendor_item_no`, `model`, `section`, `sub_section`, `season_code`, `brand_name` | `item_no` |
| `variant_reg` | `item_no`, `barcode`, `color_size`, `dim2_size` | `barcode`, `item_no` |
| `users` | `username`, `password`, `display_name`, `role`, `permissions_json`, `active` | `username` (PK) |
| `sheet_meta` | `sheet_key`, `file_name`, `row_count`, `last_updated` | `sheet_key` (PK) |

---

## 4. Local Server REST API (`server.py` on port `8900`)

| Method & Route | Purpose |
| :--- | :--- |
| `GET /` | Serves `index.html` (no-cache headers). |
| `GET /xlsx.full.min.js`, `/version.json`, `/manifest.json`, `/qattous_logo.png`, `/data_bundle.json`, `/users_bundle.json` | Serves static assets & cloud bundles. |
| `GET /api/meta` | Returns loaded Excel sheet statuses, row counts, timestamps, and store definitions. |
| `GET /api/scan?code=...` | Smart lookup by Barcode, Item No (`I...`), Bin Code, or Transfer Order (`TO Number`). |
| `GET /api/suggest?q=...` | Fast autocomplete suggestions for Bins, Items, and Barcodes. |
| `GET /api/report?scope_type=...&scope_value=...&dist_mode=...&export=0\|1` | Runs Series Distribution report (JSON or downloadable `.xlsx`). |
| `POST /api/login` | Authenticates username/password against SQLite `users` table. |
| `GET /api/users` | Lists all user accounts and permissions. |
| `POST /api/users/save` | Creates or updates a user + triggers automatic `cloud_sync` & background `git push`. |
| `POST /api/users/delete` | Deletes a non-admin user + triggers automatic `cloud_sync` & background `git push`. |
| `POST /api/sheets/update` | Selectively re-indexes chosen Excel sheets from disk (and optionally pushes to GitHub). |
| `POST /api/sheets/upload` | Accepts Drag & Drop uploaded `.xlsx` files from the browser, saves them, re-indexes SQLite, and pushes to GitHub. |
| `POST /api/cloud/push` | Rebuilds `data_bundle.json` & `users_bundle.json` and pushes to GitHub. |

---

## 5. Cloud Bundle Structure (`data_bundle.json`)

Compact key-value schema designed to minimize 4G bandwidth:
- `updated_at`: Timestamp string (`YYYY-MM-DD HH:MM:SS`)
- `items_meta`: `{ item_no: [description, brand, section, vendor_item_no, season_code] }`
- `reg_by_bc`: `{ barcode: [item_no, color_size] }`
- `variants_by_item`: `{ item_no: [barcode1, barcode2, ...] }`
- `inv_by_bc`: `{ barcode: { location_code: qty, ... } }` *(pure On-Hand; does NOT include transfer lines)*
- `bins_by_bc`: `{ barcode: [[bin_code, qty, avail_qty], ...] }`
- `bins_by_code`: `{ bin_code: [[barcode, item_no, qty], ...] }`
- `tr_by_bc`: `{ barcode: [[doc_no, from_code, to_code, qty, status, ship_date], ...] }`
- `tr_by_doc`: `{ doc_no: [[barcode, item_no, from_code, to_code, qty, status, ship_date], ...] }`

---

## 6. Android APK Native Bridge (`MainActivity.java`)

Built via `build_apk.py` targeting Android API 23–34 (`com.qattousgroup.warehouse`):
- Loads `https://ahmadalfasfous89-hue.github.io/qattous-wh-app/?apk_ver=2.0.0&v=<timestamp>` with `LOAD_NO_CACHE` and `clearCache(true)`.
- Requests `CAMERA` permission for barcode camera scanning and grants `WebChromeClient.onPermissionRequest`.
- Supports HTML `<input type="file">` via `onShowFileChooser`.
- Exposes `window.AndroidBridge` (`@JavascriptInterface`):
  - `saveBase64File(base64Data, fileName, mimeType)`: Saves exported `.xlsx` / `.csv` files directly into Android's `Downloads` folder via `MediaStore.Downloads` (Android 10+) or `Environment.DIRECTORY_DOWNLOADS` (older Android).
  - `openExternalUrl(url)`: Opens APK update links in the system browser/installer.
