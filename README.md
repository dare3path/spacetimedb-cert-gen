# SpacetimeDB Cert Generator

Bash scripts to generate TLS certificates for SpacetimeDB standalone server, compatible with [spacetimedb-tls-patch](https://github.com/dare3path/spacetimedb-tls-patch).


## Scripts
- in order of execution:
1. `01_makeCA`: Creates CA’s private key `ca.key` (secret, keep safe). CA=certificate authority
2. `02_makeCApubcertSS`: Makes CA public cert `ca.crt` (self-signed).
3. `03_genServerPrivKey`: Generates server private key `server.key` (secret, pass to your spacetimedb standalone hosting server with --key).
4. `04_gentempCSRforServer`: Creates a temporary Certificate Signing Request (CSR) `server.csr` for the server, (can delete after signing).
5. `05_signCSRwiththeCA`: Signs the CSR with the CA thus generating `server.crt` Server’s public certificate (pass to spacetimedb standalone server with --cert)
6. `06_showServerpubcert`: Displays the server’s public cert, for informational purposes only. (optional)

## Usage
- optionally edit `./san.snf` (ie. use different local IPs, hostname other than localhost?)
- you make a CA only once ever(in theory), so run `./01_makeCA` then `./02_makeCApubcertSS`
- make a new server private key(can run this as often as you want, afterwards): `./03_genServerPrivKey`, makes a new `server.key` file.
- make a temporary Certificate Signing Request (CSR) `server.csr` which you'd send to the CA(which in our case is a local CA we made above) for them to sign, by running `./04_gentempCSRforServer`
- as the local CA you can now sign that CSR with own CA private key, by running `./05_signCSRwiththeCA` and thus generating the server's public cert as `server.crt` which you can look at/inspect via `./06_showServerpubcert`
- you can start your spacetimedb standalone server now like the following by passing the server's private+public keys: `spacetime start --edition standalone --listen-addr 127.1.2.3:6543 --ssl --cert ../spacetimedb-cert-gen/server.crt --key ../spacetimedb-cert-gen/server.key`
- as a client connecting via TLS to that spacetimedb standalone server you can use the CA's public cert which can verify that your server's public key was signed by that CA:
  - login example:
```bash
servernick="mine"
spacetime server remove "$servernick"
spacetime server add --no-fingerprint -d --url https://127.1.2.3:6543 "$servernick"
spacetime server list
spacetime logout
spacetime login --server-issued-login "$servernick" --cert ../../spacetimedb-cert-gen/ca.crt
```
  - publish example:
  `spacetime publish --project-path server quickstart-chat --cert ../../spacetimedb-cert-gen/ca.crt`
  - follow the logs:
`spacetime logs --follow quickstart-chat --cert ../../spacetimedb-cert-gen/ca.crt`
  - wipe the database:
    `spacetime delete quickstart-chat --cert ../../spacetimedb-cert-gen/ca.crt`

## Extra
- (optional) generically test that the server is listening as TLS and a client that has that CA in its root store(ie. trusts that local CA we made) can connect to it: `openssl s_client -connect 127.1.2.3:6543 -CAfile ../../spacetimedb-cert-gen/ca.crt ; echo $?`
- Instead of `ca.crt` you can use `server.crt` for `--cert` arg, thus you don't need a local CA, in case you just want to use a simpler way via a self-signed server cert.

## Credits
xAI's Grok3 and Grok2 were used in the making of this.

## Licenses
Triple-licensed at your option:
- Apache License 2.0 (see `LICENSE-APACHE.txt`)
- Business Source License 1.1 (see `LICENSE-BSL.txt`), switches to AGPLv3 on March 27, 2030
- MIT License (see `LICENSE-MIT.txt`)
