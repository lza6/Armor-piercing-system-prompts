# Recon Methodology

## Phase 1: Passive (no direct contact)

### Subdomain Enumeration
```bash
subfinder -d target.com -silent -all | sort -u > subs.txt
httpx -l subs.txt -silent -status-code -tech-detect | tee live-hosts.txt
```

### URL Discovery
```bash
gau target.com | sort -u > historical-urls.txt
waybackurls target.com | sort -u >> historical-urls.txt
grep -E "\?.*=" historical-urls.txt > params.txt
grep -iE "api|admin|internal|debug|test|staging" historical-urls.txt > interesting.txt
```

### Tech Detection
```bash
curl -sI https://target.com | grep -iE "server|x-powered|x-runtime|x-generator"
```

## Phase 2: Active (light contact)

### Content Discovery
```bash
ffuf -u https://target.com/FUZZ -w wordlists/common.txt -mc 200,301,302,403
ffuf -u https://target.com/api/FUZZ -w wordlists/api-endpoints.txt -mc 200,401,403,405
```

### JS Analysis
```bash
katana -u https://target.com -d 3 -jc -ef css,png,jpg,gif | grep "\.js$" | sort -u > js-files.txt
# Grep for secrets and endpoints in downloaded JS
grep -rohE '/api/[a-zA-Z0-9/_-]+' downloaded-js/ | sort -u
```

## Phase 3: Organize

### Key Output Files
```
recon/<target>/
├── live-hosts.txt       — verified live hosts with tech
├── urls.txt             — all discovered URLs
├── api-endpoints.txt    — API paths
├── js-analysis.txt      — secrets and endpoints from JS
├── idor-candidates.txt  — URLs with ID parameters
├── ssrf-candidates.txt  — URLs with URL/redirect parameters
└── tech-stack.txt       — technology summary
```

### Extract Candidates
```bash
# IDOR candidates:
grep -E '/[0-9]+|id=|user_id=|order_id=' urls.txt > idor-candidates.txt
# SSRF candidates:
grep -iE 'url=|uri=|redirect=|next=|dest=|callback=' urls.txt > ssrf-candidates.txt
```

After recon: run `/surface` for P1/P2/Kill ranking.
