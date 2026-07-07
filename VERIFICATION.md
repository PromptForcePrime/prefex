# Verification — what our badges mean

prefex ships as a **binary**; the source is private. That makes "how do I
trust this?" a fair question, so every badge on this README is designed to be
independently checkable. Here is exactly what each one asserts — and what it
doesn't.

## Release verified (GitHub Actions, this repo)
Public CI you can click into. On every release and nightly, it downloads the
published binaries onto clean macOS/Linux/Windows runners, verifies the
SHA256 checksums, and smoke-tests them (`prefex -version`, read-only
`preview`, daemon start + `/health` + stop). It proves the artifact you
download is the artifact we published, and that it runs.

## Internal suite / coverage (shields endpoint)
Our private CI runs the full Go test suite on every push to main and
publishes two aggregate numbers here — test count and statement coverage —
to the `badges` branch of this repo. You cannot see the tests themselves;
you can see whether they pass, how many there are, how covered the code is,
and the timestamped commit history of every update. A red badge means the
suite is failing and we shipped anyway — call us on it.

## Checksums
Every release includes `SHA256SUMS`. Verify before running:

```bash
shasum -a 256 -c SHA256SUMS --ignore-missing   # macOS
sha256sum -c SHA256SUMS --ignore-missing        # Linux
```

## Signed releases (roadmap → cosign)
We are moving release builds into CI with Sigstore keyless signing and GitHub
artifact attestations, so provenance becomes verifiable even with private
source: `gh attestation verify <binary> --owner PromptForcePrime`. Until that
lands, checksums + release-verify CI are the chain of custody.

## Cache-safe
Our own claim, self-policed: every transform prefex applies to a request is
byte-deterministic inside the provider's cached prefix, and the proxy runs a
full-history divergence detector plus real-time sentinels that alert (with
dollar figures) if any layer ever violates that. The methodology is described
in the docs; your own dashboard shows the detectors' output on your traffic.

## Trust
[TRUST.md](TRUST.md): what data prefex touches, what never leaves your
machine (everything), and how to verify that with a network monitor.

## What we can't show you
The source. In exchange, nothing here asks for faith: artifacts are
checksummed and CI-verified in public, quality numbers update from real runs
with history, and the strongest claims (cache-safety, savings) are measurable
on your own machine, on your own traffic, in your own dashboard — which is
where they belong anyway, because savings are workload-dependent and anyone
promising a fixed percentage is selling you their benchmark, not your bill.
