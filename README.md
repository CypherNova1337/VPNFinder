# VPNFinder

Finds an organisation's remote-access gateways and identifies exactly what they're running.

![license](https://img.shields.io/badge/license-MIT-blue?style=flat-square)
![python](https://img.shields.io/badge/python-3.8%2B-3776AB?style=flat-square)

## What it does

Every organisation with remote workers has a way in — a FortiGate, a
GlobalProtect portal, a Cisco ASA, a Citrix gateway, a Pulse/Ivanti box. These
are the front door to the internal network, and they have a long history of
serious, actively exploited vulnerabilities.

Finding them is the easy half. The hard half is knowing *which product* you're
looking at, because that determines whether the CVE you're thinking of applies.
A login page that says nothing and a generic certificate can be any of a dozen
appliances, and guessing wrong wastes the engagement.

VPNFinder does both. It sweeps the target's hosts for remote-access services,
then fingerprints each one to name the actual product, with a confidence score
and the evidence behind it. It maps what it identifies to the CVEs relevant to
that product, so the output is a prioritised list rather than a pile of
addresses.

When the target is behind a CDN or WAF, it also works to recover the real origin
address, because the gateway is frequently reachable directly even when the
website in front of it isn't.

## Why you'd use it

- **Names the product**, rather than reporting "something VPN-shaped here."
- **Scores with evidence**, so you can see why it reached a conclusion.
- **Flags relevant CVEs** for the product it identified.
- **Sees past a CDN** to the origin where the gateway usually lives.
- **Hands off to nmap and ffuf** when you want to go deeper on what it found.

## Install

```bash
git clone https://github.com/CypherNova1337/VPNFinder
cd VPNFinder
pip install -r requirements.txt
chmod +x vpn-finder.py
```

Needs Python 3.8 or newer. `nmap` and `ffuf` are optional and only used with
their flags.

## Usage

```bash
python3 vpn-finder.py example.com
```

Discovers candidate hosts, fingerprints what answers, and reports what it found
with confidence and CVEs.

**Save the output**

```bash
python3 vpn-finder.py example.com -o results.json
```

**Only high-confidence results**

```bash
python3 vpn-finder.py example.com --min-confidence 70
```

Useful on a large estate where low-confidence noise buries the real gateways.

**Skip the noisy parts**

```bash
python3 vpn-finder.py example.com --skip-brute --skip-ports
```

Passive sources only — nothing that looks like scanning.

**Widen the search across the org's netblocks**

```bash
python3 vpn-finder.py example.com --asn-sweep --sweep-cap 4096
```

**Go deeper on what it found**

```bash
python3 vpn-finder.py example.com --nmap --ffuf
```

**Use your own wordlist**

```bash
python3 vpn-finder.py example.com -w vpn_hostnames.txt -t 50
```

## Options

| Flag | Default | What it does |
|---|---|---|
| `domain` | — | Target domain |
| `-o` | — | Write results to a file |
| `-w` | bundled | Custom hostname wordlist |
| `-t` | — | Concurrent workers |
| `--timeout` | — | Per-request timeout |
| `--min-confidence` | — | Hide results scoring below this |
| `--skip-passive` | off | Skip passive sources |
| `--skip-brute` | off | Skip hostname brute force |
| `--skip-ports` | off | Skip port scanning |
| `--asn-sweep` | off | Sweep the organisation's netblocks |
| `--sweep-cap` | — | Cap on addresses swept |
| `--no-origin` | off | Don't attempt origin discovery |
| `--force-origin` | off | Attempt origin discovery even without a CDN |
| `--nmap` | off | Run nmap against identified hosts |
| `--ffuf` | off | Run ffuf against identified hosts |
| `--ffuf-options` | — | Extra options passed to ffuf |
| `--all` | off | Report everything, including low confidence |
| `--no-color` | off | Disable coloured output |
| `--version` | — | Print version and exit |

## Good to know

- **A CVE match means the product is a candidate, not that it's vulnerable.**
  Version detection on these appliances is imprecise and many are patched in
  place. Confirm before reporting.
- **`--asn-sweep` gets big fast.** An organisation's ASN can cover a lot of
  address space. Keep `--sweep-cap` sensible.
- **Gateways are monitored.** These are the hosts a security team watches most
  closely, and brute forcing hostnames against them is very visible.
- **Don't authenticate.** Identifying a portal is reconnaissance; trying
  credentials against it is a different activity needing separate authorisation.

## Authorised use

Only against organisations you own or that are in scope for an engagement.
Remote-access infrastructure is the most sensitive thing on a perimeter and
probing it without permission will be treated as an attack.

## License

MIT — see [LICENSE](LICENSE).
