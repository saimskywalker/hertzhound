# Security policy

## Supported versions

Fixes land on the latest minor release only.

| Version | Supported |
|---|---|
| 0.1.x | Yes |
| Older | No |

## Reporting a vulnerability

`hertzhound` reads bytes from files it cannot trust. Every read is
meant to be bounded and no input is meant to throw, so a crafted file that
makes it hang, read without bound, allocate excessively, or throw is a
security problem.

Please report privately: open this repository's **Security** tab and choose
**Report a vulnerability**. This creates a private advisory that only you and
the maintainer can see.

Do **not** open a public issue, pull request or discussion for a
vulnerability.

Useful things to include:

- the package version, `dart --version`, and whether it happens on the VM,
  on the web, or both
- the API you called (for example `sniffBytes`, a file helper, or the CLI)
- the smallest input that reproduces it, as a hex dump or a short Dart
  snippet that builds the bytes; never copyrighted audio
- what an attacker could achieve, and any fix you have in mind

## What to expect

You will usually get a first reply within a week. Once the issue is
confirmed, a fix is prepared in the private advisory, released, and the
advisory is published with credit to you unless you prefer otherwise.
