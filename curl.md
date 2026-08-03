# curl Cheat Sheet

## Basics
```bash
curl https://example.com                    # GET, print body to stdout
curl -o file.html https://example.com        # save to named file
curl -O https://example.com/file.zip         # save using remote filename
curl -L https://example.com                   # follow redirects
curl -s https://example.com                   # silent (no progress meter)
curl -sS https://example.com                  # silent but show errors
curl -v https://example.com                   # verbose (debug request/response)
curl -I https://example.com                    # HEAD request only (headers)
```

## Methods & data
```bash
curl -X POST https://api.example.com/users
curl -X DELETE https://api.example.com/users/1
curl -d "name=chris&age=30" https://api.example.com/users        # form POST
curl -d @data.json -H "Content-Type: application/json" https://api.example.com/users
curl -F "file=@photo.jpg" https://api.example.com/upload          # multipart upload
```

## JSON APIs
```bash
curl -H "Content-Type: application/json" -d '{"name":"chris"}' https://api.example.com/users
curl -s https://api.example.com/users | jq .                      # pretty-print JSON
curl -s https://api.example.com/users | jq '.[] | .name'
```

## Auth & headers
```bash
curl -u user:pass https://example.com                              # basic auth
curl -H "Authorization: Bearer $TOKEN" https://api.example.com
curl -H "X-Custom-Header: value" https://example.com
curl --cookie "session=abc123" https://example.com
curl --cookie-jar cookies.txt --location https://example.com        # save cookies
```

## Debugging & inspection
```bash
curl -w "\n%{http_code}\n" -o /dev/null -s https://example.com      # just the status code
curl -w "@curl-format.txt" -o /dev/null -s https://example.com       # timing breakdown
curl -i https://example.com                                           # include response headers in output
curl --trace-ascii debug.log https://example.com                      # full trace to file
curl -sS -o /dev/null -w "%{time_total}\n" https://example.com         # total request time
```

## Retries, timeouts, misc
```bash
curl --retry 3 --retry-delay 2 https://example.com
curl --max-time 10 https://example.com                # abort after 10s total
curl --connect-timeout 5 https://example.com            # abort if connect takes >5s
curl -k https://self-signed.example.com                  # skip TLS verification (dev only)
curl --resolve example.com:443:127.0.0.1 https://example.com  # override DNS
curl -x http://proxy:8080 https://example.com               # via proxy
```

## Useful one-liners
```bash
curl -s ifconfig.me                                    # your public IP
curl -s https://httpbin.org/ip                          # alt IP check
for i in {1..5}; do curl -s -o /dev/null -w "%{http_code}\n" https://example.com; done  # hammer test
curl -sL https://example.com/script.sh | bash            # pipe remote script to shell (be careful!)
curl -sO https://example.com/{file1,file2}.zip            # multiple files via brace expansion
```
