# Recon

#### Subdomain

```
subfinder -d evil.com -o subs
```

```
py ~/tools/vt-subfinder.py -d evil.com | anew subs
```

```
curl -s https://otx.alienvault.com/api/v1/indicators/hostname/evil.com/passive_dns | jq -r '.passive_dns[]?.hostname' | grep -E "^[a-zA-Z0-9.-]+\.evil.com$" | anew subs
```

```
curl -s "https://urlscan.io/api/v1/search/?q=domain:evil.com&size=10000" | jq -r '.results[]?.page?.domain' | grep -E "^[a-zA-Z0-9.-]+\.evil.com$" | anew subs
```

```
curl -s "https://crt.sh/?q=%.evil.com&output=json" | jq -r '.[].name_value' | grep -E "^[a-zA-Z0-9.-]+\.evil.com$" | anew subs
```

#### Filter host

```
httpx-toolkit -l subs -td -server -sc -ip -o subs.httpx
```

#### Hidden file dan folder

```
dirsearch -u https://evil.com -e php,cgi,htm,html,shtm,shtml,js,txt,bak,zip,old,conf,log,pl,asp,aspx,jsp,sql,db,sqlite,mdb,tar,gz,7z,rar,json,xml,yml,yaml,ini,java,py,rb,php3,php4,php5 --full-url -i 200,302,403
```

```
dirb https://evil.com
```

#### Crawling JS

```
katana -u evil.com -jc | grep ".js$"
```

```
cut -d' ' -f1 subs.httpx | katana -jc | grep ".js$" | urldedupe >> urls.js
```

#### Cari string di file JS

```
py ~/tools/JSS-Finder/js-string-finder.py -l urls.js -s "api"
```

*Daftar kata sensitif:* <https://gist.github.com/nmwafa/352e991f3a78bf9b1baf63d15c7f0101>

*Parser:* `echo "some|word" | tr '|' ' '`

#### Cari API/String sensitif lain dengan GitHub dork

```
/[a-zA-Z0-9.-]+\.evil.com\/api.*/
```

#### Scan potensial CORS

```
cat urls | xargs -P 15 -n 1 sh -c 'curl -m 5 -s -I -H "Origin: https://evil.com" "$0" | grep -qi "evil.com" && echo "$0"'
```

#### Secrets Discovery

```
curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin

trufflehog git https://github.com/trufflesecurity/test_keys --results=verified
trufflehog github --org=trufflesecurity --results=verified
trufflehog s3 --bucket=<bucket name> --results=verified,unknown
```
