# rust-ccatoken

`rust-ccatoken` is an implementation of the Arm CCA attestation token (§A.7 of the [Realm Management Monitor (RMM) Specification](https://developer.arm.com/documentation/den0137/latest)) in Rust.


The library implements interfaces to:

* Decode a CBOR-encoded CCA token
* Verify the CCA token (Platform, Realm and their binding)
* Appraise CCA evidence using user-supplied reference values and endorsements


## `ccatoken` CLI

Alongside the library code, this crate provides a CLI to manipulate CCA tokens.

All the examples below assume all paths are relative to the root of this repository, and that the `ccatoken` executable is reachable via the shell `PATH`.  I.e.:
```sh
export PATH=$PATH:"$PWD/target/debug"
```

### `ccatoken golden`

The `golden` command creates reference values and trust anchor for the given token and CPAK.
If the token is not successfully verified with CPAK no values are extracted.

```sh
ccatoken golden \
    -e testdata/cca-token.cbor \
    -c testdata/cpak.json \
    -t golden-tastore.json \
    -r golden-rvstore.json
```

On success:
```
golden values extraction successful
```
the two "golden" stores are saved on disk.  The contents can be pretty-printed using `jq(1)` as follows:

```sh
jq . golden-*.json
```
which should produce an output similar to the following:
```json
{
  "platform": [
    {
      "implementation-id": "7f454c4602010100000000000000000003003e00010000005058000000000000",
      "sw-components": [
        {
          "measurement-value": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "signer-id": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "version": "3.4.2",
          "component-type": "BL"
        },
        {
          "measurement-value": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "signer-id": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "version": "1.2",
          "component-type": "M1"
        },
        {
          "measurement-value": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "signer-id": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "version": "1.2.3",
          "component-type": "M2"
        },
        {
          "measurement-value": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "signer-id": "07060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918",
          "version": "1",
          "component-type": "M3"
        }
      ],
      "platform-configuration": "0107060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918"
    }
  ],
  "realm": [
    {
      "initial-measurement": "0000000000000000000000000000000000000000000000000000000000000000",
      "rak-hash-algorithm": "sha-256",
      "extensible-measurements": [
        "0000000000000000000000000000000000000000000000000000000000000000",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "0000000000000000000000000000000000000000000000000000000000000000"
      ],
      "personalization-value": "54686520717569636b2062726f776e20666f78206a756d7073206f766572203133206c617a7920646f67732e54686520717569636b2062726f776e20666f7820"
    }
  ]
}
[
  {
    "pkey": {
      "crv": "P-384",
      "kty": "EC",
      "x": "IShnxS4rlQiwpCCpBWDzlNLfqiG911FP8akBr-fh94uxHU5m-Kijivp2r2oxxN6M",
      "y": "hM4tr8mWQli1P61xh3T0ViDREbF26DGOEYfbAjWjGNN7pZf-6A4OTHYqEryz6m7U"
    },
    "implementation-id": "7f454c4602010100000000000000000003003e00010000005058000000000000",
    "instance-id": "0107060504030201000f0e0d0c0b0a090817161514131211101f1e1d1c1b1a1918"
  }
]
```

### `ccatoken appraise`

The `appraise` command tries to match the supplied CCA token and reference values.

```sh
ccatoken appraise \
    -e testdata/cca-token.cbor \
    -r golden-rvstore.json
```

On successful completion, the computed trust vectors for the platform and realm are printed to stdout:
```
appraisal completed
platform trust vector: {
  "instance-identity": 2,
  "configuration": 2,
  "executables": 3,
  "hardware": 2,
  "runtime-opaque": 32
}
realm trust vector: {
  "executables": 2
}
```

### `ccatoken verify`

The `verify` command cryptographically verifies the supplied CCA token using a matching CPAK from the trust anchor store.

```sh
ccatoken verify \
    -e testdata/cca-token.cbor \
    -t golden-tastore.json
```

On successful completion, the computed trust vectors for the platform and realm are printed to stdout:
```
verification completed
platform trust vector: {
  "instance-identity": 2
}
realm trust vector: {
  "instance-identity": 2
}
```