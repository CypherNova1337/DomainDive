# DomainDive - Apex Domain Reconnaissance Arsenal

[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/CypherNova1337/DomainDive/graphs/commit-activity)
[![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-brightgreen.svg?style=flat)](https://GitHub.com/CypherNova1337/DomainDive/pulls)

This guide provides a structured approach to reconnaissance focused exclusively on apex domains, ensuring subdomains are strictly out of scope. It blends automated tools with manual analysis to achieve a deep understanding of the target domain's attack surface.

## Key Principles

* **Iterative:** Continuously feed newly discovered information back into the reconnaissance process.
* **Layered:** Employ a diverse range of tools and techniques for comprehensive coverage.
* **Manual Analysis:** Don't rely solely on automation; critical thinking and manual verification are essential.

## Preparation (One-Time Setup)

1.  **Install Required Tools:** Ensure all necessary tools are installed on your system. Refer to their respective documentation for installation instructions.
2.  **Obtain Wordlists:**
    * `vhost_wordlist.txt`: For Virtual Host discovery (e.g., `www`, `mail`). A good starting point is [https://github.com/maverickNerd/wordlists/blob/master/vhost.txt](https://github.com/maverickNerd/wordlists/blob/master/vhost.txt).
    * `parameter_fuzzing_wordlist.txt`: For fuzzing web application parameters. Consider using [SecLists](https://github.com/danielmiessler/SecLists.git).
    * `onelistforallshort.txt`: A comprehensive wordlist for directory brute-forcing. You can obtain this from [OneListForAll](https://github.com/six2dez/OneListForAll). **Ensure the path is correct.**
    * `xss_wordlist.txt`: For crafting Cross-Site Scripting (XSS) payloads. [SecLists](https://github.com/danielmiessler/SecLists.git) contains excellent XSS wordlists.
    * `resolvers.txt`: A list of reliable DNS resolvers. A good source is [https://github.com/trickest/resolvers](https://github.com/trickest/resolvers).
    * `api_endpoints.txt`: A wordlist containing potential API endpoints. [z5jt](https://github.com/z5jt/API-documentation-Wordlist) 
3.  **Configure Nuclei Templates:** Ensure your Nuclei templates directory (`/home/USER/Documents/nuclei-templates/`) is correctly configured.
4.  **`sort.py` (Optional):** A Python script (`sort.py`) for sorting and optionally limiting parameter lists.

## Manual Reconnaissance

1.  **Shodan Exploration:**
    * Visit the Shodan web interface ([https://www.shodan.io/](https://www.shodan.io/)).
    * Search for `ssl:'target.com' 200`.
    * Explore the "More" section under top organizations.
    * Examine `http.titles` and `http.components` for valuable insights.

## Reconnaissance Steps

1.  **Target Definition:**
    ```bash
    mkdir "TARGET"
    cd "TARGET"
    nano domains.txt
    ```
    Enter the target apex domains (one per line, e.g., `target.com`). Save and exit.

2.  **Phase 1: Domain Probing and Filtering**
    ```bash
    httpx -l domains.txt -ports 80,443,8080,8000,8888 -threads 200 -o domains_alive.txt -title -tech-detect
    cat domains_alive.txt | grep -oE "^[^ ]+" > live_domains_only.txt
    ```

3.  **Phase 2: URL Discovery and Content Extraction**
    ```bash
    katana -u live_domains_only.txt -d 10 -c 50 -kf -jc -fx -ef woff,css,png,svg,jpg,woff2,jpeg,gif,svg >> allurls.txt
    ```
    ```bash
    cat live_domains_only.txt | gau | anew allurls.txt
    ```
    ```bash
    cat live_domains_only.txt | waybackurls | anew allurls.txt
    ```
    ```bash
    cat allurls.txt | grep -E "\.txt|\.log|\.cache|\.secret|\.db|\.backup|\.yml|\.json|\.gz|\.rar|\.zip|\.config" >> sens1.txt
    ```
    ```bash
    cat allurls.txt | grep -E "\.js.?" | sort -u >> alljs.txt
    ```
    ```bash
    cat alljs.txt | subjs >> js_extracted_urls.txt
    ```
    ```bash
    while read -r domain; do
        awk -v domain="$domain" 'index($0, domain) == 1' js_extracted_urls.txt >> filtered_js_urls.txt
    done < domains.txt
    ```
    ```bash
    cat allurls.txt filtered_js_urls.txt | anew allurls.txt
    ```
    ```bash
    cat allurls.txt | qsreplace | awk '/\?.*=/ {print}' | grep -vE '\.(png|jpg|gif|svg|js|css|xml)$' > filterparams.txt
    ```
    ```bash
    paramspider -l live_domains_only.txt >> paramspider_params.txt
    ```

4.  **Phase 3: Directory and File Brute-Forcing**
    ```bash
    dirsearch -l live_domains_only.txt -x 500,502,429,404,400 -R 5 --random-agent -t 100 -F -o directory.txt -w /home/USER/Documents/oneListForall/onelistforallshort.txt
    ```
    ```bash
    cat allurls.txt | while read url; do
        dirsearch -u "$url" -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,aspx~,asp~,py,py~,rb,eb~,php,php~,bak,bkp,cache,cgi,conf,csv,html,inc,jar,js,json,jsp,jsp~,lock,log,rar,old,sql,sql.gz,sql.tar.gz,sql~.swp.swp~,tar,tar.bz2,tar.gz,txt,wadl,zip,.log,.xml,.js,.json -x 500,502,429,404,400 -R 2 --random-agent -t 20 -F -o "dirsearch_extensions_$(echo "$url" | sed 's/[^a-zA-Z0-9]/_/g').txt"
    done
    ```
      **Note:** This will most likely take a while to complete, but its worth the wait. Usually I have this going on in the background.
    
6.  **Phase 4: Vulnerability Scanning with Nuclei**
    ```bash
    cat alljs.txt | nuclei -t /home/USER/Documents/nuclei-templates/http/exposures/ -c 30 -o nuclei_js_exposures.txt
    ```
    ```bash
    nuclei -list live_domains_only.txt -tags cves,osint,tech -o nuclei_general.txt
    ```
    ```bash
    nuclei -list live_domains_only.txt -t /home/cyphernove1337/Documents/nuclei-templates/cors.yaml -o nuclei_cors.txt
    ```
    ```bash
    nuclei -list live_domains_only.txt -t /home/cyphernove1337/Documents/nuclei-templates/cRlf.yaml -o nuclei_crlf.txt
    ```
    ```bash
    cat allurls.txt | gf lfi | nuclei -tags lfi -o nuclei_lfi.txt
    ```

7.  **Phase 5: Specialized Checks and Tools**
    ```bash
    cat alljs.txt | while read url; do python3 /secretfinder/SecretFinder.py -i "$url" -o cli; done >> secret.txt
    ```
    ```bash
    python3 /path/to/Corsy/corsy.py -i live_domains_only.txt -t 10 --headers "User-Agent: GoogleBot\nCookies: SESSION=VoidSec" >> corsmisconf.txt
    ```
    ```bash
    cat allurls.txt | gf redirect | openredirex >> open_redirects.txt
    ```
    ```bash
    cat allurls.txt | dalfox -b your_xss_endpoint_here -o dalfox_xss.txt
    ```
    ```bash
    naabu -list live_domains_only.txt -c 50 -nmap-cli 'nmap -sV -sC -A -T4 -p-' -o naabu-full-deep.txt
    ```
    ```bash
    grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' naabu-full-deep.txt | sort -u > ips.txt
    ```
    ```bash
    while read -r ip; do
        ffuf -w live_domains_only.txt:FUZZ -u https://$ip -H "Host: FUZZ.$TARGET" -fs 0 -o vhost_results_"$ip".txt
    done < ips.txt
    ```
    ```bash
    cat filterparams.txt | gf xss | ffuf -w /path/to/your/xss_wordlist.txt:FUZZ -u FUZZ -fs 0 -o ffuf_xss_results.txt
    ```
    ```bash
    cat live_domains_only.txt | whatweb -a 3 -v -o whatweb_results.txt
    ```
    ```bash
    while read -r domain; do
        sslscan "$domain" >> sslscan_results.txt
    done < live_domains_only.txt
    ```
    ```bash
    ffuf -w /path/to/api_endpoints.txt:FUZZ -u [https://target.com/api/FUZZ](https://target.com/api/FUZZ) -mc 200,301,302,401,403 -o api_fuzz_results.txt
    ```
    ```bash
    python3 sort.py
    ```
    **Note:** the sort.py is only useful if over 100,000 results. So in many cases it is uneeded. 

## Contributions

Contributions to enhance this arsenal are highly welcome! Feel free to submit pull requests with improvements, new tools, or updated techniques.

## Disclaimer

This information is intended for educational and ethical security testing purposes only. Always ensure you have explicit permission before conducting reconnaissance on any target. Misuse of this information can lead to severe legal consequences.
