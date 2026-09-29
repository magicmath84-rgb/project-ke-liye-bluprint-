# ARCHITECT REVIEW OF PHASE 1 & INSTRUCTIONS FOR PHASE 2

Great job on Phase 1! All 8 tests passed and the foundation (`database.py`, `virtual_drive.py`, `controller.py`) is clean.

However, my Chief Architect reviewed your **ARCHITECT STATUS REPORT** and flagged a critical architectural requirement before we proceed:
- In Phase 1, you used `subst Z:` pointing to a materialized local staging folder. While `subst` is fine as a last-resort fallback, a plain local folder CANNOT intercept `open()` / `read(offset, length)` calls for un-downloaded cloud files! If we materialize real files into staging, it wastes local disk space and internet data; if we write 0-byte placeholders into a plain folder, double-clicking them opens an empty 0-byte file.

---

# YOUR TASK NOW: EXECUTE PHASE 2 (ON-DEMAND STREAMING VFS + GOOGLE DRIVE MULTI-ACCOUNT + 10KB MICRO-THUMBNAILS)

Update `PROGRESS.md` and implement **Phase 2** with the following 4 core modules:

## 1. True On-Demand Streaming VFS Engine (`cloudpool/streaming_vfs.py` + `virtual_drive.py` upgrade)
To ensure **0% local disk waste** and **Direct Cloud Streaming** on both Windows (`Z:\`) and your Linux test environment:
- Build a **Streaming Virtual File System Engine** with two adapters:
  1. **Built-in Local Loopback WebDAV Server (`127.0.0.1:<port>`)** using pure Python (`http.server` / WSGI):
     - Handles `OPTIONS`, `PROPFIND` (serves directory listings and exact file sizes directly from SQLite `nodes` — **0 KB cloud traffic**), `GET` with `Range: bytes=start-end` support (streams only the requested byte range directly from the cloud provider or staging cache in memory without saving the whole file to disk!), `PUT` (writes incoming files to staging cache + adds to `upload_queue`), `MKCOL`, `DELETE`, and `MOVE`.
     - On Windows, when `WinFsp` is not installed, mount this local loopback WebDAV server to `Z:` using `net use Z: http://127.0.0.1:<port>/CloudPool /persistent:no` (and fall back to `subst` only if WebDAV service is disabled).
  2. **WinFsp / FUSE Operations Adapter**:
     - Wire `read(path, size, offset)` and `readdir(path)` to the exact same streaming core so if `winfspy` / `fusepy` is active, it streams byte ranges directly from memory/cloud.
- **Strict Zero-Idle Guarantee:** When toggled **OFF**, the loopback server / FUSE mount and all threads must shut down completely (0 open sockets, 0 threads, 0% CPU).

## 2. Multi-Account Cloud Provider Engine + Google Drive Connector (`cloudpool/providers/`)
- Create `cloudpool/providers/base.py` defining the `CloudProvider` interface:
  - `get_quota() -> dict(total_bytes, used_bytes, free_bytes)`
  - `list_remote_nodes() -> list[RemoteNode]` (fetches metadata only: file name, size, mime_type, remote_id, thumbnail_link)
  - `read_range(remote_id: str, offset: int, length: int) -> bytes` (streams exact byte slices using HTTP `Range` headers)
  - `upload_file(local_path: Path, remote_parent_id: str, progress_cb) -> str`
  - `fetch_micro_thumbnail(remote_id: str) -> Optional[bytes]` (fetches tiny 5KB–15KB thumbnail preview)
- Create `cloudpool/providers/gdrive.py` (`GoogleDriveProvider`):
  - Supports connecting **multiple Google Drive / Gemini Pro accounts** simultaneously. Each account stores its isolated OAuth2 credentials/refresh token in the SQLite `accounts` table.
  - Uses Google Drive REST API v3 (`files.list` with `fields="files(id,name,mimeType,size,parents,thumbnailLink)"`, `alt=media` with `Range` header for streaming reads, and resumable upload support).
- Create `cloudpool/providers/mock_cloud.py` (`SimulatedCloudProvider`):
  - So we can test multi-account pooling and streaming 100% offline/in-container right now, build a realistic simulated cloud provider that can spawn multiple virtual accounts (e.g., `GoogleDrive_Account_1 (100 GB)` and `JioCloud_SIM_1 (50 GB)`) pre-loaded with sample remote photos (valid tiny JPEG/PNG images), text docs, and video files stored in a simulated remote server directory outside the local staging cache.

## 3. Smart Micro-Thumbnail Engine (`cloudpool/thumbnails.py`)
- **Rule:** Never download a full 10MB photo just to show its preview!
- Build `ThumbnailEngine`:
  - Stores and retrieves tiny **5KB–15KB micro-thumbnails** in the SQLite `thumbnails` table (`node_id`, `mime_type`, `data BLOB`, `width`, `height`, `updated_at`).
  - **For Local Uploads:** Before uploading an image from staging to the cloud, automatically generate a compressed micro-thumbnail (max `160x160` or `< 15 KB`) locally so previewing it later costs **0 KB of internet**.
  - **For Cloud Files:** Fetch the provider's tiny thumbnail endpoint (`thumbnailLink` / micro-preview) only once on demand and cache it in SQLite `thumbnails`. Second view = **0 KB internet**.
  - Expose a fast local thumbnail preview endpoint and integrate a **"Visual Drive Browser (Name + Photo Preview)"** panel inside `controller.py` so the user can also view folder contents with instant photo thumbnails right inside the Mini-Controller or File Explorer without downloading full files!

## 4. Upgrade Mini-Controller GUI & CLI (`cloudpool/controller.py`)
- Add Multi-Account Management to the GUI & CLI:
  - `--add-demo-accounts` CLI flag and GUI button to attach test multi-accounts (`Gemini Pro 100GB` + `JioCloud SIM 50GB` = `150 GB Unified Pool`) and sync their metadata tree (`0 KB` file downloads, only metadata + micro-thumbnails).
  - `--add-gdrive` CLI/GUI hook for adding real Google Drive OAuth accounts.
  - Show combined **Unified Storage Pool Bar** (e.g., `Pool: 150.0 GB Total | 2 Accounts Connected`).
  - Add a compact **Visual Preview / Thumbnail Inspector** in the controller showing file names + their cached micro-thumbnails (`KB` size badge) to prove full files are NOT downloaded.

---

# VERIFICATION & ARCHITECT STATUS REPORT (PHASE 2)
Write comprehensive tests in `tests/test_phase2.py` verifying:
1. Connecting multiple accounts combines their storage quotas into one unified pool in SQLite.
2. Syncing account metadata populates the virtual directory tree WITHOUT downloading the actual files to staging.
3. Reading/opening a remote file via the Streaming VFS (`PROPFIND` + `GET` with `Range` header or VFS `read()`) streams the exact file bytes directly from the cloud provider with **0 permanent local disk footprint**.
4. Micro-thumbnails (`< 15 KB`) are fetched/generated and cached in SQLite, and second access uses **0 bytes** of cloud network traffic.
5. Turning the drive **OFF** stops the streaming server/mount and returns to `idle = True` (0 sockets, 0 threads).

Run the full test suite (`test_phase1.py` + `test_phase2.py`) and output the **ARCHITECT STATUS REPORT — Phase 2**.
