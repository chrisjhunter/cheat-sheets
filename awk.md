# awk Cheat Sheet

## Basics
```bash
awk '{print}' file.txt              # print every line
awk '{print $1}' file.txt           # print first field (columns, whitespace-delimited)
awk '{print $1, $3}' file.txt       # print fields 1 and 3
awk '{print $NF}' file.txt          # print last field
awk '{print $(NF-1)}' file.txt      # print second-to-last field
awk -F: '{print $1}' /etc/passwd    # custom field separator
awk 'BEGIN{OFS=","} {print $1,$2}'  # custom output separator
```

## Filtering (pattern { action })
```bash
awk '/pattern/' file.txt        # lines matching pattern (like grep)
awk '!/pattern/' file.txt       # lines NOT matching
awk '$3 > 100' file.txt         # numeric comparison on field 3
awk '$2 == "active"' file.txt   # exact match
awk 'NR==5' file.txt            # 5th line
awk 'NR>=2 && NR<=10' file.txt  # line range
awk 'NF > 3' file.txt           # lines with more than 3 fields
awk 'length($0) > 80' file.txt  # long lines
```

## Aggregation
```bash
awk '{sum += $1} END {print sum}' file.txt                 # sum a column
awk '{sum += $1} END {print sum/NR}' file.txt              # average
awk '{print $1}' file.txt | sort | uniq -c | sort -rn      # frequency count
awk '{count[$1]++} END {for (k in count) print k, count[k]}' file.txt
awk 'max=="" || $1>max {max=$1} END {print max}' file.txt  # max value
```

## BEGIN / END blocks
```bash
awk 'BEGIN {print "Start"} {print} END {print "Done"}' file.txt
awk 'BEGIN {FS=","; OFS="\t"} {print $1,$2}' file.csv  # CSV to TSV
```

## Variables & printf
```bash
awk '{ printf "%-10s %5d\n", $1, $2 }' file.txt
awk -v threshold=50 '$2 > threshold' file.txt  # pass shell var in
awk '{ total[$1] += $2 } END { for (k in total) printf "%s: %d\n", k, total[k] }' file.txt
```

## Useful one-liners
```bash
awk '{print NR, $0}' file.txt                                             # number all lines
awk 'NR % 2 == 0' file.txt                                                # print even lines
awk '!seen[$0]++' file.txt                                                # remove duplicate lines, preserve order
awk '{s+=$1} END {print s}' <(cut -f1 data.txt)                           # sum piped column
ps aux | awk '{print $2, $11}'                                            # PID + command
df -h | awk '$5+0 > 80 {print $1, $5}'                                    # disks over 80% full
awk 'BEGIN{FS=OFS=","} {$2=""; print}' file.csv                           # blank out a CSV column
netstat -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn  # conns per IP
awk '{gsub(/foo/,"bar"); print}' file.txt                                 # find & replace (like sed)
```
