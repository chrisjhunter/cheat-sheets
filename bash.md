# Bash Cheat Sheet

## Shell options & debugging
```bash
set -euo pipefail  # exit on error, unset var, pipe failure
set -x             # print each command before running (trace)
bash -x script.sh  # trace a whole script
trap 'echo error at line $LINENO' ERR
```

## Variables & substitution
```bash
${VAR:-default}               # use default if unset/empty
${VAR:=default}               # set VAR to default if unset
${VAR:?error message}         # error out if unset
${#VAR}                       # length of string
${VAR#prefix} ${VAR##prefix}  # strip shortest/longest prefix
${VAR%suffix} ${VAR%%suffix}  # strip shortest/longest suffix
${VAR/foo/bar}                # replace first match
${VAR//foo/bar}               # replace all matches
$(command)                    # command substitution
$((1 + 2))                    # arithmetic
```

## Loops & conditionals
```bash
for f in *.txt; do echo "$f"; done
for i in {1..10}; do echo "$i"; done
while read -r line; do echo "$line"; done < file.txt
[ -f file ] && echo "exists"
[[ $var =~ ^[0-9]+$ ]] && echo "numeric"
```

## Process & job control
```bash
cmd &                     # run in background
jobs -l                   # list background jobs
fg %1 / bg %1             # foreground/background job 1
nohup cmd &               # survive terminal close
disown -h %1              # detach job from shell
kill -9 $(pgrep -f name)  # force kill by name
wait                      # wait for background jobs
```

## Redirection & pipes
```bash
cmd > out.txt 2>&1       # stdout+stderr to file
cmd > out.log 2>err.log  # split streams
cmd &> all.log           # both to same file (bash)
cmd 2>/dev/null          # discard stderr
cmd | tee out.txt        # pipe + save to file
cmd < input.txt          # feed stdin from file
exec 3< file.txt         # open fd 3 for reading
```

## Strings & arrays
```bash
arr=(a b c)
echo "${arr[@]}"                    # all elements
echo "${#arr[@]}"                   # array length
arr+=(d)                            # append
IFS=',' read -ra parts <<< "a,b,c"  # split string into array
str="hello world"; echo "${str^^}"  # uppercase
echo "${str,,}"                     # lowercase
```

## Useful one-liners
```bash
history | grep ssh                      # search history
!!                                      # rerun last command
!$                                      # last arg of previous command
cd -                                    # go to previous directory
alias ll='ls -lah'
watch -n 2 'df -h'                      # rerun cmd every 2s
xargs -I{} cmd {} < list.txt            # run cmd per line
find . -name "*.log" -mtime +7 -delete  # delete files older than 7 days
find . -type f -exec chmod 644 {} \;
du -sh * | sort -h                      # sizes, human sorted
df -h                                   # disk usage
ps aux --sort=-%mem | head              # top memory hogs
lsof -i :8080                           # what's on port 8080
kill -9 $(lsof -t -i:8080)              # kill process on port
diff <(cmd1) <(cmd2)                    # process substitution diff
seq 1 5                                 # print 1..5
printf "%s\n" "${arr[@]}"               # print array, one per line
mkdir -p a/b/c                          # nested dirs
tar -czvf out.tar.gz dir/               # compress
tar -xzvf out.tar.gz                    # extract
rsync -avh --progress src/ dest/        # sync dirs
date +%Y-%m-%d                          # formatted date
readlink -f file                        # absolute path
command -v git                          # check if binary exists
```

## Functions
```bash
myfunc() {
  local name="$1"
  echo "hello $name"
}
myfunc "world"
```

## Here-docs
```bash
cat <<EOF > file.txt
line1
line2
EOF

cat <<'EOF'  # single quotes = no variable expansion
literal $HOME
EOF
```
