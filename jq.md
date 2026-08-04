# jq Cheat Sheet

## Basics
```bash
echo '{"name":"chris"}' | jq .          # pretty-print JSON
curl -s https://api.example.com | jq .  # pretty-print API response
jq . file.json                          # pretty-print a file
jq -c . file.json                       # compact (one line) output
jq -r .name file.json                   # raw output, no quotes
jq -S . file.json                       # sort keys
```

## Field access
```bash
jq '.name' file.json               # top-level field
jq '.user.address.city' file.json  # nested field
jq '.items[0]' file.json           # first array element
jq '.items[-1]' file.json          # last array element
jq '.items[]' file.json            # iterate all array elements
jq '.items[2:5]' file.json         # slice
jq '.["odd-key"]' file.json        # key with special chars
```

## Filtering & selecting
```bash
jq '.items[] | select(.active == true)' file.json
jq '.items[] | select(.price > 100)' file.json
jq '[.items[] | select(.category == "tools")]' file.json  # wrap results back into an array
jq 'map(select(.active))' file.json                       # same idea, using map
jq 'any(.items[]; .price > 1000)' file.json               # true if any item matches
jq 'all(.items[]; .active)' file.json                     # true if all items match
```

## Transforming
```bash
jq '.items[] | {name, price}' file.json      # pick specific fields into new objects
jq '.items | map(.price)' file.json          # extract array of just prices
jq '.items | map(.price) | add' file.json    # sum a field
jq '.items | length' file.json               # count array elements
jq '.items | sort_by(.price)' file.json      # sort array by field
jq '.items | group_by(.category)' file.json  # group into arrays by field
jq '.items | unique_by(.id)' file.json       # dedupe by field
jq 'to_entries' file.json                    # object -> array of {key,value}
jq 'from_entries' file.json                  # reverse of to_entries
```

## Building objects/arrays
```bash
jq -n '{name: "chris", age: 30}'             # build JSON from scratch
jq '. + {"active": true}' file.json          # merge in a new field
jq 'del(.password)' file.json                # remove a field
jq '{id, total: (.price * .qty)}' file.json  # computed field
```

## Working with multiple files / streams
```bash
jq -s '.' file1.json file2.json              # slurp multiple files into one array
jq -s 'add' file1.json file2.json            # merge arrays from multiple files
jq --slurpfile refs refs.json '.' file.json  # load a file into a named variable
jq -n --arg name "chris" '{name: $name}'     # inject a shell variable as a string
jq -n --argjson count 5 '{count: $count}'    # inject a shell variable as JSON (number/bool/etc)
```

## Useful one-liners
```bash
curl -s https://api.example.com/users | jq -r '.[] | .email'      # extract a column as plain text
kubectl get pods -o json | jq -r '.items[].metadata.name'         # k8s pod names
docker inspect mycontainer | jq '.[0].NetworkSettings.IPAddress'  # extract nested docker field
cat access.json | jq -r '[.time, .status, .path] | @csv'          # JSON to CSV rows
jq 'paths' file.json                                              # list all key paths (great for exploring unfamiliar JSON)
jq empty file.json && echo "valid JSON"                           # validate JSON, no output
echo '[1,2,3]' | jq 'add'                                         # quick sum
```
