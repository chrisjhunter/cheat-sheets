# Go Cheat Sheet

## Build & run
```bash
go run main.go                                 # compile + run in one step (no binary left behind)
go run .                                       # run the package in current dir
go build                                       # build binary named after the module/dir
go build -o myapp .                            # build with a specific output name
go build ./...                                 # build all packages in the module
go install ./cmd/mytool                        # build + install to $GOBIN
GOOS=linux GOARCH=amd64 go build -o app-linux  # cross-compile
go build -ldflags="-s -w"                      # strip debug info, smaller binary
```

## Modules
```bash
go mod init github.com/user/project   # start a new module
go mod tidy                           # add missing / remove unused deps
go mod download                       # download deps into local cache
go mod vendor                         # copy deps into ./vendor
go mod why github.com/pkg/errors      # why is this dependency needed
go mod graph                          # full dependency graph
go get github.com/gorilla/mux         # add/update a dependency
go get github.com/gorilla/mux@v1.8.0  # pin a specific version
go get -u ./...                       # update all deps to latest minor/patch
go list -m all                        # list all modules in the build
```

## Testing
```bash
go test ./...                          # run all tests in module
go test -v ./...                       # verbose output
go test -run TestFoo ./...             # run tests matching a name pattern
go test -cover ./...                   # coverage percentage
go test -coverprofile=cover.out ./...  # write coverage profile
go tool cover -html=cover.out          # view coverage in browser
go test -race ./...                    # detect data races
go test -bench=. ./...                 # run benchmarks
go test -bench=. -benchmem ./...       # benchmarks + memory allocation stats
go test -count=1 ./...                 # force re-run, bypass test cache
```

## Formatting, vetting, linting
```bash
go fmt ./...       # format all files
gofmt -l .         # list files that need formatting (no changes)
go vet ./...       # catch common mistakes (unreachable code, bad printf, etc.)
staticcheck ./...  # deeper static analysis (needs staticcheck installed)
golangci-lint run  # aggregated linter suite (needs golangci-lint installed)
```

## Environment
```bash
go env                                             # dump full Go environment
go env GOPATH                                      # just one variable
go env GOROOT
go env -w GOPROXY=https://proxy.golang.org,direct  # persist a config change
go version
```

## Analysis & debugging tools
```bash
go install golang.org/x/tools/cmd/callgraph@latest
callgraph -algo=cha ./... | less    # static call graph (cha/rta/pta/static algos)
go install github.com/google/pprof@latest
go tool pprof cpu.prof              # analyze a CPU profile
go tool pprof -http=:8080 cpu.prof  # interactive web UI for a profile
go build -gcflags="-m" main.go      # show escape analysis decisions
dlv debug main.go                   # interactive debugger (needs delve)
dlv test ./...                      # debug a test
```

## Profiling in code (net/http/pprof)
```go
import _ "net/http/pprof"
// then: go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

## Useful one-liners
```bash
go doc fmt.Println                           # docs for a specific symbol
go doc -all ./mypkg                          # full package docs
find . -name "*.go" | xargs wc -l | tail -1  # total lines of Go code
go list ./... | xargs -I{} go test {}        # test packages one at a time
GOFLAGS=-mod=mod go build                    # temporarily allow go.mod edits during build
go build -v ./... 2>&1 | grep -v "^$"        # verbose build, drop blank lines
```
