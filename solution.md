# Space Mission Log Analysis — Solution

## Answer

**Security Code: `XRT-421-ZQP`**

Mission WGU-0200 — the longest completed Mars mission at 1,629 days.

## Approach

The log file (~105K lines) has several parsing challenges:
- Comment lines starting with `#`
- Metadata lines (`SYSTEM:`, `CONFIG:`, `CHECKSUM:`, `CHECKPOINT:`)
- Inconsistent whitespace around the `|` delimiter
- Empty lines scattered throughout

### Step 1: Filter and extract with awk

```bash
awk -F'|' '!/^#/ && !/^SYSTEM/ && !/^CONFIG/ && !/^CHECKSUM/ && !/^CHECKPOINT/ && !/^$/ {
  gsub(/^[ \t]+|[ \t]+$/, "", $3)  # Destination
  gsub(/^[ \t]+|[ \t]+$/, "", $4)  # Status
  gsub(/^[ \t]+|[ \t]+$/, "", $6)  # Duration
  gsub(/^[ \t]+|[ \t]+$/, "", $8)  # Security Code
  gsub(/^[ \t]+|[ \t]+$/, "", $2)  # Mission ID

  if ($3 == "Mars" && $4 == "Completed") {
    duration = $6 + 0
    if (duration > max_duration) {
      max_duration = duration
      security_code = $8
      mission_id = $2
    }
  }
}
END {
  print "Duration:", max_duration, "days"
  print "Mission ID:", mission_id
  print "Security Code:", security_code
}' space_missions.log
```

Key decisions:
- `-F'|'` splits on pipe characters
- Pattern negation (`!/^#/` etc.) skips non-data lines in a single pass
- `gsub` trims whitespace from each field to handle inconsistent spacing
- `$6 + 0` coerces the duration string to a number for comparison

### Step 2: Verify by sorting all candidates

```bash
awk -F'|' '!/^#/ && !/^SYSTEM/ && !/^CONFIG/ && !/^CHECKSUM/ && !/^CHECKPOINT/ && !/^$/ {
  gsub(/^[ \t]+|[ \t]+$/, "", $3)
  gsub(/^[ \t]+|[ \t]+$/, "", $4)
  gsub(/^[ \t]+|[ \t]+$/, "", $6)
  gsub(/^[ \t]+|[ \t]+$/, "", $8)
  gsub(/^[ \t]+|[ \t]+$/, "", $2)

  if ($3 == "Mars" && $4 == "Completed") {
    printf "%s\t%s\t%s\n", $6, $2, $8
  }
}' space_missions.log | sort -t$'\t' -k1 -rn | head -5
```

Result:
```
1629    WGU-0200    XRT-421-ZQP
1482    AJV-3533    WCN-103-DVD
1479    LBS-1848    ZCA-027-KCP
1422    LTZ-4413    DHA-730-NYP
1417    PGQ-7628    NQT-363-IFR
```

The top result is a clear winner — 147 days longer than the runner-up.
