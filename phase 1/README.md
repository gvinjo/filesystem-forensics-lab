# Filesystem Forensics Lab — Phase 1

## Case brief

Case ID: **FF-2026-017**  
Evidence item: **E01 — `fin-ws-17.img`**  
Time standard: **UTC**

An employee workstation was isolated after an alert involving removable media. You have received a forensic image for initial examination. All people, organizations, logs, and files in this lab are fictional.

Your Phase 1 goal is to preserve the evidence record, identify the filesystem, and produce an initial triage report without modifying the image.

## Rules of engagement

- Treat `evidence/fin-ws-17.img` as read-only evidence.
- Do not mount it read-write.
- Record commands and observations as you go.
- Distinguish facts from interpretations.
- Use UTC for all timestamps.
- Do not attempt deleted-file recovery yet; that belongs to a later phase.

## Mission 1 — Intake and integrity

1. Record the evidence filename and byte size.
2. Calculate its SHA-256 digest and compare it with `evidence/SHA256SUMS.txt`.
3. State whether integrity verification passed.

## Mission 2 — Filesystem profile

Determine and record:

1. Filesystem family and type.
2. Volume label.
3. Filesystem UUID.
4. Block size.
5. Total inode count.

## Mission 3 — Read-only triage

Using read-only inspection, identify:

1. The hostname and primary interactive user.
2. The removable-media label, serial number, mount path, connection time, and disconnection time.
3. The suspicious downloaded file and its recorded download time.
4. Any persistence-related artifact.
5. Evidence that a user document was copied or queued to removable media.

## Mission 4 — Initial timeline and assessment

Create a concise UTC timeline containing at least five relevant events. Then write:

- one evidence-backed working hypothesis;
- two plausible alternative explanations or unresolved questions;
- the next three forensic actions you recommend for Phase 2.

Complete `worksheet.md`. Stop before deleted-file recovery or execution of any artifact.

## Suggested local tools

The image is a standard Linux filesystem image. Common read-only tools such as `sha256sum`, `file`, `dumpe2fs`, and `debugfs` are sufficient. See `tool-notes.md` for safe starting examples.
