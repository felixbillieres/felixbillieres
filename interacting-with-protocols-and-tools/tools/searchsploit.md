# 📩 Searchsploit

#### **SearchSploit: A Quick Guide for Exploit Searching**

SearchSploit is a powerful tool used for searching, indexing, and retrieving exploits from multiple databases. It simplifies the process of finding relevant exploits for known vulnerabilities. Below is a quick guide on using SearchSploit:

<figure><img src="https://www.exploit-db.com/images/searchsploit-example.png" alt=""><figcaption></figcaption></figure>

**\*\*1. Installation:**

SearchSploit is typically pre-installed in Kali Linux. If you don't have it, you can install it using the following command:

```bash
sudo apt-get install exploitdb
```

**2. Basic Usage:**

* **Search by Keyword:**
  *   Use SearchSploit to find exploits related to a specific software or vulnerability by providing a keyword. For example:

      ```bash
      searchsploit <keyword>
      ```

      Replace `<keyword>` with the software or vulnerability you're interested in.
* **Search by Exact Match:**
  *   If you want an exact match for a term, enclose it in double quotes. For example:

      ```bash
      searchsploit "Apache Struts"
      ```
* **Search by Platform:**
  *   Specify the target platform using the `-w` option. For example, to search for Windows exploits:

      ```bash
      searchsploit -w windows
      ```

**3. Advanced Searching:**

* **Search by CVE:**
  *   Look for exploits associated with a specific CVE (Common Vulnerabilities and Exposures) identifier. For example:

      ```bash
      searchsploit -c <CVE>
      ```

      Replace `<CVE>` with the CVE identifier.
* **Search by EDB-ID:**
  *   Search for a specific exploit by its Exploit Database (EDB) identifier. For example:

      ```bash
      searchsploit -x <EDB-ID>
      ```

      Replace `<EDB-ID>` with the exploit's EDB identifier.

**4. Viewing Exploits:**

* **View Details:**
  *   View details about a particular exploit, including its path, platform, author, and description. For example:

      ```bash
      searchsploit -p 12345
      ```

      Replace `12345` with the EDB-ID of the exploit.
* **Copy Exploit:**
  *   Copy an exploit to the current directory for further analysis or use. For example:

      ```bash
      searchsploit -m 12345
      ```

      Replace `12345` with the EDB-ID of the exploit.

**5. Updating SearchSploit:**

* **Update Database:**
  *   Keep your SearchSploit database up-to-date by regularly updating it. Use the following command:

      ```bash
      searchsploit -u
      ```

**6. Notes:**

* SearchSploit may have false positives or outdated entries. Always verify the suitability of an exploit for your target system.
* Use SearchSploit responsibly and ethically. Ensure you have proper authorization before attempting to use any exploits.
