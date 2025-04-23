# DomainDive - Installation Guide

This guide provides step-by-step instructions for installing the necessary tools for the DomainDive Apex Domain Reconnaissance Arsenal. Follow these instructions to set up your environment.

## Core Tools Installation

These are the primary tools used in the DomainDive workflow.

1.  **httpx-toolkit:** A fast and multi-purpose HTTP toolkit that allows to run multiple probes using retryablehttp library.

    ```bash
    go install github.com/projectdiscovery/httpx/cmd/httpx@latest
    ```

2.  **katana:** A next-generation crawling and spidering framework.

    ```bash
    go install github.com/projectdiscovery/katana/cmd/katana@latest
    ```

3.  **gau (Get All URLs):** Fetches known URLs from various sources.

    ```bash
    go install github.com/lc/gau/cmd/gau@latest
    ```

4.  **waybackurls:** Fetch all URLs that the Wayback Machine knows about for a given domain.

    ```bash
    go install github.com/tomnomnom/waybackurls@latest
    ```

5.  **subjs:** A tool to get javascript files URLs on a given domain.

    ```bash
    go install github.com/lc/subjs@latest
    ```

6.  **uro:** A simple tool to extract unique URLs from a list.

    ```bash
    pipx install uro
    ```

7.  **qsreplace:** A tool to replace values of query string parameters in URLs.

    ```bash
    go install github.com/tomnomnom/qsreplace@latest
    ```

8.  **dirsearch:** An advanced command-line tool for brute-forcing web directories and files.

    ```bash
    git clone https://github.com/maurosoria/dirsearch.git
    ```

9.  **Nuclei:** Fast and customisable vulnerability scanner based on YAML based DSL.

    ```bash
    go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
    nuclei --update
    ```

10. **SecretFinder:** A python script for finding sensitive data in JavaScript files.

    ```bash
    git clone https://github.com/m4ll0k/SecretFinder.git
    ```

11. **Corsy:** A tool to check for CORS misconfigurations.

    ```bash
    pip3 install corsy
    ```

12. **OpenRedirex:** A simple tool to detect open redirect vulnerabilities.

    ```bash
    go install github.com/rjmandell/OpenRedirex@latest
    ```

13. **dalfox:** A powerful XSS scanner and parameter analyzer.

    ```bash
    go install github.com/hahwul/dalfox@latest
    ```

14. **naabu:** A fast port scanner.

    ```bash
    go install github.com/projectdiscovery/naabu/v2/cmd/naabu@latest
    ```

15. **ffuf (Fuzz Faster U Fool):** A fast web fuzzer written in Go.

    ```bash
    go install github.com/ffuf/ffuf@latest
    ```

16. **whatweb:** Next generation web scanner.

    ```bash
    sudo apt-get update && sudo apt-get install whatweb # Debian/Ubuntu
    ```
    OR
    ```bash
    sudo yum install whatweb # CentOS/Fedora
    ```
    OR
    ```bash
    brew install whatweb # macOS
    ```

18. **sslscan:** Tests SSL/TLS services to find out the ciphers supported.

    ```bash
    sudo apt-get update && sudo apt-get install sslscan # Debian/Ubuntu
    ```
    OR
    ```bash
    sudo yum install sslscan # CentOS/Fedora
    ```
    OR
    ```bash
    brew install sslscan # macOS
    ```

20. **paramspider:** Mining parameters from dark corners of web archives.

    ```bash
    git clone https://github.com/devanshbatham/paramspider
    cd paramspider
    pip install .
    ```

## Wordlists

Ensure you have the necessary wordlists. You can download them using the provided links in the main `README.md` or use your own curated lists. Consider cloning the following repositories for comprehensive wordlists:

* **SecLists:** A collection of multiple types of lists used during security assessments.
    ```bash
    git clone https://github.com/danielmiessler/SecLists.git
    ```
* **OneListForAll:** A compilation of wordlists from various sources.
    ```bash
    git clone https://github.com/six2dez/OneListForAll.git
    ```

## Configuration

* **Ensure correct paths:** Update the paths in the main `README.md` to reflect the actual locations of your wordlists and tools.
* **Nuclei Templates:** Keep your Nuclei templates updated:
    ```bash
    nuclei -update-templates
    ```

## Usage

Once the installation is complete, you can follow the step-by-step guide in the main `README.md` to start your apex domain reconnaissance.

Happy hunting!
