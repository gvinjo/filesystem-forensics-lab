# Safe Tool Notes

Run examples from the `filesystem-forensics-phase-1` directory.

```sh
# Record size without altering evidence
stat -c '%n %s bytes' evidence/fin-ws-17.img

# Verify the supplied digest
sha256sum -c evidence/SHA256SUMS.txt

# Identify the image
file evidence/fin-ws-17.img

# Read the filesystem header
dumpe2fs -h evidence/fin-ws-17.img

# Make metadata timestamps display in the case time standard (UTC),
# then issue read-only filesystem requests
TZ=UTC debugfs -R 'ls -l /' evidence/fin-ws-17.img
TZ=UTC debugfs -R 'cat /etc/hostname' evidence/fin-ws-17.img
```
 
Useful `debugfs` requests include `ls -l <path>`, `cat <path>`, `stat <path>`, and `dump <source> <local-destination>`. The `debugfs` directory listing includes dot-prefixed entries without an `-a` option. Avoid commands that open the image for writing.

Keep a copy of important command output in the examination log. A correct conclusion should be traceable to a specific artifact or metadata field.
