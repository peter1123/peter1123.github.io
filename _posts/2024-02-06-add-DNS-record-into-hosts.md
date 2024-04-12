通过 https://api.api-ninjas.com/v1/dnslookup 查询DNS解析，并写入/etc/hosts

因为我这个是放在openwrt里面，所以第一行是/bin/ash

API KEY可以自己申请

DOMAIN填写自己需要的

```
#!/bin/ash

# Set the domain and API key
DOMAIN="api.themoviedb.org"
API_KEY="Your API Key"

# Perform DNS lookup using API
API_RESPONSE=$(wget --header="X-Api-Key:$API_KEY" "https://api.api-ninjas.com/v1/dnslookup?domain=$DOMAIN" -O-)

# Parse JSON response to extract DNS records
A_RECORDS=$(echo "$API_RESPONSE" | jq -r '.[] | select(.record_type == "A") | .value')
AAAA_RECORDS=$(echo "$API_RESPONSE" | jq -r '.[] | select(.record_type == "AAAA") | .value')

sed -i "/$DOMAIN/d" /etc/hosts

# Create or append to /etc/hosts file with a timestamp
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
# Create or append to /etc/hosts file
echo "# DNS records for $DOMAIN (Updated: $TIMESTAMP)." | tee -a /etc/hosts
for IP in $A_RECORDS; do
    echo "$IP $DOMAIN" | tee -a /etc/hosts
done

for IPV6 in $AAAA_RECORDS; do
    echo "$IPV6 $DOMAIN" | tee -a /etc/hosts
done

echo "DNS records added to /etc/hosts (Updated: $TIMESTAMP)."
```
