# GNU parallel Cheat Sheet

## Basics
```bash
parallel echo ::: A B C        # run "echo A", "echo B", "echo C" in parallel
parallel echo ::: {1..4}       # same idea with a brace-expanded range
cat list.txt | parallel echo   # read args from stdin, one per line
parallel echo :::: file.txt    # read args from a file (:::: instead of :::)
parallel -j4 echo ::: {1..20}  # limit to 4 concurrent jobs
parallel -j+2 cmd ::: *        # jobs = CPU cores + 2
parallel -j150% cmd ::: *      # jobs = 150% of CPU cores
```

## Placeholders
```bash
parallel echo {} ::: aa bb                     # {} = the argument itself
parallel echo {.} ::: file.txt archive.tar.gz  # {.} = argument without last extension
parallel echo {/} ::: /path/to/file.txt        # {/} = basename only
parallel echo {//} ::: /path/to/file.txt       # {//} = dirname only
parallel echo {/.} ::: /path/to/file.txt       # basename without extension
parallel echo {#} ::: A B C                    # {#} = job sequence number (1, 2, 3...)
```

## Multiple input sources
```bash
parallel echo {1} {2} ::: A B ::: C D         # cartesian product: every combo of the two lists
parallel --link echo {1} {2} ::: A B ::: C D  # linked (pairwise): A-C, B-D, not full cartesian
parallel echo {1} {2} {3} ::: 6 7 ::: 4 5 ::: 1 2 3
```

## Multiple commands / complex jobs
```bash
parallel 'echo start {}; sleep 1; echo done {}' ::: A B C
parallel --dry-run echo sample{}.txt ::: {1..3}  # preview commands without running them
parallel --dryrun cmd ::: *                      # same, older flag spelling
```

## Progress, logging, and control
```bash
parallel --progress cmd ::: *                 # show progress bar
parallel --eta cmd ::: *                      # estimated time remaining
parallel --bar cmd ::: *                      # visual progress bar
parallel --joblog log.txt cmd ::: *           # log exit codes/timing per job
parallel --resume --joblog log.txt cmd ::: *  # resume a previously interrupted run
parallel --halt now,fail=1 cmd ::: *          # abort all jobs on first failure
parallel -k echo {} ::: 3 1 2                 # keep output in input order (default may interleave)
```

## Practical file/data processing patterns
```bash
find . -name "*.log" | parallel gzip                            # compress many files at once
ls *.jpg | parallel convert {} {.}.png                          # bulk image conversion
parallel -a hosts.txt ssh {} uptime                             # run a command across many hosts
parallel --colsep ',' echo {1} {2} :::: data.csv                # split CSV rows into fields
find . -name "*.csv" | parallel --jobs 4 'wc -l {} > {}.count'  # per-file output redirection
```

## Useful one-liners
```bash
seq 1 100 | parallel -j8 'curl -s -o /dev/null -w "%{http_code}\n" https://example.com/{}'  # load test
parallel --shuf echo ::: {1..10}                                                            # randomize job order
parallel -N2 echo ::: a b c d                                                               # group args, N per job (2 here)
parallel --pipe -N1000 wc -l < bigfile.txt                                                  # split stdin into chunks for parallel processing
parallel --dry-run -j0 'command {}' ::: * | wc -l                                           # count how many jobs would run
```
