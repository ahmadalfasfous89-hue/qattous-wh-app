# Operations, Remote Sync & APK Build Guide — `Qattous SM`

**Project Directory:** `C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP`

---

## 1. Daily Operations (Admin PC)

### 1.1 Starting the Local Admin Server
- Double-click **`Run_Qattous_App.bat`**, or run in PowerShell:
  ```powershell
  python "C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP\server.py"
  ```
- Open **`http://localhost:8900`** in Chrome/Edge and log in with your Admin account.

### 1.2 Updating Inventory / Excel Sheets (Remote 4G Push)
You have **two easy methods**:
- **Method A — Drag & Drop inside the Web UI (Recommended):**
  1. Open `http://localhost:8900` → go to the **البيانات (`Data`)** tab.
  2. Drag & drop the updated `.xlsx` files (e.g. `Bin Contents`, `Item Variant Inv. By Location`, `Al Najma Transfer Line`) into the dashed drop zone.
  3. The server automatically saves the files, selectively re-indexes SQLite in seconds, rebuilds `data_bundle.json`, and pushes to GitHub!
- **Method B — 1-Click Batch File:**
  1. Drop the new `.xlsx` files into the `WH aPP` folder.
  2. Double-click **`Update_Cloud_GitHub.bat`**.

> **Propagating to Android / PDA:** Within **~2–5 seconds** of the GitHub push completing, every Android phone/PDA receives the new data automatically when opened or when tapping **`🔄`**. No APK reinstall is needed.

### 1.3 Managing Users & Permissions
1. Open `http://localhost:8900` (must be done on the Local Admin PC, not the static GitHub Pages URL).
2. Go to **اليوزرات (`Users`)** → click **`➕ يوزر جديد`** or **`✏️ تعديل`**.
3. Set username, password, role preset (`admin` / `supervisor` / `user`), and toggle individual permissions.
4. Click **`💾 حفظ`** — `server.py` automatically updates `users_bundle.json` and pushes to GitHub in the background. New users can sign in on Android within seconds.

---

## 2. Deploying Online UI Updates (No APK Reinstall)

Whenever `index.html`, `xlsx.full.min.js`, or `manifest.json` is modified:
```powershell
cd "C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP"
git add index.html manifest.json
git commit -m "Describe UI update"
git push origin main
```
- GitHub Pages republishes `index.html` within ~30–60 seconds.
- Because the Android APK uses `LOAD_NO_CACHE` and appends `?v=<timestamp>`, all devices automatically load the updated UI on next launch.

---

## 3. Building & Publishing a New Android APK Version

You **only** need to rebuild the APK when changing native Android properties (app name, launcher icon, Android permissions, or `MainActivity.java` Java code).

### 3.1 Build Toolchain Prerequisites (Already Installed on This PC)
- **JDK 17:** `C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot\bin`
- **Android SDK:** `%LOCALAPPDATA%\Android\Sdk` (`platforms\android-34`, `build-tools\34.0.0`)
- **Python 3** with `Pillow` (`PIL`), `pandas`, `openpyxl`, `python-calamine`
- **Release Keystore:** `qattous_release.jks` (alias `qattous`)

### 3.2 Step-by-Step APK Release Procedure
1. In `build_apk.py`, increment `APP_VERSION` (e.g. `"2.1.0"`) and `APP_VERSION_CODE` (e.g. `3`).
2. Update `version.json` to match `"version": "2.1.0"` and `"updated_at"`.
3. Run the build script (~5 seconds):
   ```powershell
   python "C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP\build_apk.py"
   ```
   Confirm output ends with:
   `SUCCESS! Built and signed: ...\Qattous_Stock_Management_App.apk`
4. Push the new APK and `version.json` to GitHub:
   ```powershell
   cd "C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP"
   git add version.json Qattous_Stock_Management_App.apk
   git commit -m "Release APK v2.1.0"
   git push origin main
   ```
5. On Android devices, users open **`☰` → «فحص تحديث التطبيق»**, which detects the new version and shows a 1-tap **Download & Install** button (or they can scan `Qattous_SM_APK_QR.png`).

---

## 4. Verification & Troubleshooting Runbook

### 4.1 Verify Zero JavaScript Errors Before Pushing `index.html`
```powershell
$chrome = "C:\Program Files\Google\Chrome\Application\chrome.exe"
& $chrome --headless=new --disable-gpu --enable-logging=stderr --v=0 --virtual-time-budget=3000 --dump-dom "http://localhost:8900/" 2>&1 | Select-String -Pattern "SyntaxError|ReferenceError|TypeError|Uncaught"
```
*(Should return zero error lines.)*

### 4.2 Restarting `server.py` If Port `8900` Is Busy (`WinError 10048`)
```powershell
Get-CimInstance Win32_Process | Where-Object { $_.CommandLine -like '*server.py*' } | ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
python "C:\Users\AhmadAlFasfous\OneDrive - Qattous Group\Desktop\Automations\WH aPP\server.py"
```

### 4.3 Common Issues & Fixes

| Symptom | Cause | Fix |
| :--- | :--- | :--- |
| New user cannot log in on Android | User was added on GitHub Pages URL instead of `localhost:8900`, or Android hasn't refreshed yet. | Always add users on `http://localhost:8900`. On Android, tap `🔄` or reopen the app (syncs from `raw.githubusercontent.com` in ~2s). |
| Soft keyboard pops up on PDA laser scan | `🎯 PDA` toggle was unchecked or input was tapped manually. | Ensure `🎯 PDA` is checked; keyboard only opens when manually tapping the input box and hides again after scanning. |
| Excel export fails on Android | Old APK version (`< v2.0.0`) without `AndroidBridge`. | Install `Qattous_Stock_Management_App.apk` (`v2.0.0+`). Exported `.xlsx` files are saved directly to Android `Downloads`. |
| `git push` fails with auth error | GitHub credentials expired. | Run `gh auth login` or sign in via Git Credential Manager for account `ahmadalfasfous89-hue`. |
