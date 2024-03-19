# ⛰️ Hunting Subdomains

## Hunting Subdomains 🎯🌐

Discovering subdomains is a crucial aspect of reconnaissance, whether you're a security professional or just curious about the online footprint of a domain. Here are some tools and techniques to help you hunt for subdomains effectively:

### 1. **Sublist3r:**

* **Description:** Sublist3r is a powerful subdomain enumeration tool that utilizes various search engines and APIs to discover subdomains associated with a target domain.
*   **Usage:**

    ```bash
    python sublist3r.py -d example.com
    ```

### 2. **Amass:**

* **Description:** Amass is an open-source tool that helps with subdomain discovery by actively querying multiple intelligence sources, performing DNS scraping, and more.
*   **Usage:**

    ```bash
    amass enum -d example.com
    ```

### 3. **Censys Subdomain Finder:**

* **Description:** Censys offers a subdomain finder that leverages its extensive internet-wide scans to provide comprehensive subdomain information.
* **Usage:** [Censys Subdomain Finder](https://censys.io/finder)

### 4. **Subfinder:**

* **Description:** Subfinder is a subdomain discovery tool designed to find valid subdomains using passive online sources.
*   **Usage:**

    ```bash
    subfinder -d example.com
    ```

### 5. **Google Dorking:**

* **Description:** Utilize Google's advanced search operators (dorks) to discover subdomains indexed by the search engine.
*   **Example:**

    ```
    site:example.com -www
    ```

### 6. **DNS Dumpster:**

* **Description:** DNS Dumpster is an online service that provides DNS information about a target domain, including discovered subdomains.
* **Usage:** [DNS Dumpster](https://dnsdumpster.com/)

### 7. **Certspotter:**

* **Description:** Certspotter is a tool that monitors Certificate Transparency logs, helping identify subdomains based on SSL/TLS certificates.
*   **Usage:**

    ```bash
    certspotter -d example.com
    ```

### 8. **OWASP Amass:**

* **Description:** OWASP Amass is a versatile tool for subdomain enumeration, DNS name resolution, and network mapping.
*   **Usage:**

    ```bash
    amass enum -d example.com
    ```

### 9. **Aquatone:**

* **Description:** Aquatone is a reconnaissance tool that can discover subdomains, take screenshots, and gather other information about a target.
*   **Usage:**

    ```bash
    aquatone-discover -d example.com
    ```
