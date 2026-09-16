# Phase 1 Examiner Worksheet

Case ID: FF-2026-017  
Examiner: Giorgi Gvinjilia  
Start time (UTC): Not recorded—retrospective examination  

## 1. Intake and integrity

| Field | Observation |
|---|---|
| Evidence filename | fin-ws-17.img|
| Size in bytes | 33554432 bytes|
| Expected SHA-256 | a8120a2903373d2d4dc23892ff08e725af3d919f313b822bbfa5162ba8489a0f|
| Calculated SHA-256 | a8120a2903373d2d4dc23892ff08e725af3d919f313b822bbfa5162ba8489a0f|
| Integrity result | OK|

## 2. Filesystem profile

| Field | Observation |
|---|---|
| Filesystem family/type |Linux rev 1.0 ext4 filesystem data |
| Volume label | "LAB_PHASE1"|
| UUID | 8f1b3d22-6c74-4a5c-9e17-202608170017|
| Block size | 1024|
| Total inode count | 8192|

## 3. System and user profile

| Field | Observation | Source path or command |
|---|---|---|
| Hostname |fin-ws-17 | debugfs -R 'cat /etc/hostname' evidence/fin-ws-17.img|
| Primary interactive user |morgan | debugfs -R 'ls -l /home' evidence/fin-ws-17.img|


password:

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
morgan:x:1000:1000:Morgan Hale:/home/morgan:/bin/bash

## 4. Removable-media activity

| Field | Observation | Source path |
|---|---|---|
| Device label |label=SILVERKEY | |
| Serial number | serial=VX9A-44Q2 | |
| Mount path | /media/morgan/SILVERKEY |/var/log/usb-events.log|
| Connected (UTC) |2026-08-17T14:27:43Z |/var/log/usb-events.log |
| Unmounted (UTC) |2026-08-17T15:02:18Z | `/var/log/usb-events.log` |
| Disconnected (UTC) |2026-08-17T15:02:20Z | `/var/log/usb-events.log` |

## 5. Triage findings

The .desktop file is the persistence configuration.
The file named by Exec= is the configured executable.
This proves that autostart was configured, but it does not alone prove the executable ran successfully.

--------------------------
Type=Application
Name=System Update Monitor
Exec=/home/morgan/Downloads/Invoice_August.pdf.exe --silent
Hidden=false
X-GNOME-Autostart-enabled=true
------------------------
User:  1000   Group:  1000   Project:     0   Size: 132
File ACL: 0
Links: 1   Blockcount: 2
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x6a906e51:00000000 -- Thu Aug 27 17:05:21 2026
 atime: 0x6a831b3a:00000000 -- Mon Aug 17 14:31:22 2026
 mtime: 0x6a831b3a:00000000 -- Mon Aug 17 14:31:22 2026
crtime: 0x6a906e51:00000000 -- Thu Aug 27 17:05:21 2026
Size of extra inode fields: 32
Inode checksum: 0x76988db1
----------------
atime: last content access time.
mtime: last content modification time.
ctime: last inode metadata-change time—permissions, ownership, link count, or when the file was introduced into this filesystem.

----------------
atime: Aug 17 — simulated access time preserved from the source
ctime: Aug 27 — inode created/changed while the lab image was assembled
---------------------
A copy indicator is evidence that a copy command or operation was requested.
A queue indicator is a record created by software describing a pending or completed transfer job.

----------------------------

{
  "job_id": "job-8841",
  "source": "/home/morgan/Documents/client_list.csv",
  "destination": "/media/morgan/SILVERKEY/.archive/client_list.csv",
  "status": "completed",
  "completed_utc": "2026-08-17T14:47:09Z"
}

sync-service.log

debugfs 1.47.4 (6-Mar-2025)
2026-08-17T14:46:51Z user=morgan job=job-8841 source=/home/morgan/Documents/client_list.csv destination=/media/morgan/SILVERKEY/.archive/client_list.csv state=queued
2026-08-17T14:47:09Z user=morgan job=job-8841 bytes=143 state=completed

.bash_history

debugfs 1.47.4 (6-Mar-2025)
cd ~/Downloads
chmod +x Invoice_August.pdf.exe
./Invoice_August.pdf.exe --silent
mkdir -p /media/morgan/SILVERKEY/.archive
cp ~/Documents/client_list.csv /media/morgan/SILVERKEY/.archive/
history -c

.config/recent-files.txt

file:///home/morgan/Documents/Q3-planning.txt
file:///home/morgan/Documents/client_list.csv
file:///home/morgan/Downloads/Invoice_August.pdf.exe
file:///media/morgan/SILVERKEY/client_list.csv

| Finding | Timestamp (UTC) | Artifact/source | Why it matters |
|---|---|---|---|
| Suspicious download | `2026-08-17T14:31:22Z` | `/var/log/downloads.log`; `/home/morgan/Downloads/Invoice_August.pdf.exe` | The file has a misleading double extension and was later referenced by an autostart entry. |
| Persistence indicator | Exact configuration time not independently confirmed | `/home/morgan/.config/autostart/system-update.desktop` | The enabled autostart entry launches the executable from `Downloads` when the user logs in. |
| Copy/queue indicator | Queued `2026-08-17T14:46:51Z`; completed `14:47:09Z` | `queue.json`, `sync-service.log`, `.bash_history` | Multiple artifacts indicate that `client_list.csv` was copied to the USB’s hidden `.archive` directory. |
## 6. Event timeline

| UTC time | Event | Source | Confidence |
|---|---|---|---|
| `2026-08-17T14:27:43Z` | SILVERKEY connected | `/var/log/usb-events.log` | High |
| `2026-08-17T14:27:45Z` | SILVERKEY mounted at `/media/morgan/SILVERKEY` for UID 1000 | `/var/log/usb-events.log` | High |
| `2026-08-17T14:31:22Z` | `Invoice_August.pdf.exe` downloaded | `/var/log/downloads.log` | High |
| `2026-08-17T14:33:02Z` | Morgan used `sudo` for a `chmod` operation | `/var/log/auth.log` | High |
| Time not recorded | Executable invoked and copy command entered | `/home/morgan/.bash_history` | Medium—history has no timestamps |
| `2026-08-17T14:46:51Z` | Client-list transfer job queued | `/var/log/sync-service.log` | High |
| `2026-08-17T14:47:09Z` | Transfer service reported 143-byte job completed | `sync-service.log`; `queue.json` | High |
| `2026-08-17T15:02:18Z` | SILVERKEY unmounted | `/var/log/usb-events.log` | High |
| `2026-08-17T15:02:20Z` | SILVERKEY disconnected | `/var/log/usb-events.log` | High |
## 7. Assessment

Working hypothesis:

> Evidence strongly indicates that client_list.csv was transferred to the mounted SILVERKEY USB device and that Invoice_August.pdf.exe remained on the workstation with an autostart entry configured.

Alternative explanations or unresolved questions:

1. The shell history records execution of `Invoice_August.pdf.exe`, but it does not independently prove that the process executed successfully or created the autostart entry.
2. The transfer logs report that `client_list.csv` was copied successfully, but the destination USB image was not acquired. The destination file and its hash therefore could not be independently verified.

Recommended Phase 2 actions:

Acquire and forensically image the SILVERKEY USB device, calculate and record its hash, and determine whether .archive/client_list.csv exists. Compare its hash with the workstation copy to confirm a byte-for-byte transfer.

Examine deleted files, unallocated space, ext4 inode metadata, and the filesystem journal. Attempt recovery without modifying the original image and hash every recovered artifact.

Extract Invoice_August.pdf.exe to a separate analysis location without executing it. Calculate its hash, identify its true file type, inspect permissions and readable strings, and analyze its behavior in an isolated environment if further execution analysis is required.

## 8. Examination log

Note: The following entries were added retrospectively because exact examination times were not recorded. All commands were read-only unless otherwise noted.

| UTC time | Action/command | Result or note |
|---|---|---|
| Not recorded—retrospective | `stat -c '%n %s bytes' evidence/fin-ws-17.img` | Evidence filename confirmed as `fin-ws-17.img`; size was 33,554,432 bytes. |
| Not recorded—retrospective | `sha256sum -c evidence/SHA256SUMS.txt` | Integrity verification passed; calculated SHA-256 matched the supplied digest. |
| Not recorded—retrospective | `file evidence/fin-ws-17.img` | Identified a Linux ext4 filesystem image. |
| Not recorded—retrospective | `TZ=UTC dumpe2fs -h evidence/fin-ws-17.img` | Recorded volume label, UUID, block size, inode count, and clean filesystem state. |
| Not recorded—retrospective | `debugfs -R 'cat /etc/hostname' evidence/fin-ws-17.img` | Hostname identified as `fin-ws-17`. |
| Not recorded—retrospective | `debugfs -R 'cat /etc/passwd' evidence/fin-ws-17.img` | Morgan identified as UID 1000 with home directory `/home/morgan` and interactive shell `/bin/bash`. |
| Not recorded—retrospective | `debugfs -R 'cat /var/log/usb-events.log' evidence/fin-ws-17.img` | Identified SILVERKEY connection, mount, unmount, and disconnection events. |
| Not recorded—retrospective | Examined `/home/morgan/Downloads` and `downloads.log` | Located `Invoice_August.pdf.exe`; download recorded at `2026-08-17T14:31:22Z`. |
| Not recorded—retrospective | Examined `/home/morgan/.config/autostart/system-update.desktop` | Found an enabled autostart entry pointing to the executable in `Downloads`. |
| Not recorded—retrospective | Examined `.bash_history` | Found commands that made and ran the executable and copied `client_list.csv` to the USB’s hidden `.archive` directory. History has no individual command timestamps. |
| Not recorded—retrospective | Examined `.cache/.sync/queue.json` and `/var/log/sync-service.log` | Transfer job `job-8841` was queued at `14:46:51Z` and reported completed at `14:47:09Z`. |
| Not recorded—retrospective | Correlated USB, download, autostart, history, and synchronization artifacts | Evidence supports a USB transfer and configured persistence; destination copy cannot be verified without examining the USB. |


Status: Complete
Integrity at completion: Verified
