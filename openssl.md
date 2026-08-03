# OpenSSL / TLS Cheat Sheet

## Inspecting certificates
```bash
openssl x509 -in cert.pem -text -noout                  # full decoded cert details
openssl x509 -in cert.pem -noout -subject -issuer -dates  # just subject/issuer/validity
openssl x509 -in cert.pem -noout -enddate                 # expiry date only
openssl x509 -in cert.pem -noout -fingerprint -sha256        # cert fingerprint
openssl req -in csr.pem -text -noout                            # inspect a CSR
```

## Checking a live server
```bash
openssl s_client -connect example.com:443                            # connect and dump handshake
openssl s_client -connect example.com:443 -servername example.com      # with SNI
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates   # remote cert expiry
openssl s_client -connect example.com:443 -showcerts                      # show full chain
echo -n | openssl s_client -connect example.com:443 -status 2>/dev/null | grep -A5 "OCSP Response"  # OCSP status
```

## Generating keys & CSRs
```bash
openssl genrsa -out key.pem 4096                          # RSA private key
openssl ecparam -genkey -name prime256v1 -out key.pem        # EC private key (faster, modern)
openssl req -new -key key.pem -out csr.pem                     # generate a CSR (interactive prompts)
openssl req -new -key key.pem -out csr.pem -subj "/CN=example.com/O=MyOrg"  # non-interactive
openssl req -x509 -new -key key.pem -days 365 -out cert.pem       # self-signed cert
```

## Self-signed cert in one shot (dev/testing)
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes \
  -subj "/CN=localhost" -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

## Converting formats
```bash
openssl x509 -in cert.pem -outform der -out cert.der           # PEM -> DER
openssl x509 -in cert.der -inform der -outform pem -out cert.pem  # DER -> PEM
openssl pkcs12 -export -out cert.pfx -inkey key.pem -in cert.pem   # PEM -> PKCS12 (.pfx)
openssl pkcs12 -in cert.pfx -out cert.pem -nodes                     # PKCS12 -> PEM
```

## Verifying & matching
```bash
openssl verify -CAfile ca.pem cert.pem                      # verify cert against a CA
# confirm a cert and key match (hashes should be identical):
openssl x509 -noout -modulus -in cert.pem | openssl md5
openssl rsa -noout -modulus -in key.pem | openssl md5
```

## Hashing & encoding
```bash
openssl dgst -sha256 file.txt                    # sha256 checksum
openssl dgst -sha256 -hmac "secret" file.txt        # HMAC-SHA256
openssl base64 -in file.txt -out file.b64             # base64 encode
openssl base64 -d -in file.b64 -out file.txt             # base64 decode
openssl passwd -6 mypassword                               # generate a SHA-512 crypt hash (e.g. /etc/shadow)
openssl rand -hex 32                                          # random hex string (e.g. secrets)
openssl rand -base64 32
```

## Symmetric encryption
```bash
openssl enc -aes-256-cbc -salt -in file.txt -out file.enc -pass pass:mysecret
openssl enc -aes-256-cbc -d -in file.enc -out file.txt -pass pass:mysecret
```

## Useful one-liners
```bash
for host in example.com api.example.com; do echo "$host:"; echo | openssl s_client -connect $host:443 -servername $host 2>/dev/null | openssl x509 -noout -enddate; done  # bulk expiry check
openssl s_client -connect example.com:443 -tls1_2                 # force a specific TLS version
openssl ciphers -v                                                    # list supported ciphers
nmap --script ssl-enum-ciphers -p 443 example.com                       # audit server's cipher support
openssl x509 -noout -text -in cert.pem | grep -A1 "Subject Alternative Name"   # extract SANs
```
