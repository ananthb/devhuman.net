+++
title = 'Bifrost - A no-frills mTLS authentication system'
date = 2023-11-04T21:39:06+05:30
draft = true
tags = ['mTLS', 'authentication']
+++

Bifrost is an mTLS authentication system comprised of an HTTP based
Certificate Authority (CA) accompanied by Go library code.

<!--more-->

Its purpose built for use with AWS API Gateway instances set up for mTLS
authentication. The Bifrost CA issues certificates to clients over HTTP, responding
to valid Certificate Signing Requests (CSRs) with signed certificates.
CA accepts CSRs either as PEM encoded ASN.1 DER data or ASN.1 DER binary data itself.
It does not authenticate clients.
Deployed by itself, it will issue a certificate to anyone.
