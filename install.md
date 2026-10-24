# DomainDive - Installation Guide

This guide installs everything the DomainDive Apex Domain Reconnaissance Arsenal uses. Commands are for **Debian/Ubuntu**; adapt the package-manager lines for other distributions or macOS.

Everything that isn't a Go binary or a system package goes in `~/tools/`. The commands in `README.md` assume that location.

```bash
mkdir -p ~/tools/wordlists
```

## 1. System Packages

```bash
sudo apt update
sudo apt-get -y install git curl jq nmap sslscan python3 python3-pip pipx libpcap-dev p7zip-full
pipx ensurepath
```

`libpcap-dev` is needed to build `naabu`.

## 2. Go

Installs the current Go release (dns-helix and the other Go tools need 1.24 or newer).

```bash
GO_VERSION=$(curl -s 'https://go.dev/VERSION?m=text' | head -1)
wget "https://go.dev/dl/${GO_VERSION}.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf "${GO_VERSION}.linux-amd64.tar.gz"
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

## 3. My Tools

These are my own tools, used throughout the workflow.

```bash
# DomainDive itself (for scope_filter.py and sort.py)
git clone https://github.com/CypherNova1337/DomainDive ~/tools/DomainDive

# Go tools
go install github.com/CypherNova1337/paramvoid@latest
go install github.com/cyphernova1337/sourcemapper@latest

# Python tools installed as commands
for t in VoidRecon LFI-Machine SmartSQL VoidStrike Auto-IDOR; do
    git clone https://github.com/CypherNova1337/$t ~/tools/$t
    pipx install ~/tools/$t
done

# Python tools run from their folder
for t in Personal_Scipts NovaLure VoidOrigin; do
    git clone https://github.com/CypherNova1337/$t ~/tools/$t
    pip3 install -r ~/tools/$t/requirements.txt
done

# Nuclei templates and gf patterns
git clone https://github.com/CypherNova1337/Personal_Nuclei_Templates ~/tools/Personal_Nuclei_Templates
git clone https://github.com/CypherNova1337/Firebase_Nuclei ~/tools/Firebase_Nuclei
git clone https://github.com/CypherNova1337/GF_Patterns ~/tools/GF_Patterns
mkdir -p ~/.gf && cp ~/tools/GF_Patterns/*.json ~/.gf/

# Code Specter is a bookmarklet: copy the contents of Code Specter.js into a bookmark's URL field
git clone https://github.com/CypherNova1337/Code-Specter ~/tools/Code-Specter
```

On newer Debian/Ubuntu releases `pip3 install` outside a virtual environment is blocked. Either add `--break-system-packages`, or make a venv first (`python3 -m venv ~/tools/venv && source ~/tools/venv/bin/activate`) and run the `pip3` lines inside it.

| Tool | What it does here |
|---|---|
| `scope_filter.py` | Keeps only exact apex hosts and strips out-of-scope entries from every step's output |
| [VoidRecon](https://github.com/CypherNova1337/VoidRecon) | Full recon pipeline with enforced scope (`voidrecon setup` after install to add API keys) |
| [paramvoid](https://github.com/CypherNova1337/paramvoid) | Hidden HTTP parameter discovery |
| [sourcemapper](https://github.com/cyphernova1337/sourcemapper) | Rebuilds original JavaScript source from source maps |
| [Personal_Scipts](https://github.com/CypherNova1337/Personal_Scipts) | `webscan.py` (CORS, CRLF, hop-by-hop) and `secretscan.py` (secrets in JS) |
| [NovaLure](https://github.com/CypherNova1337/NovaLure) | Blind SSRF and open redirects via out-of-band callbacks |
| [LFI-Machine](https://github.com/CypherNova1337/LFI-Machine) | Adaptive local file inclusion |
| [SmartSQL](https://github.com/CypherNova1337/SmartSQL) | WAF-aware SQL injection |
| [VoidStrike](https://github.com/CypherNova1337/VoidStrike) | RCE, SSTI, command injection, deserialisation |
| [Auto-IDOR](https://github.com/CypherNova1337/Auto-IDOR) | IDOR testing with two real accounts |
| [VoidOrigin](https://github.com/CypherNova1337/VoidOrigin) | Origin IP discovery behind a CDN |
| [Personal_Nuclei_Templates](https://github.com/CypherNova1337/Personal_Nuclei_Templates) | High-signal Nuclei templates |
| [Firebase_Nuclei](https://github.com/CypherNova1337/Firebase_Nuclei) | Firebase service discovery template |
| [GF_Patterns](https://github.com/CypherNova1337/GF_Patterns) | Modern gf patterns (`lfi`, `sqli`, `xss`, `rce`, `ssti`, `idor`, `firebase`, `js-sinks`, ...) |
| [Code-Specter](https://github.com/CypherNova1337/Code-Specter) | In-browser DOM XSS sink and developer-note finder |

## 4. Third-Party Tools

```bash
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
CGO_ENABLED=1 go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
go install github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/tomnomnom/waybackurls@latest
go install github.com/tomnomnom/anew@latest
go install github.com/tomnomnom/gf@latest
go install github.com/tomnomnom/qsreplace@latest
go install github.com/lc/subjs@latest
go install github.com/hahwul/dalfox/v2@latest
go install github.com/ffuf/ffuf/v2@latest

pipx install uro
pipx install dirsearch

nuclei -update-templates
```

| Tool | What it does here |
|---|---|
| httpx | Probes the apex domains and fingerprints their technology |
| katana | Crawls the site (kept on the exact host with `-fs fqdn`) |
| gau / waybackurls | Historical URLs from public archives |
| subjs | Extracts URLs from JavaScript files |
| uro | Removes near-duplicate URLs |
| anew / qsreplace | Deduplicated appends / query-string rewriting |
| gf | Runs the GF_Patterns against URLs and files |
| dirsearch | Directory and file brute-forcing |
| nuclei | Template-based vulnerability scanning |
| interactsh-client | Out-of-band callbacks, needed by NovaLure |
| dalfox | XSS scanning |
| naabu + nmap | Port and service scanning |
| ffuf | API endpoint fuzzing |
| sslscan | TLS configuration |

## 5. Wordlists

```bash
git clone https://github.com/danielmiessler/SecLists.git ~/tools/wordlists/SecLists
git clone https://github.com/z5jt/API-documentation-Wordlist.git ~/tools/wordlists/API-documentation-Wordlist
ln -s ~/tools/wordlists/API-documentation-Wordlist/API-Documentation-Wordlist/api-documentation-endpoint.txt ~/tools/wordlists/api_endpoints.txt
```

**OneListForAll:** The old `onelistforallshort.txt` is gone. The repo now builds `onelistforall.txt` (the curated micro list plus every category's short list). Download it from the [releases page](https://github.com/six2dez/OneListForAll/releases), where it ships as a 7z archive, and extract it with `7z x` so it ends up at `~/tools/wordlists/onelistforall.txt`.

## 6. Keeping Up to Date

```bash
nuclei -update-templates
voidrecon update
for d in ~/tools/*/.git; do git -C "${d%/.git}" pull --ff-only; done
cp ~/tools/GF_Patterns/*.json ~/.gf/
```

## Usage

Once the installation is complete, follow the step-by-step guide in the main `README.md` to start your apex domain reconnaissance.

Happy hunting!
