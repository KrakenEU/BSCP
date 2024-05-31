script to spot differences between apps and find vulns
```bash
#!/bin/bash

domainONE="ABC.web-security-academy.net"
sessONE="ABC"
domainTWO="ABC.web-security-academy.net"
sessTWO="ABC"

wget --no-cookies --header "Cookie: session=$sessONE" --recursive --page-requisites --adjust-extension --span-hosts --convert-links --restrict-file-names=windows --no-parent --no-check-certificate --domain "$domainONE" "https://$domainONE"

wget --no-cookies --header "Cookie: session=$sessTWO" --recursive --page-requisites --adjust-extension --span-hosts --convert-links --restrict-file-names=windows --no-parent --no-check-certificate --domain "$domainONE" "https://$domainTWO"

diff -qr "$domainONE" "$domainTWO" | sort
```



dump .mds to obsidian
```python
#!/usr/bin/env python3
import sys
import requests
import bs4
import html2text
from urllib.parse import urlparse
def writemd(url):
 parts = urlparse(url)
 directories = parts.path.strip('/').split('/')
 filename=directories[-1]
 response = requests.get(url).content
 soup = bs4.BeautifulSoup(response, "lxml")
 div = soup.find("div", {"class": "component-solution is-expandable"})
 h = html2text.HTML2Text()
 md=h.handle(str(div))
 f=open(filename+".md","w")
 f.write(md)
 f.close()
response = requests.get("https://portswigger.net/web-security/all-labs").content
soup = bs4.BeautifulSoup(response, "lxml")
divs = soup.find_all("div", {"class": "widgetcontainer-lab-link"})
for div in divs:
 linker = div.find("a", href=True)
 link = linker['href']
 print(link)
 writemd("https://portswigger.net"+link)
 ```