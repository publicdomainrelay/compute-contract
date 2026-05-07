# Compute Contract

- Alice, Bob, and Eve are on the network
- Alice wants to issue a Compute Contract Request For Proposal (CCRFP)
  - Alice's CCRFP will state she wants an OpenCode instance
- Bob has plenty of builder machines
- Eve wants to know what Alice is doing
- Alice has vouched for Bob
- Alice has denounced Eve
- Alice creates a CCRFP manifest
- Alice makes her CCRFP manifest available to the network
- Bob and Eve each issue a Compute Contract Bid (CCB) against the CCRFP
- Alice's policy engine sees that she's denounced Eve and vouched for Bob
- (Optionally) Alice issues a Compute Contract Bid Accept Payment (CCBAP)
  against Bob's CCB.
- Alice issues a Compute Contract Bid Accept (CCBA) against Bob's CCB
- Bob builds to the CCRFP manifest's spec
- Bob makes a Compute Contract Event (CCE) and makes it available to the network
  - The event is the `heartbeat=1` event. Indicating the compute has entered a
    state where it is now functional.
- Alice confirms the compute is functional
- Alice issues a Compute Contract Event Accept (CCEA) on the network
- Bob issues a Compute Contract Event (CCE) on the network
  - The event is the `heartbeat=2` event.
- Alice is done using the compute
- Alice issues the Compute Contract Finalize (CCF) event to the network
- Bob issues a Compute Contract Event (CCE) on the network
  - The event is the `heartbeat=0` event. The compute contract is now complete.
- Compensation can be tied to these heartbeat events.

## Data Formats

- Alice CCRFP manifest

```yaml
---
$type: "com.publicdomainrelay.ccrfp.simple.machine.manifest.v.0.0.0"
cpus: 1
mem: '512M'
disk: '10G'
network: '500G'
```

- Bob CCB

```yaml
---
$type: "com.publicdomainrelay.ccb.simple.time.monthly.v.0.0.0"
embed:
  $type: "com.publicdomainrelay.ccrfp.simple.machine.manifest.v.0.0.0"
  record:
    cid: "asdlfkjsdlkfjlasdkfqeuhoj134j3lk43lk2j4308j43n4l3n2lk3j4l32"
    uri: "at://did:plc:alice0000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/3m21312k9jnkl"
cost: 4.00
currency: 'USDC'
prepay: true
```

- Alice CCBP

```yaml
---
$type: 'com.publicdomainrelay.ccbap.simple.v.0.0.0"
embed:
  $type: "com.publicdomainrelay.ccb.simple.time.monthly.v.0.0.0"
  record:
    cid: "7hvb3njk42348nlk4jh5njhlkjhkdfjsdbfsjfje92yh7yhd98sf98d0sus"
    uri: "at://did:plc:bob000000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/31983y1jkdhsa"
txid: "${TXID}"
```

- Alice CCBA

```yaml
---
$type: 'com.publicdomainrelay.ccba.simple.v.0.0.0"
embed:
  $type: "com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0"
  record:
    cid: "7hvb3njk42348nlk4jh5njhlkjhkdfjsdbfsjfje92yh7yhd98sf98d0sus"
    uri: "at://did:plc:bob000000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/31983y1jkdhsa"
payment:
  embed:
    $type: "com.publicdomainrelay.ccbap.simple.abstract.manifest.v.0.0.0"
    record:
      cid: "k78sfdjk01jek1j23012j31i2l3jkjdsh9sjdkfl1jh4j1j3l2k1jn9djlk"
      uri: "at://did:plc:alice0000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/3masdfj1301jl"
```

## Notes

- https://keripy.readthedocs.io/en/latest/ref/getting_started/#receipts
- Should we "just" use TCP
