## Scan Open Port with RUSTSCAN

```
cat list.txt | while IFS= read -r ip; do rustscan -a "$ip" --ulimit 99999 --accessible -g | grep -oP "\d+" | sort -urn | xargs -I {} echo "$ip:{}"; done
```
## Scan Subdomain & DNS
  - Anew: https://github.com/tomnomnom/anew
  - RustScan: https://github.com/RustScan/RustScan
  - Subdomain C99: https://api.c99.nl 
  - Censys: https://search.censys.io
  - AMASS: https://github.com/owasp-amass/amass
  - Subfinder: https://github.com/projectdiscovery/subfinder
  - Yusub: https://github.com/justakazh/yusub
  - CRT.sh: https://crt.sh
  - DNSX: https://github.com/projectdiscovery/dnsx

```
#!/bin/bash

#Domain
domain="$1"
#Output File
output_file="$domain.txt"

#Subdomain Scanner C99
#curl -s "https://api.c99.nl/subdomainfinder?key={key}&domain=$domain&format=json" | sed 's/<[^>]*>/\n/g' | sort -u | anew "$output_file"
curl -X GET "https://search.censys.io/api/v2/hosts/search?virtual_hosts=EXCLUDE&q=$domain+and+dns.names%3D%22*.$domain%22" --silent -H "Accept: application/json" --user "{api_id}:{api_secret}" | jq -r '.result.hits[] | "\(.ip):\(.services[].port)"' | anew "$output_file"
sleep 5
# Subdomain Scanner AMASS
amass enum -silent -brute -active -d "$domain" | anew "$output_file"
sleep 5
#SubdomainScanner Subfinder
subfinder -all -silent -d "$domain" | anew "$output_file"
sleep 5
#Subdomain Scanner Yusub
yusub "$domain" | anew "$output_file"
sleep 5
#crt.sh
curl -skL "https://crt.sh/?q=$domain&output=json" | jq -r ".[].common_name,.[].name_value" | tr -d '"' | sed
-E 's/\\n/\n/g;s/\*.//g;/[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,4}/d' | sort -u | anew "$output_file"
#DNSX
dnsx -silent -a -resp-only -l "$output_file" | anew "$output_file"
sleep 10
grep -v ":" "$output_file" | while IFS= read -r ip_address; do rustscan -a "$ip_address" --ulimit 99999 --accessible -g | grep -oP "\d+" | sort -urn | xargs -I {} echo "$ip_address:{}"; done | anew "$output_file"
```
