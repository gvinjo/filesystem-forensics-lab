Let’s investigate one step at a time. You’ll make an observation, form a hypothesis, then test it—rather than run a list of commands without knowing why.
First: what does deletion leave behind?
A directory maps filenames to inode numbers. An inode describes the file and locates its data.
Deleting a file can remove its directory entry and free its inode and data blocks. But marking space as free does not necessarily erase the old bytes immediately. Recovery depends on what survives and whether it has been overwritten.
We’ll examine those layers separately.
Step 1 — Establish your starting point
From the Phase 1 lab directory, run:
date -u '+%Y-%m-%dT%H:%M:%SZ'
sha256sum -c evidence/SHA256SUMS.txt
Record the examination time and verification result in your Phase 2 worksheet.
Step 2 — Observe what is currently visible
TZ=UTC debugfs -R 'ls -l /home/morgan' evidence/fin-ws-17.img
For this output, the columns are:
inode   mode   (file type)   UID   GID   size   modification time   name
The inode number lets you investigate the object independently of its name. The mode combines the object type and permissions.
Now inspect the Notes directory:
TZ=UTC debugfs -R 'ls -l /home/morgan/Notes' evidence/fin-ws-17.img
Ask yourself:
- Are there any named files?
- Are only . and .. present?
- Does an empty directory prove it has always been empty?
The answer to the last question is no. A directory listing tells you what is currently linked there, not its complete history.
Step 3 — Look for surviving deletion metadata
TZ=UTC debugfs -R 'lsdel' evidence/fin-ws-17.img
This examines deleted-inode information. It does not search every free block for leftover content. On ext4, it can miss deleted files.
Record exactly what it reports, including an empty result. Don’t interpret “nothing found by this method” as “nothing ever existed.”
Paste the Notes listing and lsdel output here. We’ll read those results together and decide whether to follow an inode or search the underlying directory/data blocks next. Keep debugfs read-only—no -w.


Monday 3:36 PM








18   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 .     17   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 ..     19  100664 (1)   1000   1000     199 17-Aug-2026 14:45 .bash_history     20   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 .cache     23   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 .config     27   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 Documents     30   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 Downloads     32   40775 (2)   1000   1000    1024 27-Aug-2026 17:05 Notes what is desplayed







Monday 3:46 PM






This is the listing of /home/morgan. Each row describes one directory entry.
For example:
19  100664 (1)  1000  1000  199  17-Aug-2026 14:45  .bash_history
Column	Meaning	This example
19	Inode number	File’s metadata is in inode 19
100664	Object type and permissions, in octal	Regular file; permissions 664
(1)	Type recorded in the directory entry	Regular file
First 1000	Owner UID	Morgan
Second 1000	Group GID	Group 1000
199	Size in bytes	199 bytes
Date/time	Last content modification time	August 17, 14:45 UTC if you used TZ=UTC
.bash_history	Entry name	Shell history file


For directories, 40775 means directory with permissions 775, and (2) also identifies a directory.
What the entries tell you
Entry	Inode	Meaning
.	18	The current directory: /home/morgan
..	17	Its parent: /home
.bash_history	19	Hidden shell-history file
.cache	20	Hidden cache directory
.config	23	Hidden configuration directory
Documents	27	Directory
Downloads	30	Directory
Notes	32	Directory


A directory’s 1024-byte size describes its directory-entry storage—not the combined size of files inside it.
Your next observation is inside Notes:
TZ=UTC debugfs -R 'ls -l /home/morgan/Notes' evidence/fin-ws-17.img
Look at the names returned: are there named files, or only . and ..? Paste that output next.