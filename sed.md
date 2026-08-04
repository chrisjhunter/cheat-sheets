# sed Cheat Sheet

> Note: examples use GNU sed syntax. On macOS (BSD sed), `-i` requires an explicit (even empty) backup extension: `sed -i '' ...`.

## Basics
```bash
sed 's/foo/bar/' file.txt          # replace first occurrence per line
sed 's/foo/bar/g' file.txt         # replace all occurrences
sed 's/foo/bar/2' file.txt         # replace only 2nd occurrence
sed -i 's/foo/bar/g' file.txt      # edit file in place
sed -i.bak 's/foo/bar/g' file.txt  # in place, keep .bak backup
sed 's/foo/bar/gi' file.txt        # case-insensitive
```

## Line selection
```bash
sed -n '5p' file.txt              # print only line 5
sed -n '5,10p' file.txt           # print lines 5-10
sed '5d' file.txt                 # delete line 5
sed '5,10d' file.txt              # delete lines 5-10
sed '/pattern/d' file.txt         # delete lines matching pattern
sed -n '/start/,/end/p' file.txt  # print between two patterns
sed '1!G;h;$!d' file.txt          # reverse file (tac equivalent)
```

## Regex & groups
```bash
sed -E 's/([a-z]+)@([a-z]+)/\2@\1/' file.txt  # extended regex + backreferences
sed 's/^/PREFIX: /' file.txt                  # prepend to each line
sed 's/$/  SUFFIX/' file.txt                  # append to each line
sed 's/[ \t]*$//' file.txt                    # strip trailing whitespace
sed '/^$/d' file.txt                          # remove blank lines
sed 's/\r$//' file.txt                        # strip Windows CRLF
```

## Multiple commands
```bash
sed -e 's/foo/bar/' -e 's/baz/qux/' file.txt
sed 's/foo/bar/; s/baz/qux/' file.txt
sed -f script.sed file.txt  # commands from a file
```

## Useful one-liners
```bash
sed -n '$p' file.txt                                  # print last line
sed '2,4!d' file.txt                                  # print only lines 2-4
sed -i '1i\NEW FIRST LINE' file.txt                   # insert at top
sed -i '$a\NEW LAST LINE' file.txt                    # append at end
sed 's/,/\n/g' file.csv                               # replace commas with newlines
sed -n 's/.*ERROR: \(.*\)/\1/p' log.txt               # extract text after "ERROR:"
sed -i 's/\(.*\)/\L\1/' file.txt                      # lowercase whole line
find . -name "*.txt" -exec sed -i 's/foo/bar/g' {} +  # bulk edit files
```
