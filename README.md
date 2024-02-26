## Scan Open Port with RUSTSCAN

```
cat list.txt | while IFS= read -r ip; do rustscan -a "$ip" --ulimit 99999 --accessible -g | sort -rn | while IFS= read -r port; do echo "$ip: $port"; done; done
```
## Scan Subdomain & DNS
- Subdomain C99: https://api.c99.nl 
- Censys: https://search.censys.io
- AMASS: https://github.com/owasp-amass/amass
- Subfinder: https://github.com/projectdiscovery/subfinder
- Yusub: https://github.com/justakazh/yusub
- DNSX: https://github.com/projectdiscovery/dnsx

```
#!/bin/bash
domain="$1"
output_file="$domain.txt"


curl -s "https://api.c99.nl/subdomainfinder?key={key}&domain=$domain&format=json" | sed 's/<[^>]*>/\n/g' | sort -u | anew "$output_file"
curl -X GET "https://search.censys.io/api/v2/hosts/search?virtual_hosts=EXCLUDE&q=$domain" --silent -H "Accept: application/json" --user "{CENSYS_API_ID}:{CENSYS_API_SECRET}" | jq -r '.result.hits[] | "\(.ip):\(.services[].port)"' | anew "$output_file"
sleep 5
amass enum -silent -brute -active -d "$domain" | anew "$output_file"
sleep 5
subfinder -all -silent -d "$domain" | anew "$output_file"
sleep 5
yusub "$domain" | anew "$output_file"
sleep 5
dnsx -silent -a -resp-only -l "$output_file" | anew "$output_file"
```
