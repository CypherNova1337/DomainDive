# DomainDive - Apex Domain Reconnaissance Arsenal

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/CypherNova1337/DomainDive/graphs/commit-activity)
[![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg?style=flat)](https://GitHub.com/CypherNova1337/DomainDive/pulls)

This guide is for programs where **only the apex domain is in scope** — `target.com` and nothing under it. No `www`, no `api.`, no `dev.`: zero subdomains. Every step below is built to stay on the exact hosts you list, and every result is passed through `scope_filter.py` before it feeds the next step, so a subdomain never slips into a scan.

If the program allows subdomains (`*.target.com`), use [WebRecon-Arsenal](https://github.com/CypherNova1337/WebRecon-Arsenal) instead.

## Key Principles

* **Exact scope:** A host is in scope only if it matches a line in `domains.txt` exactly. Out-of-scope paths and keywords are stripped as well.
* **Iterative:** Continuously feed newly discovered information back into the reconnaissance process.
* **Layered:** Employ a diverse range of tools and techniques for comprehensive coverage.
* **Manual Analysis:** Don't rely solely on automation; critical thinking and manual verification are essential.

## Preparation (One-Time Setup)

1.  **Install Required Tools:** Follow [install.md](install.md). It puts everything under `~/tools/`, which is the path used in the commands below.
2.  **Obtain Wordlists:**
    * `onelistforall.txt`: A comprehensive wordlist for directory brute-forcing, from [OneListForAll](https://github.com/six2dez/OneListForAll) (see install.md — the old `onelistforallshort.txt` was replaced).
    * `api_endpoints.txt`: A wordlist containing potential API endpoints. [z5jt](https://github.com/z5jt/API-documentation-Wordlist)
    * Parameter names are built into [paramvoid](https://github.com/CypherNova1337/paramvoid), so no separate parameter wordlist is needed.
3.  **gf Patterns:** Install [GF_Patterns](https://github.com/CypherNova1337/GF_Patterns) into `~/.gf` (covered in install.md). Every `gf` command below uses them.
4.  **Nuclei Templates:** Keep the community templates updated (`nuclei -update-templates`) and clone [Personal_Nuclei_Templates](https://github.com/CypherNova1337/Personal_Nuclei_Templates) to `~/tools/`.

## Manual Reconnaissance

1.  **Shodan Exploration:**
    * Visit the Shodan web interface ([https://www.shodan.io/](https://www.shodan.io/)).
    * Search for `ssl:'target.com' 200`.
    * Explore the "More" section under top organizations.
    * Examine `http.titles` and `http.components` for valuable insights.
    * Only the apex itself is in scope. Shodan will show you subdomains and other hosts on the same certificate — note them, don't touch them.
2.  **Code Specter:** Install the [Code-Specter](https://github.com/CypherNova1337/Code-Specter) bookmarklet and click it on the target's main pages, especially behind a login. It flags developer notes and user input that reaches `innerHTML`, `eval` or `document.write` (DOM XSS sinks), right in your browser.

## Fast Path: VoidRecon

[VoidRecon](https://github.com/CypherNova1337/VoidRecon) runs most of this pipeline as one engine and enforces scope itself. By default a bare apex in scope also covers its subdomains, so switch that off for apex-only programs:

```bash
export VOIDRECON_GENERAL_WILDCARD_APEX=false
voidrecon run -T domains.txt -p quick                     # passive first
voidrecon run -T domains.txt --active -p standard          # then active
```

With wildcard apex off, discovered subdomains are recorded as leads but never actively probed. VoidRecon scopes by host, so path and keyword exclusions from `out_of_scope.txt` still need the manual filter below. The manual steps below are still worth running on top: they go deeper on content and parameters, and you control every request.

## Reconnaissance Steps

1.  **Target Definition:**
    ```bash
    mkdir "TARGET"
    cd "TARGET"
    nano domains.txt
    ```
    Enter the target apex domains (one per line, e.g. `target.com`) without `http://`, `https://` or trailing slashes. If the program lists `www.target.com` as in scope, add it as its own line — otherwise it is treated as a subdomain and dropped. Save and exit.

    **Out-of-scope entries (optional but recommended):**
    ```bash
    nano out_of_scope.txt
    ```
    One entry per line. Anything with a dot is a host; anything else is a keyword or path, and any URL containing it is removed:
    ```text
    contact
    /logout
    /admin/billing
    ```
    If you have nothing to exclude, still create the file so the filter below works:
    ```bash
    touch out_of_scope.txt
    ```

    **Scope filter:** Load this function in the terminal you're working in. Every step pipes its results through it.
    ```bash
    inscope() { python3 ~/tools/DomainDive/scope_filter.py -s domains.txt -x out_of_scope.txt --apex-only "$@"; }
    ```
    It keeps only lines whose host matches `domains.txt` exactly and drops anything matching `out_of_scope.txt`. It reads URLs, bare hosts, or tool output like httpx's.

2.  **Phase 1: Domain Probing**
    ```bash
    httpx -l domains.txt -ports 80,443,8080,8000,8888 -threads 200 -title -tech-detect -status-code -o domains_alive.txt
    cat domains_alive.txt | awk '{print $1}' | inscope > live_domains_only.txt
    ```
    `-tech-detect` identifies the web server, framework and CMS, so no separate fingerprinting tool is needed.

3.  **Phase 2: URL Discovery and Content Extraction**
    ```bash
    katana -list live_domains_only.txt -d 10 -c 50 -fs fqdn -kf all -jc -fx -ef woff,css,png,svg,jpg,woff2,jpeg,gif | inscope | anew allurls.txt
    ```
    `-fs fqdn` keeps the crawler on the exact host instead of following links to subdomains.
    ```bash
    cat domains.txt | gau | inscope | anew allurls.txt
    ```
    ```bash
    cat domains.txt | waybackurls -no-subs | inscope | anew allurls.txt
    ```
    `gau` excludes subdomains by default; `waybackurls` needs `-no-subs`.
    ```bash
    cat allurls.txt | uro > filterparams.txt
    ```
    `uro` removes near-duplicate URLs (same path, different parameter values), which cuts the list down for everything that follows.
    ```bash
    cat allurls.txt | grep -iE "\.txt|\.log|\.cache|\.secret|\.db|\.backup|\.bak|\.yml|\.json|\.gz|\.rar|\.zip|\.config|\.env|\.sql" >> sens1.txt
    ```
    ```bash
    cat allurls.txt | grep -iE "\.js(\?|$)" | sort -u >> alljs.txt
    ```
    ```bash
    cat alljs.txt | subjs | inscope | anew allurls.txt
    ```
    `subjs` pulls URLs out of the JavaScript; `inscope` drops the ones pointing at subdomains or third parties.

4.  **Phase 3: Hidden Parameters**
    ```bash
    cat filterparams.txt | grep -viE "\.(js|css|png|jpg|jpeg|gif|svg|woff2?|ico)(\?|$)" | sed 's/?.*//' | sort -u > endpoints.txt
    paramvoid -i endpoints.txt -oJ paramvoid_results.json
    ```
    [paramvoid](https://github.com/CypherNova1337/paramvoid) finds parameters the application reads but never links to (`debug`, `admin`, `user_id`, ...). Add `-headers 'Cookie: session=...'` to test the logged-in surface, and try a second pass with `-casing likeThis` or `-casing like_this`.

5.  **Phase 4: Directory and File Brute-Forcing**
    ```bash
    dirsearch -l live_domains_only.txt -x 500,502,429,404,400 -R 5 --random-agent -t 100 -F -o directory.txt -w ~/tools/wordlists/onelistforall.txt
    ```
    ```bash
    cat live_domains_only.txt | while read -r url; do
        dirsearch -u "$url" -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,aspx~,asp~,py,py~,rb,rb~,php,php~,bkp,cache,cgi,csv,html,inc,jar,js,json,jsp,jsp~,lock,log,rar,sql.gz,sql.zip,sql.tar.gz,sql~,swp~,tar,tar.bz2,tar.gz,txt,wadl,zip,xml -x 500,502,429,404,400 -R 2 --random-agent -t 20 -F -o "dirsearch_extensions_$(echo "$url" | sed 's/[^a-zA-Z0-9]/_/g').txt"
    done
    ```
    **Note:** This will most likely take a while to complete, but it's worth the wait. Usually I have this going on in the background.

    dirsearch follows redirects (`-F`). If a redirect lands on a subdomain, ignore those results — they are out of scope.

6.  **Phase 5: JavaScript Analysis**
    ```bash
    mkdir -p js
    cat alljs.txt | inscope | while read -r url; do
        curl -sk "$url" -o "js/$(echo "$url" | md5sum | cut -c1-16).js"
    done
    ```
    ```bash
    python3 ~/tools/Personal_Scipts/recon/secretscan.py -r js -o secrets.jsonl
    ```
    [secretscan.py](https://github.com/CypherNova1337/Personal_Scipts) combines known credential formats with entropy scoring, so it catches unknown tokens as well as AWS, GitHub, Stripe and other keys.
    ```bash
    gf firebase js/ > firebase_hits.txt
    gf js-sinks js/ > js_sinks.txt
    ```
    If `firebase_hits.txt` shows a project ID (from `firebaseConfig` / `projectId`), check it with [Firebase_Nuclei](https://github.com/CypherNova1337/Firebase_Nuclei):
    ```bash
    nuclei -t ~/tools/Firebase_Nuclei/firebase-discovery.yaml -u <FIREBASE_PROJECT_ID>
    ```
    Source maps rebuild the original, readable source:
    ```bash
    cat alljs.txt | inscope | while read -r url; do
        sourcemapper -jsurl "$url" -output "sourcemaps/$(echo "$url" | md5sum | cut -c1-16)"
    done
    ```

7.  **Phase 6: Vulnerability Scanning with Nuclei**
    ```bash
    nuclei -list live_domains_only.txt -t ~/tools/Personal_Nuclei_Templates/exposures/ -t ~/tools/Personal_Nuclei_Templates/misconfiguration/ -t ~/tools/Personal_Nuclei_Templates/technologies/ -t ~/tools/Personal_Nuclei_Templates/cves/ -o nuclei_personal.txt
    ```
    [Personal_Nuclei_Templates](https://github.com/CypherNova1337/Personal_Nuclei_Templates) are tuned to fire only on corroborated evidence, so start with these. The `takeovers/` folder is left out on purpose — subdomain takeovers are out of scope here.
    ```bash
    cat alljs.txt | inscope | nuclei -t ~/nuclei-templates/http/exposures/ -c 30 -o nuclei_js_exposures.txt
    ```
    ```bash
    nuclei -list live_domains_only.txt -tags cve,osint,tech -o nuclei_general.txt
    ```
    ```bash
    cat filterparams.txt | nuclei -t ~/tools/Personal_Nuclei_Templates/vulnerabilities/ -t ~/tools/Personal_Nuclei_Templates/fuzzing/ -dast -o nuclei_dast.txt
    ```
    The DAST templates (SQLi, LFI, SSRF, SSTI, open redirect) fuzz the query parameters in the URLs you collected.

8.  **Phase 7: Specialized Checks and Tools**

    **CORS, CRLF and hop-by-hop headers ([webscan.py](https://github.com/CypherNova1337/Personal_Scipts)):**
    ```bash
    python3 ~/tools/Personal_Scipts/exploit/webscan.py all -i live_domains_only.txt -o webscan_findings.jsonl
    ```

    **Blind SSRF and open redirects ([NovaLure](https://github.com/CypherNova1337/NovaLure)):**
    ```bash
    python3 ~/tools/NovaLure/NovaLure.py -i filterparams.txt --skip-assetfinder --skip-httprobe -o novalure_report.md
    ```
    `--skip-assetfinder` matters here: without it NovaLure goes looking for subdomains.

    **LFI ([LFI-Machine](https://github.com/CypherNova1337/LFI-Machine)):**
    ```bash
    cat filterparams.txt | gf lfi | inscope > lfi_candidates.txt
    lfimachine -l lfi_candidates.txt -o lfi_loot/ --json lfi_results.json
    ```

    **SQL injection ([SmartSQL](https://github.com/CypherNova1337/SmartSQL)):**
    ```bash
    cat filterparams.txt | gf sqli | inscope > sqli_candidates.txt
    while read -r url; do smartsql -u "$url" --random-agent; done < sqli_candidates.txt | tee smartsql_results.txt
    ```
    SmartSQL tests every parameter in the URL and adapts when a WAF blocks it.

    **RCE, SSTI and command injection ([VoidStrike](https://github.com/CypherNova1337/VoidStrike)):**
    ```bash
    cat filterparams.txt | gf rce | inscope > rce_candidates.txt
    cat filterparams.txt | gf ssti | inscope | anew rce_candidates.txt
    cat rce_candidates.txt | xargs voidstrike scan -s scope.yaml -o voidstrike-results
    ```
    Copy `scope.example.yaml` from the VoidStrike repo to `scope.yaml` and list only the apex hosts under `in_scope` (no `*.` entries) and your exclusions under `out_of_scope`.

    **XSS (dalfox):** Replace `your_xss_endpoint_here` with your blind XSS collaborator URL.
    ```bash
    cat filterparams.txt | gf xss | inscope | dalfox pipe -b your_xss_endpoint_here -o dalfox_xss.txt
    ```

    **IDOR ([Auto-IDOR](https://github.com/CypherNova1337/Auto-IDOR)):** Needs two accounts on the target. Pick an endpoint that returns your own objects (`gf idor` on `filterparams.txt` helps find them):
    ```bash
    auto-idor quick -u 'https://target.com/api/invoices/{id}' --token-a "$USER_A_JWT" --token-b "$USER_B_JWT" --owns-a 1041,1042 --owns-b 2001,2002
    ```

    **Origin IP behind a CDN ([VoidOrigin](https://github.com/CypherNova1337/VoidOrigin)):**
    ```bash
    python3 ~/tools/VoidOrigin/voidorigin.py --targets domains.txt --no-brute -o voidorigin.json
    ```
    `--no-brute` skips subdomain brute forcing. VoidOrigin verifies each candidate IP by asking it for the apex itself, so a confirmed origin is the in-scope site with the CDN stepped over. Check the program rules on testing origin IPs directly before you do.

    **Ports and services:**
    ```bash
    sed -E 's#^https?://##; s#[:/].*##' live_domains_only.txt | sort -u > live_hosts.txt
    naabu -list live_hosts.txt -c 50 -nmap-cli 'nmap -sV -sC -Pn' -o naabu-full-deep.txt
    ```

    **TLS:**
    ```bash
    while read -r host; do
        sslscan "$host" >> sslscan_results.txt
    done < live_hosts.txt
    ```

    **API endpoints:**
    ```bash
    while read -r url; do
        ffuf -w ~/tools/wordlists/api_endpoints.txt:FUZZ -u "${url}FUZZ" -mc 200,301,302,401,403 -o "api_fuzz_$(echo "$url" | sed 's/[^a-zA-Z0-9]/_/g').json"
    done < live_domains_only.txt
    ```

    **`sort.py`:**
    ```bash
    cp ~/tools/DomainDive/sort.py .
    python3 sort.py
    ```
    **Note:** sort.py is only useful if `filterparams.txt` has over 100,000 results, so in many cases it is unneeded.

### What's not here, and why

Virtual host fuzzing, subdomain permutation, subdomain takeover checks and VPN gateway discovery all work by finding or touching other hostnames. On an apex-only program those are out of scope, so they live in [WebRecon-Arsenal](https://github.com/CypherNova1337/WebRecon-Arsenal) instead.

## Contributions

Contributions to enhance this arsenal are highly welcome! Feel free to submit pull requests with improvements, new tools, or updated techniques.

## Disclaimer

This information is intended for educational and ethical security testing purposes only. Always ensure you have explicit permission before conducting reconnaissance on any target. Misuse of this information can lead to severe legal consequences.
