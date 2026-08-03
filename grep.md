# grep Cheat Sheet

## Basics
```bash
grep "pattern" file.txt
grep -i "pattern" file.txt        # case-insensitive
grep -v "pattern" file.txt        # invert match (lines NOT matching)
grep -r "pattern" dir/            # recursive
grep -n "pattern" file.txt        # show line numbers
grep -c "pattern" file.txt        # count matching lines
grep -l "pattern" *.txt           # list filenames with a match
grep -L "pattern" *.txt           # list filenames WITHOUT a match
```

## Regex modes
```bash
grep -E "foo|bar" file.txt        # extended regex (egrep)
grep -P "(?<=foo)bar" file.txt    # Perl regex (lookahead/behind)
grep -F "literal.string" file.txt # fixed string, no regex (fgrep, fastest)
```

## Context
```bash
grep -A 3 "pattern" file.txt      # 3 lines after match
grep -B 3 "pattern" file.txt      # 3 lines before match
grep -C 3 "pattern" file.txt      # 3 lines before + after
```

## Word / line matching
```bash
grep -w "cat" file.txt            # whole word only (not "category")
grep -x "exact line" file.txt     # entire line must match
grep -o "pattern" file.txt        # print only the matched part
```

## Multiple patterns
```bash
grep -e "foo" -e "bar" file.txt
grep -f patterns.txt file.txt     # patterns from a file
```

## Useful one-liners
```bash
grep -rn "TODO" --include="*.js" .           # search JS files recursively
grep -rIl "pattern" .                        # -I skips binary files
grep -c "" file.txt                          # count total lines (like wc -l)
grep -oP '\d+' file.txt                      # extract all numbers
grep -E "^\s*$" -c file.txt                  # count blank lines
grep --color=always "pattern" file.txt | less -R
ps aux | grep -i "[n]ginx"                   # avoid matching grep itself
grep -rn "pattern" . --exclude-dir={node_modules,.git}
find . -name "*.log" | xargs grep -l "ERROR"
zgrep "pattern" file.gz                      # grep inside gzipped files
grep -v "^#" config.conf | grep -v "^$"       # strip comments and blank lines
```
