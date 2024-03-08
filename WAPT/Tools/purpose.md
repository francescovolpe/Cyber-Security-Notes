# Tools

**Files/Directory discovery**
- **hakrawler** *Web crawler, quick discovery of endpoints and assets within a web application*
  - You can set depth
  - `echo https://example.com | hakrawler`
- **feroxbuster** *A fast, simple, recursive content discovery*
  - You can set depth (recursion), extract links from response body
  - `feroxbuster -u https://example.com -x html,php,js,txt,pdf,json`
- **waybackurls** *Fetch all the URLs that the Wayback Machine knows about for a domain*
  - Uncover historical data about a website
  - `waybackurls https://example.com`
