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

## References

- https://github.com/publicdomainrelay/compute-contract-provider-relay-digitalocean

## Data Formats

- Alice CCRFP manifest

```yaml
---
$type: "com.publicdomainrelay.ccrfp.simple.machine.manifest.v.0.0.0"
cpus: 1
mem: '512M'
disk: '10G'
network: '500G'
location:
  country: 'USA'
  region: 'west'
user_data: |
  #cloud-init
  # Setup opkssh
```

- Alice watches for bids
  - **TODO** Filter by `embed.record.cid && uri` using jq

```bash
timeout 15s uv run ~/src/digitalocean-labs/droplet-oidc-poc/src/workload_identity_oauth_reverse_proxy/firehose_to_ndjson.py | jq 'select(.collection | startswith("com.publicdomainrelay."))'
```

- Bob CCB

```yaml
---
$type: "com.publicdomainrelay.ccb.simple"
embed:
  $type: "com.publicdomainrelay.ccrfp.simple.machine.manifest.v.0.0.0"
  record:
    cid: "asdlfkjsdlkfjlasdkfqeuhoj134j3lk43lk2j4308j43n4l3n2lk3j4l32"
    uri: "at://did:plc:alice0000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/3m21312k9jnkl"
bid:
  cost: 4.00
  currency: 'USDC'
  frequency: 'monthly'
  prepay: true
  x402:
    base_url: 'https://builder.bob.example.com/ccr/{at}/{cid}'
    # TODO This says, use the uri and cid of this ccb in the URL path params
    # path:
    #   at: '$this.uri'
    #   cid: '$this.cid'
# Workload Identity Federation details for when service comes up to get secrets
wif:
  issuer_uri: 'https://builder.bob.example.com'
  subject:
    format: 'ccr-cid:{cid}:ccr-at:{at}'
```

- Alice chooses and pays

```bash
$ npx awal x402 pay https://spindle-0001.johnandersen777.bsky.social.fedproxy.com/weather
✓ Request completed (HTTP 200)

Response:
{
  "report": {
    "weather": "sunny",
    "temperature": 70
  }
}
$ npx awal auth login johnandersenpdx@gmail.com
$ https://github.com/googleworkspace/cli get emails
$ npx awal auth verify $CODE
# echo npx awal@latest show
# ✓ Wallet window opened
$ npx awal address
EVM (Base): 0x9012310923809128309182903812093801923211
Solana: Fs10238091283091283098109283091283928010101
$ npx awal balance

Base
────────────────────────
USDC    5.00
ETH     0.00

Polygon
────────────────────────
USDC    0.00
POL     0.00

Solana
────────────────────────
USDC    0.00
SOL     0.00
```

- Bob CCR (Compute Contract Receipt) at createRecord response returned from
  payment.base_url on x402 success which resolves to this record:

```yaml
---
$type: "com.publicdomainrelay.ccr.simple"
rfp:
  $type: "com.publicdomainrelay.ccrfp.simple.machine.manifest.v.0.0.0"
  record:
    cid: "asdlfkjsdlkfjlasdkfqeuhoj134j3lk43lk2j4308j43n4l3n2lk3j4l32"
    uri: "at://did:plc:alice0000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/3m21312k9jnkl"
bid:
  $type: com.publicdomainrelay.ccb
  record:
    cid: "7hvb3njk42348nlk4jh5njhlkjhkdfjsdbfsjfje92yh7yhd98sf98d0sus"
    uri: "at://did:plc:bob000000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/31983y1jkdhsa"
```

- Bob CCR (Compute Contract Event) at createRecord response returned from
  payment.base_url on x402 success which resolves to this record:

```yaml
---
$type: 'com.publicdomainrelay.ccbap.simple.v.0.0.0"
ccrfp:
  $type: "com.publicdomainrelay.ccrfp.simple.machine.manifest.v.0.0.0"
  record:
    cid: "asdlfkjsdlkfjlasdkfqeuhoj134j3lk43lk2j4308j43n4l3n2lk3j4l32"
    uri: "at://did:plc:alice0000000000000000000/com.publicdomainrelay.ccrfp.simple.abstract.manifest.v.0.0.0/3m21312k9jnkl"
bid:
  $type: com.publicdomainrelay.ccb
  record:
    cid: "7hvb3njk42348nlk4jh5njhlkjhkdfjsdbfsjfje92yh7yhd98sf98d0sus"
    uri: "at://did:plc:bob000000000000000000000/com.publicdomainrelay.ccb/js9df8jo2j32l"
ccr:
  $type: com.publicdomainrelay.ccr
  record:
    cid: "dfsknml1823j12k3m1l2jn31288j12k3jkl3n439j41pk32m8sdjfoisdjf"
    uri: "at://did:plc:bob000000000000000000000/com.publicdomainrelay.ccr/3kjsdf98sdf89"
compute:
  # The IPv4 address of the provisioned compute
  ipv4: '1.1.1.1'
```

## Examples

Read CCRFPs from the firehose

```bash
uv run ~/src/digitalocean-labs/droplet-oidc-poc/src/workload_identity_oauth_reverse_proxy/firehose_to_ndjson.py alice.example.com | jq
```

**request.json**

```json
{
  "repo": "did:plc:alice0000000000000000000",
  "collection": "com.publicdomainrelay.ccrfp",
  "record": {
    "$type": "com.publicdomainrelay.ccrfp",
    "cpus": 1,
    "mem": "512M",
    "disk": "10G",
    "network": "500G"
  }
}
```

```bash
goat get $(goat xrpc procedure @pds com.atproto.repo.createRecord - < request.json  | tee response.json | jq -r '.uri')
```

```json
{
  "repo": "did:plc:alice0000000000000000000",
  "handle": "alice.example.com",
  "seq": 29814868114,
  "time": "2026-05-07T03:26:47.466Z",
  "action": "create",
  "collection": "com.publicdomainrelay.ccrfp",
  "rkey": "3mlabgut5c62t",
  "uri": "at://did:plc:alice0000000000000000000/com.publicdomainrelay.ccrfp/3mlabgut5c62t",
  "record_type": "unknown",
  "record": {
    "mem": "512M",
    "cpus": 1,
    "disk": "10G",
    "$type": "com.publicdomainrelay.ccrfp",
    "network": "500G"
  }
}
```

## Testing

```bash
$ (set -x; for dir in $(ls examples/data/spin-droplet-0001/); do file="examples/data/spin-droplet-0001/${dir}/request.json"; goat xrpc procedure @pds com.atproto.repo.createRecord - < "${file}" | tee "$(dirname "${file}")/response.json" | yq -P; done)
++ find examples/data/spin-droplet-0001/ -type f -name request.json
+ for file in $(find examples/data/spin-droplet-0001/ -type f -name request.json)
+ goat xrpc procedure @pds com.atproto.repo.createRecord -
+ yq -P
++ dirname examples/data/spin-droplet-0001/0001-ccrfp/request.json
+ tee examples/data/spin-droplet-0001/0001-ccrfp/response.json
uri: at://did:plc:5svqtrhheairglgiiyvutzik/com.publicdomainrelay.ccrfp/3mlabxf5xxg2t
cid: bafyreiblivinfkc2hqhoe367b5ggdlieyviyun652g7qxn2p2rl4orfpsq
commit:
  cid: bafyreibszjqfmvk6nbrdofqjkwvrvcdscphoidun6yxrtm55quhaylj62a
  rev: 3mlabxf65sw2t
validationStatus: unknown
```

## Notes

- https://keripy.readthedocs.io/en/latest/ref/getting_started/#receipts
- Should we "just" use TCP
