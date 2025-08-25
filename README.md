[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=for-the-badge)](https://github.com/mohamedtobgi/IPFindX/releases)

# IPFindX — Terminal IP Intelligence: Geolocation, ASN & Proxy 🔎🌐

A compact, fast command-line tool for IP intelligence. IPFindX pulls geolocation, ASN, ISP, and proxy signals. Use it for triage, network mapping, OSINT, or automation. It prints JSON and human-friendly tables. It works on Linux, macOS, and Windows.

Badges
- ![License](https://img.shields.io/badge/license-MIT-green)
- ![Language](https://img.shields.io/badge/lang-Go-blue)
- Topics: asn-lookup · command-line-tool · geolocation-api · ip-api · ip-geolocation · ip-intelligence · ip-tracker · network-tools · osint · proxy-detection

Preview image
![Network map](https://images.unsplash.com/photo-1545239351-1141bd82e8a6?ixlib=rb-4.0.3&q=80&w=1200&auto=format&fit=crop&crop=faces&sat=-100)

Table of contents
- Features
- Quick start
- Install from releases
- Install from source
- Usage
- Examples
- Output fields
- Detection methods
- Data sources
- Performance and limits
- Integration tips
- Contributing
- License
- Releases

Features
- Single binary CLI. No heavy deps.
- Geolocation (country, region, city, coordinates).
- ASN lookup and prefix mapping.
- ISP and organization resolution.
- Proxy and VPN detection heuristics.
- Passive and active checks: WHOIS, Team Cymru, HTTP headers.
- Batch mode for lists and CIDR ranges.
- JSON, table, CSV outputs.
- Quiet mode for automation.
- Color and no-color modes.

Quick start

1. Download a release binary from the Releases page and run it.
2. Run a single IP lookup:
```
./ipfindx lookup 8.8.8.8
```

Install from releases

Download the appropriate binary file from the releases page and execute it. The releases page contains prebuilt binaries and archives for common platforms. Pick the file that matches your OS and CPU architecture, download it, make it executable, and run.

Example:
```
# Linux x86_64
wget https://github.com/mohamedtobgi/IPFindX/releases/download/v1.2.0/ipfindx-linux-amd64.tar.gz
tar -xzf ipfindx-linux-amd64.tar.gz
chmod +x ipfindx
./ipfindx lookup 1.1.1.1
```

Visit the releases page to get the file:
https://github.com/mohamedtobgi/IPFindX/releases

Install from source

If you want to build from source, follow these steps. These commands assume Go is installed.

```
git clone https://github.com/mohamedtobgi/IPFindX.git
cd IPFindX
go build -o ipfindx ./cmd/ipfindx
./ipfindx --help
```

Usage

IPFindX uses subcommands and flags. The CLI focuses on clear commands and stable output formats.

Primary commands
- lookup <IP|host> — Lookup a single IP or domain.
- batch <file> — Process a newline list of IPs/hosts.
- cidr <CIDR> — Expand and scan a CIDR range.
- whois <IP|ASN> — WHOIS or ASN details.
- version — Show version and data timestamps.

Global flags
- -o, --output [json|table|csv]  Output format (default: table)
- -q, --quiet                    Minimal output for scripts
- -t, --timeout <seconds>        Network timeout
- --no-color                     Disable color output
- --probe-ports <list>           Probe ports (comma separated)
- --as-lookup                    Force ASN lookup

Examples

Single IP
```
./ipfindx lookup 8.8.8.8
```

Domain
```
./ipfindx lookup example.com
```

Batch mode (file of IPs)
```
./ipfindx batch ips.txt -o json > ips.json
```

CIDR scan
```
./ipfindx cidr 192.0.2.0/28 -o csv > block.csv
```

ASN lookup
```
./ipfindx whois AS15169
```

Probe specific ports
```
./ipfindx lookup 198.51.100.42 --probe-ports 80,443
```

Script friendly
```
ips="8.8.8.8 1.1.1.1"
for ip in $ips; do
  ./ipfindx lookup $ip -o json | jq '.ip, .asn, .country'
done
```

Sample output (table)
```
IP           ASN      ISP                Country   City         Proxy
8.8.8.8      AS15169  Google LLC         US        Mountain View  no
1.1.1.1      AS13335  Cloudflare, Inc.   US        San Francisco  maybe
```

Sample output (json)
```
{
  "ip": "8.8.8.8",
  "asn": "AS15169",
  "asn_org": "Google LLC",
  "prefix": "8.8.8.0/24",
  "country": "US",
  "region": "California",
  "city": "Mountain View",
  "latitude": 37.386,
  "longitude": -122.0838,
  "isp": "Google LLC",
  "proxy": false,
  "last_seen": "2025-06-01T12:34:56Z"
}
```

Output fields

- ip: Queried IP address.
- asn: Autonomous System Number (ASnnn).
- asn_org: ASN organization name.
- prefix: Announced prefix that contains the IP.
- isp: Service provider name.
- country, region, city: Geolocation fields.
- latitude, longitude: Decimal degrees.
- proxy: false|maybe|true — proxy detection result.
- last_seen: Last observed timestamp in data sources.
- ports: Detected open ports (when probed).
- raw: Raw provider responses when --output json contains a raw block.

Detection methods

IPFindX combines multiple detection techniques. It merges passive and active signals to increase accuracy.

Geolocation
- Use multiple IP geolocation providers and apply a consensus algorithm.
- Prefer data from regional registries and validated datasets.
- Fall back to WHOIS prefix location if providers disagree.

ASN and prefix
- Query Team Cymru and public BGP feeds.
- Cross-check with RIPE/ARIN/AFRINIC/LACNIC APNIC registries.
- Resolve the most specific prefix covering the IP.

ISP and org
- Resolve org from ASN, whois, and reverse DNS.
- Use provider heuristics to collapse subsidiaries to their parent org.

Proxy and VPN detection
- Check provider lists from commercial proxy-detection APIs.
- Inspect HTTP headers via a lightweight probe when allowed.
- Compare IP to cloud provider ranges and known data center prefixes.
- Consider port patterns and TLS certificate common names.
- Mark detection as:
  - true: strong signals from multiple sources.
  - maybe: mixed signals or weak evidence.
  - false: no proxy signals.

Active probes
- Optional TCP connect to common ports (80,443).
- TLS handshake capture to inspect SNI and cert issuer.
- HTTP header probe to detect proxy server headers (X-Forwarded-For, Via).

Scoring and confidence
- IPFindX computes a confidence score per field.
- Confidence factors:
  - Number of agreeing providers.
  - Freshness of dataset.
  - Presence of matching BGP and whois data.
- The CLI exposes confidence in JSON.

Data sources

IPFindX aggregates multiple public and private sources:
- Team Cymru IP->ASN data
- Regional Internet Registries (ARIN, RIPE, APNIC, LACNIC, AFRINIC)
- Public geolocation providers (open databases and commercial APIs)
- Passive DNS and certificate transparency logs
- Proxy detection lists and community blocklists
- HTTP and TLS probe results

You can configure which providers the tool queries via config file or environment variables. The config file supports API keys for compatible providers.

Performance and limits

- Single lookup completes in under 300 ms on a fast connection when cache hits occur.
- Batch mode processes thousands of IPs per minute, depending on network and probes.
- Respect rate limits of third-party providers. IPFindX supports local caching and backoff.
- When you download a release binary, make it executable and run with proper permissions.

Integration tips

- Use JSON output for ingestion into SIEM, scripts, or databases.
- Run batch lookups overnight to populate local caches.
- Combine CIDR expansion with ASN queries to map networks.
- Use --no-color in CI environments.
- Use -q in scripts to reduce noise.

Example integration with jq
```
./ipfindx batch ips.txt -o json | jq -r '. | "\(.ip),\(.asn),\(.country),\(.proxy)"' > report.csv
```

Config file

The config file lives at ~/.config/ipfindx/config.yaml by default. Example:
```
providers:
  geolocation:
    - type: freegeoip
    - type: maxmind
      database: /path/to/GeoLite2-City.mmdb
  proxy_detection:
    - type: ip-reputation
      api_key: YOUR_KEY
cache:
  enabled: true
  ttl: 86400
timeout: 5
```

Contributing

- Fork the repo.
- Create a feature branch.
- Add tests for new features.
- Send a pull request.
- Follow the code style and lint rules.
- Open issues for bugs and feature requests.

Development

- Code lives in /cmd and /pkg with a clean separation of CLI and core libs.
- Use make for common tasks:
```
make build
make test
make fmt
```
- Tests run with go test ./...

License

IPFindX uses an MIT license. See LICENSE file in the repo.

Releases and downloads

Download the latest binary or source archive from the releases page. Pick the file for your platform, download it, and execute the binary or extract the archive. Example: download the binary, set executable bit, then run.

Releases page:
[![Get Releases](https://img.shields.io/badge/Get%20Releases-Click%20Here-orange?logo=github&style=for-the-badge)](https://github.com/mohamedtobgi/IPFindX/releases)

Changelog

Each release on the releases page includes a changelog entry. Look at release notes for breaking changes and migration tips.

Common CLI recipes

Scan a whole ASN
```
./ipfindx whois AS396982 -o json | jq '.prefixes[] | .cidr' | while read cidr; do ./ipfindx cidr $cidr -o csv >> as_scan.csv; done
```

Detect likely proxies in a list
```
./ipfindx batch ips.txt -o json | jq 'select(.proxy=="true") | .ip' > proxies.txt
```

Automated daily update (cron)
```
0 3 * * * /usr/local/bin/ipfindx update-data
```

Support

Open an issue on GitHub. Provide sample output, commands, and environment details for faster triage.

Project roadmap

Planned items:
- Enrich proxy signals with ML model.
- Add dynamic rate limiting per provider.
- Add Windows native installer.
- Expand CI tests for cross-platform builds.

Contact and references

- Repository: https://github.com/mohamedtobgi/IPFindX
- Releases: https://github.com/mohamedtobgi/IPFindX/releases

This README provides the commands and background to start using IPFindX from the terminal. Follow the releases page to get official binaries and release notes.