# Qattous Stock Management App (`Qattous SM`) — Master Project Handover

**Last Updated:** 2026-11-25  
**Current App Version:** `v2.0.0` (APK `versionCode: 2`)  
**Owner / Organization:** Qattous Group — La Familia & Warehouse (`W0001`)  
**Workspace Path:** `C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP`

---

## 1. Executive Summary

**Qattous Stock Management App** (displayed internally as **`Qattous SM`**) is a **100% free**, bilingual (**Arabic / English**) inventory lookup, warehouse bin tracking, transfer-order monitoring, and series-distribution system built for **Qattous Group**.

It operates seamlessly across two synchronized channels:
1. **Local Admin PC Server (`http://localhost:8900`)**: Powered by Python + SQLite (`qattous_warehouse.db`), used by Admin/Supervisors to drag-and-drop updated Excel sheets, manage users/permissions, run full distribution reports, and push updates to the cloud in one click.
2. **4G / Wi-Fi Cloud Web App & Native Android APK (`Qattous_Stock_Management_App.apk`)**: Hosted for free on **GitHub Pages**, working on any smartphone, Android PDA (hardware laser scanner), or browser over 4G or Wi-Fi.

---

## 2. Live URLs & Distribution Links

| Resource | URL / Path |
| :--- | :--- |
| **Live 4G Web App** | https://ahmadalfasfous89-hue.github.io/qattous-wh-app/ |
| **Direct APK Download (v2.0.0)** | https://ahmadalfasfous89-hue.github.io/qattous-wh-app/Qattous_Stock_Management_App.apk |
| **GitHub Repository** | https://github.com/ahmadalfasfous89-hue/qattous-wh-app (`main` branch) |
| **Raw Instant Sync Endpoint** | `https://raw.githubusercontent.com/ahmadalfasfous89-hue/qattous-wh-app/main/` |
| **Local PC Admin Panel** | `http://localhost:8900` |
| **Printable APK QR Code** | `Qattous_SM_APK_QR.png` |

---

## 3. Official Brand Identity

Extracted from `https://qattousgroup.com/Aboutus`:
- **Logo:** `qattous_logo.png`
- **Primary Bronze:** `#906f49`
- **Dark Bronze:** `#745736`
- **Light Warm Background:** `#f7f5f2` / `#f5efe8`
- **Charcoal Text:** `#3d3d3d`
- **Onyx Dark:** `#151515`
- **External App Name (Android Launcher / Browser Title):** `Qattous Stock Management App`
- **Internal Display Name (Header / Login):** `Qattous SM`

---

## 4. Core Business Rules & Data Logic

### 4.1 Five Source Excel Files
1. **`Bin Contents (*.xlsx)`** — Warehouse bin locations (`W0001` bins, e.g. `B-21-...`), quantities per barcode per bin. *(Updated daily/frequently)*
2. **`Item Variant Inv. By Location (*.xlsx)`** — On-Hand inventory per barcode across all stores (`S0001`–`S0012`), Main Warehouse (`W0001`), and `IN-TRANSIT`. *(Updated daily/frequently)*
3. **`Al Najma Transfer Line (*.xlsx)`** — Transfer Orders (`TO Number` / `Document No.`), `From Location`, `To Store`, `Quantity`, `Status` (`Released`, etc.), `Shipment Date`. *(Updated daily/frequently)*
4. **`Retail Item List (*.xlsx)`** — Master catalog (Description, Brand, Section/DEPT, Vendor Item No / ART NO, Season Code). *(Updated occasionally)*
5. **`AL Najma Item Variant Registration (*.xlsx)`** — Master barcode-to-item & Color/Size mapping. *(Updated occasionally)*

> **Automatic Latest-File Detection:** `db_engine.py` automatically detects the newest version of each Excel file in the folder (e.g., `Bin Contents (10).xlsx` over `Bin Contents (8).xlsx`) by modification time / filename prefix.

### 4.2 Transfer Line (`Al Najma Transfer Line`) Rule — CRITICAL
- Rows where `from_code == 'IN-TRANSIT'` are ignored to avoid double-counting.
- **Released transfers are NOT added to Store On-Hand stock.**
  - Store On-Hand (`oh_stores[code]`) reflects **ONLY actual physical stock** from `Item Variant Inv. By Location`.
  - Incoming `Released` transfer quantities are displayed **separately as an orange indicator badge** next to the store (e.g. `5 (+1 TO)` on Desktop or `5 (+1 🚚)` on Mobile/PDA), even when actual store stock is `0` (`0 (+1 TO)`).
  - `TO Number` (`Document No.`) is displayed alongside `In-Transit`, expandable in a dedicated Transfers detail panel, and **directly searchable** in the scanner box (scanning/typing a `TO Number` lists all lines of that Transfer Order).

### 4.3 Store Codes & La Familia Priority Tiers

| Code | Store Name | Priority Tier | Base Series Allocation |
| :--- | :--- | :--- | :--- |
| `S0001` | La Familia Avenue Mall | Priority (Top) | 3 series |
| `S0011` | La Familia Mecca Mall | Priority (Top) | 3 series |
| `S0002` | La Familia Khalda | Priority | 2 series |
| `S0003` | La Familia Tabarbour | Priority | 2 series |
| `S0004` | La Familia Marj Al Hamam | Priority | 2 series |
| `S0010` | La Familia Irbid | Priority | 2 series |
| `S0012` | La Familia Bawabat Al Amadina (Zarqa) | Priority | 2 series |
| `S0005` | Istiklal | Standard | — |
| `S0006` | Online | Standard | — |
| `S0007` | Ohana Avenue | Standard | — |
| `S0008` | Ohana Khalda | Standard | — |
| `S0009` | LaVenta Avenue | Standard | — |
| `W0001` | Main Warehouse | Warehouse | Source |
| `IN-TRANSIT` | In-Transit | Transit | Displayed with TO Numbers |

### 4.4 Series Distribution & Eligible Store Engine
- Computes per-color size series plans (`series_units`) and priority allocation (`S0001`, `S0011`, `S0002`, `S0003`, `S0004`, `S0010`, `S0012`).
- Supports `+1 extra series` overflow layer when WH stock exceeds base series needs.
- **Zero-Replenishment Rule:** When total warehouse stock for a color is `< 1 full series`, pieces replenish the highest-priority store that already carries that color (`Availability > 0`) and has `0` on that specific size.
- Supports **Store-to-Store transfer suggestions** when WH stock is `0`.

---

## 5. User Roles & Granular Permissions

All accounts are stored in SQLite (`users` table) and synced to `users_bundle.json`. Authentication is enforced on startup via a full-screen Production Login Gate (`#loginModal`) with `localStorage` session persistence.

| Permission Key | Description | Admin | Supervisor | User (Default) |
| :--- | :--- | :---: | :---: | :---: |
| `mode_wh_only` | Check Warehouse (`W0001`) & Bins only | ✅ | ✅ | ✅ |
| `mode_stores_only` | Check Stores & In-Transit only | ✅ | ✅ | ❌ |
| `mode_wh_stores` | Check Warehouse & Stores combined | ✅ | ✅ | ✅ |
| `scan_bin` | Scan a Bin code to list all its contents | ✅ | ✅ | ✅ |
| `view_eligible_store` | View Eligible Store (`⭐`) priority indicator | ✅ | ✅ | ❌ |
| `generate_reports` | Access Distribution Reports & Excel export | ✅ | ✅ | ❌ |
| `manage_data` | Access Data Sync / Drag-and-Drop Excel uploader | ✅ | ❌ | ❌ |
| `manage_users` | Create, edit, and delete users & permissions | ✅ | ❌ | ❌ |

> Every individual permission can be toggled per user in the **Users (`اليوزرات`)** tab regardless of base role.

---

## 6. Key Implemented Features (v2.0.0)

- **Dual-Mode Responsive UI:**
  - **Desktop (`> 760px`):** Flat Excel-style tables with dynamic availability-only store columns (`> 0` or incoming TO), repeated row per Bin, clickable Barcode/Item/Bin/TO drill-down.
  - **Mobile / Android PDA (`<= 760px`):** Touch-friendly barcode cards, compact segmented mode toggle (`مستودع | محلات | الاثنان`), compact bottom navigation bar, slide-up More (`☰`) settings sheet.
- **PDA Laser Scanner Optimization:**
  - `inputmode="none"` by default when `🎯 PDA` is enabled — prevents Android soft keyboard from popping up on hardware laser scans; tapping the input box manually switches to `inputmode="text"` for typing.
  - **Auto-Enter (`⚡ إنتر تلقائي`)**: Auto-submits 300ms after scanner input burst (`>= 6` chars), persisted in `localStorage`.
  - **Icon-Only Search Bar:** Clean `🔍`, `📷` (live camera scanner), and `✖` buttons.
- **Universal Excel Export (`📥 إكسل`):**
  - Available above every table (Warehouse Bins, Stock, Transfers, TO Details, Reports, and Mobile Summary cards).
  - Uses locally hosted `xlsx.full.min.js` (zero external CDN dependency) with automatic CSV fallback.
  - Native Android `AndroidBridge.saveBase64File(...)` saves `.xlsx` directly into Android `Downloads` folder.
- **Instant Remote Updates:**
  - Data, Users, and UI updates pull directly from `raw.githubusercontent.com` (2-second propagation after push) and WebView runs with `LOAD_NO_CACHE` — **no APK reinstall required** for daily data, user, or UI updates.
  - In-app **Check App Update (`⬇️ فحص تحديث التطبيق`)** inside the `☰` menu compares installed `apk_ver` against `version.json` on GitHub.

---

## 7. Next Phase / Backlog (Planned)

1. **Dedicated Replenishment Report:** Build a specialized Replenishment report view & Excel export for Admin/Supervisors (requested to be designed after Store/WH user field testing is complete).
2. **Admin/Supervisor Report Refinements:** Any layout or filtering adjustments based on live store & warehouse team feedback.

---

## 8. Related Handover Documents

- [ARCHITECTURE_AND_FILES.md](./ARCHITECTURE_AND_FILES.md) — Complete file inventory, SQLite schema, JSON bundle structure, and API endpoints.
- [OPERATIONS_AND_APK_GUIDE.md](./OPERATIONS_AND_APK_GUIDE.md) — Daily operations, Excel syncing, user management, APK building/signing, and troubleshooting.
