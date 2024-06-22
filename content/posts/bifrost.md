+++
title = 'Bifrost - A no-frills mTLS authentication system'
date = 2024-06-20T21:39:06+05:30
tags = ['mTLS', 'authentication', 'golang']
tldr = 'Bifrost brings simple mTLS authentication to Go applications.'
mermaid = true
+++

[Bifrost](https://github.com/RealImage/bifrost) is an [mTLS](https://www.cloudflare.com/en-in/learning/access-management/what-is-mutual-tls/)
authentication system comprised of an HTTP Certificate Authority (CA) server,
Go client library, and Go server middleware.

<!--more-->

## Wot it be

With the jargon out of the way, we can jump into how Bifrost came to be
and what it actually is.

The idea behind Bifrost is to provide clients a mechanism to create
unique identities and register with a central authority, without having to
provide sensitive information like passwords or API keys.

Say you have a fleet of servers that you dispatch off to your partners.
These servers are running your Operating System and applications
in partner networks far away from home.
You need to uniquely identify each one and map them to the client they're
working on behalf of.

Enter Bifrost.

Application clients generate key pairs and start talking to API servers over mTLS.
Servers store all the client UUIDs they see.
The first time your server sees a UUID, that ID is still untrusted, so the client
doesn't have access to sensitive APIs.
Along comes a trusted operator who tells your server that UUID xyz belongs to
partner foo.
The application client is now trusted based on its UUID alone and is free to
talk to more trusted APIs.

## You (probably) don't want this

We built Bifrost to fit in a niche and solve a specific problem.
Here are some reasons for you to not use it:

* It works for us folks over at [Qube Wire](https://qubewire.com).
It might not for you.
* The [OAuth 2.0 Device Authorization Grant](https://oauth.net/2/device-flow/)
(OAuth Device Flow) might be more suited to authorise devices and assign
them to users.
* [SmallStep CA](https://github.com/smallstep/certificates)
or [ACME](https://datatracker.ietf.org/doc/html/rfc8555/)
might suit your PKI needs better. Do your homework!
* Your friendly neighbourhood cloud provider might have an offering
that gets your audit department going.
If you need regulatory compliance, this ain't it chief.

## How it works

You're sure you want to use Bifrost?
You're plain curious? None of my business‽
OK.
Here's how it comes together.

1. Clients generate a key pair and fetch a signed TLS certificate from the CA server.
2. They "register" their UUIDs by talking to servers that trust the same CA, over
mTLS.
3. Operators use an out-of-band mechanism to verify and trust client UUIDs.
4. Trusted clients continue talking over mTLS to access privileged routes.

{{<mermaid>}}
sequenceDiagram
    actor Alice
    Note left of Alice: Client
    participant CA
    participant API
    actor Bob
    Note right of Bob: Trusted operator
    Alice->>+CA: CSR
    CA->>-Alice: Certificate
    Alice->>API: Register UUID from certificate
    Bob->>API: Trust UUID
    Alice->>+API: Authenticated & encrypted request
    API->>-Alice: Authenticated & encrypted response
{{</mermaid>}}

## Bifrost UUIDs

Bifrost recognises clients by their ECDSA P-256 key pairs.
A client's UUID is the hash of the public key and the namespace.
The namespace is any UUID that identifies a domain or application.

```go
// namespace is a UUID
// pubkey is an ecdsa.PublicKey of curve P-256

var buf [64]byte
pubkey.X.FillBytes(buf[:32])
pubkey.Y.FillBytes(buf[32:])
// buf now contains the bytes of the X and Y coordinates of the public key,
// one after the other in big-endian order.

// The UUID is the SHA-1 hash of the namespace and the public key bytes.
id := uuid.NewSHA1(namespace, buf[:])

fmt.Println(id)
// Output:
// 6ba7b810-9dad-11d1-80b4-00c04fd430c8  # for example
```

A key pair maps strongly to a UUID.
Systems using Bifrost can identify clients by their UUIDs alone,
without needing to store or send public keys.

## Certificate Authority Server

Bifrost CA server is a plain HTTP server that responds to X.509 Certificate
Signing Requests (CSRs) sent via POST requests.
The server validates CSRs, signs them, and returns signed certificates to clients.

The server is stateless and doesn't store any information about clients.
The server is also unauthenticated, meaning that anyone can request a certificate.

Operators can secure access to the server in one of two ways:

1. Place it behind a network-based protection mechanism (reverse proxy, secure gateway, firewall); or

2. Implement a [`tinyca.Gauntlet`](https://pkg.go.dev/github.com/RealImage/bifrost@v1.20.1/tinyca#Gauntlet)
function and build it into a custom binary.

A custom `Gauntlet` function can also customise client certificate fields.

A standard Bifrost certificate contains the client's UUID in the Subject
Common Name (CN) field, and the Namespace in the Organization (O) field.

The Bifrost CA certificate must also follow this pattern, so the Issuer
Common Name (CN) field is the UUID of the CA certificate, and the
Organization (O) field is the UUID again.

### Typical Bifrost Certificate

Namespace: `01881c8c-e2e1-4950-9dee-3a9558c6c741`

Server UUID: `033fc353-f618-5c18-acd1-f9d4313cc052`

Client UUID: `f6057aa6-6553-586a-9fda-319faa78958f`

```console
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 6573843113666499538 (0x5b3afb7b6f3d53d2)
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: O=01881c8c-e2e1-4950-9dee-3a9558c6c741, CN=033fc353-f618-5c18-acd1-f9d4313cc052
        Validity
            Not Before: Jun 12 15:08:54 2024 GMT
            Not After : Jun 12 16:08:54 2024 GMT
        Subject: O=01881c8c-e2e1-4950-9dee-3a9558c6c741, CN=f6057aa6-6553-586a-9fda-319faa78958f
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:7a:88:ce:51:88:ac:8e:75:a4:17:79:0b:fe:6c:
                    ab:0c:89:be:fb:66:d7:e0:b2:b3:ec:e3:5d:02:4a:
                    cc:04:24:36:1f:33:64:8f:4d:61:aa:0a:ef:44:c3:
                    7b:60:7b:7d:48:ab:89:36:eb:d0:90:6e:d6:c1:78:
                    e7:52:82:9e:7f
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:BD:BE:8A:D6:16:0A:08:46:01:27:71:25:42:04:60:DE:8C:23:8E:B3

    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:21:00:f7:dd:97:18:ef:ec:95:e0:88:6e:d7:93:66:
         74:ca:4f:96:fe:34:b1:f8:0b:90:65:c0:bc:08:a3:49:fc:8f:
         37:02:20:6d:6a:fe:b5:d1:ab:77:59:3a:d1:94:6c:4c:f7:a2:
         3d:7f:69:a8:5e:85:52:aa:6b:7e:35:c4:9f:7e:11:92:d2
```

## Go Library

The Go library provides a simple API to generate key pairs,
fetch certificates, and make authenticated requests.

### Client Library

The [`bifrost`](https://pkg.go.dev/github.com/RealImage/bifrost) package
defines Bifrost wrapper types around Go `ecdsa.PublicKey`, `ecdsa.PrivateKey`,
`x509.Certificate`, and `x509.CertificateRequest` types.

The wrappers serve two functions.
They provide storage for the UUID and namespace, and they implement
marshal and unmarshal methods for JSON, DynamoDB, text, and binary formats.

Clients can use `bifrost.HTTPClient` to create an `http.Client` that fetches
certificates from the CA server on demand.

```go
package main

import (
    "fmt"
    "io"
    "log"

    "github.com/RealImage/bifrost"
    "github.com/google/uuid"
)

const (
    caUrl  = "https://bifrost-ca.example.com"
    apiUrl = "https://api.example.com"
)

var namespace = uuid.MustParse("01881c8c-e2e1-4950-9dee-3a9558c6c741")

func main() {
    // Generate a new private key.
    // Store this key securely and load it on startup.
    key, err := bifrost.NewPrivateKey()
    if err != nil {
        log.Fatal(err)
    }

    // bifrost.HTTPClient returns an http.Client which fetches certificates
    // from caUrl on demand.
    client, err := bifrost.HTTPClient(caUrl, namespace, key, nil, nil)
    if err != nil {
        log.Fatal(err)
    }

    // Make an authenticated request.
    resp, err := client.Get(apiUrl)
    if err != nil {
        log.Fatal(err)
    }
    defer resp.Body.Close()

    // Read the response body.
    body, err := io.ReadAll(resp.Body)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(string(body))
}
```

### Server Middleware

The [`asgard`](https://pkg.go.dev/github.com/RealImage/bifrost/asgard) package
provides two Go HTTP middleware functions to identify and authenticate clients.

[`asgard.Heimdallr`](https://pkg.go.dev/github.com/RealImage/bifrost/asgard#Heimdallr)
parses client certificates from request headers and stores them in the request context.
Use this in HTTP API servers that are behind TLS terminating proxies.

[`asgard.Hofund`](https://pkg.go.dev/github.com/RealImage/bifrost/asgard#Hofund)
parses client certificates from TLS connections and stores them in the request context.
Use this if you're terminating TLS in your Go server.

```go
package main

import (
    "fmt"
    "net/http"

    "github.com/RealImage/bifrost/asgard"
    "github.com/google/uuid"
)

var namespace = uuid.MustParse("01881c8c-e2e1-4950-9dee-3a9558c6c741")

func handler(w http.ResponseWriter, r *http.Request) {
    // Get the client UUID from the request context.
    cert, ok := asgard.ClientCert(r.Context())
    if !ok {
        http.Error(w, "Unauthorized", http.StatusUnauthorized)
        return
    }

    fmt.Fprintf(w, "Hello, %s!", cert.ID)
}

func main() {
    auth := asgard.Heimdallr(asgard.HeaderNameClientCertLeaf, namespace)
    http.Handle("/", auth(http.HandlerFunc(handler)))
    http.ListenAndServe(":8080", nil)
}
```

### Configuration

The [`cafiles`](https://pkg.go.dev/github.com/RealImage/bifrost/cafiles) package
provides functions to load and parse Bifrost CA certificates from PEM files.
It reads them from disk, AWS S3, or AWS secrets manager.

---

And that's all there is to Bifrost.
