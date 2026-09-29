# project-ke-liye-bluprint-
# ROLE & MISSION
You are the Lead Systems Developer building a Windows desktop application called **"CloudPool USB"** (Unified Multi-Account Cloud Virtual Drive). 
I am working with a Chief Architect who designed the Master Blueprint below. Your job is to strictly follow this blueprint, build the project phase-by-phase, test your code on my Windows machine, and output an **ARCHITECT STATUS REPORT** at the end of every phase so my Architect can review your work.

---

# MASTER BLUEPRINT (SAVE THIS TO `BLUEPRINT.md`)

## 1. Core Vision
Turn multiple free cloud storage accounts (multiple JioCloud SIM accounts + multiple Google Drive / Gemini Pro accounts) into a **single unified Virtual USB Drive (`Z:\ CloudPool`)** inside Windows File Explorer (`This PC`).

## 2. Non-Negotiable Architecture Rules
1. **Master ON/OFF Switch & Zero Idle Load:**
   - Must have a lightweight Taskbar/System Tray or Mini-Controller with a Master **ON / OFF** switch.
   - **When OFF:** The virtual drive `Z:\` completely unmounts and disappears from `This PC`. Background CPU/RAM usage must drop to **0%** and network usage to **0 KB/s**.
   - **When ON:** Mounts `Z:\ CloudPool` within 2 seconds.
2. **Strict Data-Saver Shield (For Limited Daily Internet):**
   - **NO Auto-Sync / NO Full Local Mirroring:** Opening the drive or folders must ONLY fetch file metadata (names, sizes, folder tree) from the local SQLite index.
   - **Direct Cloud Streaming (0% Local Storage Waste):** Files must NOT be permanently downloaded to local storage just to view them. Opening a file streams the required bytes directly on-demand (like watching a stream or reading from a USB drive).
3. **Smart Micro-Thumbnail Engine (Name + Photo Preview Without Full Download):**
   - Users must see image/video thumbnails inside File Explorer without Windows downloading full 10MB–500MB files.
   - **How:** Intercept thumbnail/preview requests or use Windows Shell Thumbnail Handler / Cloud Thumbnail APIs to fetch ONLY tiny **5KB–15KB micro-thumbnails** and cache those tiny previews locally in SQLite/disk.
   - When uploading files from PC, generate a 10KB micro-thumbnail locally before uploading so previewing costs **0 KB** of internet.
4. **Server-Grade Storage Pool & 50MB Block Chunking:**
   - A local **SQLite Master Index** tracks all connected accounts, free space per account, virtual directory trees, and file-to-account mappings.
   - **Small Files (< 50 MB):** Stored whole on a single account (ensuring existing photos/videos already in Google Drive or JioCloud appear natively in the drive).
   - **Large Files (>= 50 MB):** Split into **50 MB chunks** (`Part_1`, `Part_2`, etc.) distributed across multiple accounts if needed, with **resumable upload/download** support so interrupted transfers never waste mobile data.
5. **Fast Temp Write Cache + Real-Time Upload Meter:**
   - Copying a file into `Z:\` writes instantly to a local temporary staging folder, then uploads in the background and auto-deletes the temp file once 100% uploaded.
   - The Mini-Controller must display **Real-Time Upload Progress**: Files in queue, exact percentage (`%`), MB uploaded / Total MB, Live Speed (`MB/s`), and a **Pause/Resume** button.

---

# YOUR TASK RIGHT NOW: EXECUTE PHASE 1 (FOUNDATION & VIRTUAL USB ENGINE)

Do NOT build the whole app at once. Right now, execute **ONLY Phase 1**:

1. **Initialize Project & Memory Files:**
   - Create `BLUEPRINT.md` containing the full architecture above.
   - Create `PROGRESS.md` tracking our 4 Phases:
     - Phase 1: Base Virtual USB Drive (`Z:\`) + Master ON/OFF Controller + Local SQLite Metadata Index.
     - Phase 2: Google Drive Multi-Account Connector + On-Demand Streaming + Micro-Thumbnails.
     - Phase 3: JioCloud Multi-Account Connector + 50MB Server Chunking & Storage Pooling.
     - Phase 4: Real-Time Upload Queue, Speed Meter, Pause/Resume & Final Polish.
2. **Check Environment & Dependencies (Windows):**
   - Check Python version and virtual environment.
   - Check if **WinFsp** (Windows File System Proxy) is installed on this PC (required to mount a real Virtual Drive in `This PC`). If not installed, install it via `winget install WinFsp.WinFsp` or set up the cleanest reliable Windows Virtual Drive mount method (`winfsp` / `refuse` / `fusepy` or local WebDAV-to-Drive mount fallback if WinFsp requires reboot).
3. **Build Phase 1 Working Prototype:**
   - Build the **SQLite Master Index** (`database.py`) for virtual files/folders and storage pool stats.
   - Build the **Virtual Drive Engine** (`virtual_drive.py`) that mounts `Z:\` (labeled `CloudPool`) in Windows File Explorer and serves a test directory structure from the SQLite index + staging cache so I can actually see `Z:\` in `This PC`, open it, and test creating/viewing a test file.
   - Build the **Mini-Controller GUI / Tray Toggle** (`controller.py`) with:
     - A big **DRIVE ON / OFF** toggle button.
     - Status indicator (`Mounted at Z:\` vs `Offline - 0% Load`).
     - Placeholder UI for the Live Upload Meter (`0 of 0 Files | 0.0 MB/s`) and Storage Pool bar.
4. **Verify & Run:**
   - Test the code, fix any errors, and make sure the Mini-Controller launches and mounts/unmounts `Z:\` cleanly.

---

# REQUIRED OUTPUT FORMAT (ARCHITECT STATUS REPORT)
When you finish Phase 1, provide a concise **ARCHITECT STATUS REPORT** at the end of your response with:
1. **Environment Check:** Python version, WinFsp status, and mount mechanism used.
2. **Files Created:** List of files and their exact purpose.
3. **What Works Right Now:** How `Z:\` mounts/unmounts and what passed testing.
4. **Any Blockers/Warnings:** Anything my Architect needs to know before Phase 2.
