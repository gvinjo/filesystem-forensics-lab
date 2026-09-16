# Phase 2 Examiner Worksheet — Deleted-File Recovery

Case ID: FF-2026-017  
Examiner: Giorgi Gvinjilia  
Start time (UTC): Not recorded—retrospective examination  
End time (UTC): Not recorded—retrospective examination  
Evidence item: E01 — fin-ws-17.img  

date -u '+%Y-%m-%dT%H:%M:%SZ'
2026-08-31T11:38:39Z

date -r evidence/fin-ws-17.img 
Thu Aug 27 09:05:21 PM +04 2026

## 1. Objective and scope

Identify deleted-file remnants, recover available content to a separate output directory, and assess its relevance to the Phase 1 findings.

The original evidence image must remain unchanged. Recovery may be partial or unsuccessful; document either result.

## 2. Evidence integrity

debugfs -R 'ls -l /home/morgan' evidence/fin-ws-17.img

     18   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 .
     17   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 ..
     19  100664 (1)   1000   1000     199 17-Aug-2026 14:45 .bash_history
     20   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 .cache
     23   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 .config
     27   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 Documents
     30   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 Downloads
     32   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 Notes


| Field | Observation |
|---|---|
| Image path | filesystem-forensics-phase-1/evidence/fin-ws-17.img|
| Expected SHA-256 |a8120a2903373d2d4dc23892ff08e725af3d919f313b822bbfa5162ba8489a0f  ./fin-ws-17.img |
| Calculated SHA-256 before examination |a8120a2903373d2d4dc23892ff08e725af3d919f313b822bbfa5162ba8489a0f  ./fin-ws-17.img |
| Initial verification result | Passed—the calculated digest matched the expected SHA-256 |
| Calculated SHA-256 after examination | a8120a2903373d2d4dc23892ff08e725af3d919f313b822bbfa5162ba8489a0f |
| Final verification result | Passed—the image digest remained unchanged |

## 3. Deleted-file discovery

Initial command:

```sh
TZ=UTC debugfs -R 'lsdel' evidence/fin-ws-17.img
```

 Inode  Owner  Mode    Size      Blocks   Time deleted
    33   1000 100664    111      1/     1 Thu Aug 27 17:05:21 2026
1 deleted inodes found.

----------------------------------
stat <33>
blocks <33>
dump <33> recovered_file

Check the filesystem block size with:
sudo tune2fs -l /dev/DEVICE | grep 'Block size'

-----------------------------------


- 33 is the inode’s identifier—not 33 bytes of data.
- 111 is the deleted file’s logical size.
- 1/1 means it occupied one filesystem block.
If the block size is 4096 bytes, only the first 111 bytes belong to the file; the remaining 3985 bytes are potential file slack. That slack might contain old data, zeros, or overwritten data—it is not guaranteed to contain inode 33.

| Finding | Observation |
|---|---|
| Number of deleted inodes reported | TZ=UTC debugfs -R 'lsdel' evidence/fin-ws-17.img 1 inode 33|
| Relevant inode numbers | 33|
| Errors or tool limitations | |

An empty listing does not establish that no files were deleted.

## 4. Candidate metadata

Add one row per candidate. Use “unknown” when information cannot be established.

debugfs -R 'stat <33>' evidence/fin-ws-17.img

Inode: 33   Type: regular    Mode:  0664   Flags: 0x80000
Generation: 0    Version: 0x00000000:00000000
User:  1000   Group:  1000   Project:     0   Size: 111
File ACL: 0
Links: 0   Blockcount: 2
Fragment:  Address: 0    Number: 0    Size: 0
 ctime: 0x6a906e3e:00000000 -- Thu Aug 27 21:05:02 2026
 atime: 0x6a906e3e:00000000 -- Thu Aug 27 21:05:02 2026
 mtime: 0x6a906e3e:00000000 -- Thu Aug 27 21:05:02 2026
crtime: 0x6a906e51:00000000 -- Thu Aug 27 21:05:21 2026
 dtime: 0x6a906e51:(00000000) -- Thu Aug 27 21:05:21 2026
Size of extra inode fields: 32
Inode checksum: 0x210d5cc4

| Inode | Original name/path, if supported | Size in bytes | Deletion time (UTC), if available | Referenced blocks/extents | Evidence source |
|---|---|---|---|---|---|
| 33| Unknown—not supported by the recovered metadata | 111| Thu Aug 27 21:05:21 2026| Filesystem block 2380 | `debugfs stat <33>` and `debugfs blocks <33>` |

Record original names only when supported by evidence. A name assigned during recovery is not necessarily the original filename.

## 5. Recovery record

| Candidate inode or byte offset | Recovery method/command | Saved output path | Recovered size | SHA-256 | Result: complete, partial, or unknown |
|---|---|---|---|---|---|
| 33| `debugfs -R "dump <33> recovered.txt" evidence/fin-ws-17.img`| `phase-2/recovered/recovered.txt`| 111 bytes| a2c64a64d7892041df8f3755dcc9e1075497335622aafb4374bf7d6dc2ee6393 | Apparently complete; original name and path remain unknown |

Recovered files must be saved outside the evidence image.

## 6. Content analysis

For each recovered artifact, record:

- Recovered filename:../filesystem-forensics-phase-2/recovered/recovered.txt 
- Detected file type:ASCII text
- Relevant content:
Synthetic deleted-file seed for a later phase.
Destination: SILVERKEY/.archive
Window: after the 14:30 meeting

- Links to Phase 1 artifacts:destination": "/media/morgan/SILVERKEY/.archive
- Evidence supporting that connection:
- Limitations or alternative explanations:
it is ctf marking the silverkey directory and .archive hidden dir it is ascii text and it was recovered from inode 33 

A successful extraction does not automatically prove the recovered content is complete or belongs to the suspected deleted file.

## 7. Additional timeline findings

| Incident time (UTC) | Event or observation | Source | Confidence and limitations |
|---|---|---|---|
| `2026-08-27T21:05:21Z` | Inode 33 was deleted | `debugfs lsdel`; `debugfs stat <33>` | High confidence for the filesystem deletion time; it appears to be image-construction activity rather than the simulated incident |
| After `2026-08-17T14:30:00Z` | Recovered text references `SILVERKEY/.archive` and a transfer window after 14:30 | Recovered inode 33 | The time is written in the recovered content and is not an independently verified filesystem timestamp |

Distinguish filesystem timestamps from dates written inside a file. Do not treat image-construction timestamps as incident events.

## 8. Assessment

What was recovered?

> A 111-byte ASCII text artifact was recovered from deleted inode 33 using filesystem block 2380. It references SILVERKEY, the hidden `.archive` directory, and a transfer window after 14:30.

What does it add to the Phase 1 findings?

> The recovered content is consistent with the Phase 1 transfer evidence and provides another connection between the deleted artifact and the destination directory `/media/morgan/SILVERKEY/.archive`.

What remains uncertain?

1. The original filename and exact directory entry were not recovered.
2. The USB evidence was not acquired, so the destination file cannot be independently verified.
3. The recovered content may be a note about the transfer rather than proof that the transfer occurred.

Recommended next actions:

1. Acquire and hash the SILVERKEY USB device.
2. Compare the source and destination copies of `client_list.csv` using cryptographic hashes.
3. Extract and statically analyze `Invoice_August.pdf.exe` without executing it.

## 9. Examination log

Record when you performed each action—not when the simulated incident happened.

| Examination time (UTC) | Action/command | Result or note |
|---|---|---|
| Not recorded—retrospective entry | Initial image hash verification | Passed; calculated SHA-256 matched the supplied digest |
| Not recorded—retrospective entry | Listed deleted inodes | Located deleted inode 33 |
| Not recorded—retrospective entry | Inspected candidate metadata | Regular file, 111 bytes, deleted at `2026-08-27T21:05:21Z`; referenced block 2380 |
| Not recorded—retrospective entry | Recovered candidate to a separate directory | Saved as `phase-2/recovered/recovered.txt` |
| Not recorded—retrospective entry | Calculated recovered-file hash | `a2c64a64d7892041df8f3755dcc9e1075497335622aafb4374bf7d6dc2ee6393` |
| Not recorded—retrospective entry | Final image hash verification | Passed; evidence-image SHA-256 remained unchanged |

If a past action's time was not recorded, write “Not recorded—retrospective entry” rather than inventing a timestamp.

## 10. Completion

Status: Complete  
Evidence image unchanged: Yes—verified using matching SHA-256 digests  
Recovery outputs saved to: `phase-2/recovered/`  
Key limitations: Original deleted filename and path are unknown; the destination USB was unavailable for examination.
