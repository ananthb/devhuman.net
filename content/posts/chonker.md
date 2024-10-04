+++
title = 'Chonker - Fast S3 file downloads'
date = 2024-03-17T03:22:32+05:30
tags = ['go', 's3', 'http']
tldr = 'Chonker makes S3 file downloads fast using HTTP range requests.'
+++

Chonker speeds up large file downloads from S3
by fetching files in chunks using HTTP range requests.

<!--more-->

## wot it be

DDOS-ing your upstream file server is generally not a good idea.
AWS S3's horizontal scaling design essentially mandates that you
use parallel network requests to fully saturate available bandwidth.
To that end, the AWS CLI itself makes ten parallel requests to S3 out of the box.
Chonker brings that same experience to your Go programs.

[Chonker](https://github.com/ananthb/chonker) is a rewrite
of a now non-existent project called ranger written by the venerable
[@sudhirj](https://github.com/sudhirj).
It doesn't exist there for boring `$job` reasons,
but the original code lives on in Chonker's git history.

The standard Go HTTP request response API lives on in Chonker whereby
a single HTTP request yields a single HTTP response object.
Clients read off this response's body blissfully unaware that
chonker is fetching chunks and piping them to the `io.Reader` in order.

## vroooom

Go made creating Chonker an absolute delight.
An `io.Reader` is connected to an `io.Writer` interface
by an `io.Pipe`. Chunk download goroutines fetch chunks
quickly and potentially out-of-order.
Writes to the pipe happen in-order ensuring that clients
see the right file.
Faking a `http.Response` for the entire file instead of
each chunk is straight forward as well.

## how to use

```go
import (
    "net/http"
    
    "github.com/ananthb/chonker"
)

func main() {
    // Create a new Chonker client
    client := chonker.NewClient()

    // Create a new HTTP request
    req, _ := http.NewRequest("GET", "https://example.com/file", nil)

    // Fetch the file
    resp, _ := client.Do(req)

    // Read the file
    body, _ := io.ReadAll(resp.Body)
    fmt.Println(string(body))
}
```

Chonker ends up being a drop-in replacement in any code that
expects a `http.Client`. If you're not pointing your clients
at S3/CloudFront, make sure that your backend server is okay
with clients opening more than one connection to fetch the same file.

## AWS CloudFront Weirdness

AWS CloudFront may be at odds with IETF RFC 7231
[Section 4.3.2](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2)
which states that servers should response with HTTP HEAD requests as if they
were get requests. The server just has to omit the response body in this case.

If you have a file in CloudFront larger than 25GiB, then GET requests unadorned
with a `Range` header will fail with a 400 error. This has the side effect of
also making HEAD requests fail.
Chonker works around this by always sending a `Range` header with a 0-0 range
for the first discovery request, but this leaves a strange feeling in my gut.
I want my HEAD requests dammit!
