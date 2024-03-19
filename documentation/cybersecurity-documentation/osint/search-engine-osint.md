# ☎️ Search Engine OSINT

Other documentation about google searches:

[google-search.md](information-gathering/google-search.md "mention")

Google - [https://www.google.com/](https://www.google.com/)

**Google Advanced Search -** [**https://www.google.com/advanced\_search**](https://www.google.com/advanced\_search)

Google Search Guide - [http://www.googleguide.com/print/adv\_op\_ref.pdf](http://www.googleguide.com/print/adv\_op\_ref.pdf)

Bing - [https://www.bing.com/](https://www.bing.com/)

Bing Search Guide - [https://www.bruceclay.com/blog/bing-google-advanced-search-operators/](https://www.bruceclay.com/blog/bing-google-advanced-search-operators/)

Yandex - [https://yandex.com/](https://yandex.com/)

DuckDuckGo - [https://duckduckgo.com/](https://duckduckgo.com/)

DuckDuckGo Search Guide - [https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/)

Baidu - [http://www.baidu.com/](http://www.baidu.com/)

#### 1. Basic Site Search:

* **Query:** `site:example.com`
* **Explanation:** This narrows down your search results to a specific website. For instance, `site:reddit.com` will show results only from Reddit.

#### 2. Combining Keywords:

* **Query:** `example AND example2 site:reddit`
* **Explanation:** Use "AND" to combine keywords. This query looks for pages on Reddit that mention both "example" and "example2".

#### 3. Exact Phrase Search:

* **Query:** `"exact phrase" site:reddit`
* **Explanation:** Enclose a phrase in quotes to find pages where the words appear exactly as you typed them. Useful for finding specific discussions.

#### 4. Combining Exact Phrase and Keywords:

* **Query:** `"exact phrase" theme * site:reddit.com`
* **Explanation:** Looks for pages on Reddit that contain an exact phrase and the word "theme".

#### 5. Excluding Terms:

* **Query:** `site:tesla -www -forums`
* **Explanation:** Exclude certain terms using "-" to refine your search. This example excludes pages from Tesla's main website and forums.

#### 6. Searching Filetypes:

* **Query:** `site:tesla password filetype:pdf`
* **Explanation:** Find PDF files on Tesla's site containing the word "password".

#### 7. Incorrect Filetype Search:

* **Query:** `site:tesla filetype:dockxbn`
* **Explanation:** Search for a specific filetype on a website. In this case, looking for files with a "dockxbn" extension.

#### 8. Removing Subdomains:

* **Query:** `site:tesla.com -www`
* **Explanation:** Exclude pages from the "www" subdomain of Tesla's website.

#### 9. Complex Exclusions:

* **Query:** `site:tesla.com -www -forums`
* **Explanation:** Exclude multiple terms to further filter results from Tesla's website.

#### 10. Intext, Inurl, Intitle Searches:

* **Queries:**
  * `name intext:password`
  * `name intext:password site:twitter.com`
  * `name inurl:password`
  * `name intitle:password`
* **Explanation:** These search methods help find pages where a specific word (like "password") appears in the text, URL, or title, often used for security research.
