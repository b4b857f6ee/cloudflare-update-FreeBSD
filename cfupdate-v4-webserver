#!/usr/local/bin/bash

# Forked from b4b857f6ee/cloudflare-update-livebox
# CHANGE THESE
auth_email="myemail@domain.com"            # The email used to login 'https://dash.cloudflare.com'
auth_key="00000000aaaaaaaabbbbbbccccccc"   # Top right corner, "My profile" > "Global API Key"
zone_identifier="00000000aaaaaaaabbbbbbccccccc" # Can be found in the "Overview" tab of your domain
record_name="mydomain.com"                     # Which record you want to be synced
interface="mynetworkinterface"   # You can find it with ifconfig

# DO NOT CHANGE LINES BELOW (Check using FreeBSD)
ip=$(ifconfig $interface | grep 'inet ' | cut -d' ' -f2)

# SCRIPT START
echo "[Cloudflare DDNS] Check Initiated"

# Seek for the record
record=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$zone_identifier/dns_records?type=A&name=$record_name" -H "X-Auth-Email: $auth
_email" -H "X-Auth-Key: $auth_key" -H "Content-Type: application/json")
# Can't do anything without the record
if [[ $record == *"\"count\":0"* ]]; then
  >&2 echo -e "[Cloudflare DDNS] Record does not exist, perhaps create one first?"
  exit 1
fi

# Set existing IP address from the fetched record
old_ip=$(echo "$record" | grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b")

# Compare if they're the same
if [[ $ip == $old_ip ]]; then
  echo "[Cloudflare DDNS] IP has not changed."
  exit 0
fi

# Set the record identifier from result
record_identifier=$(echo "$record" | grep -oE "id.*\"([a-zA-Z0-9_.-]+).*\",\"zone_id" | cut -d'"' -f3)
# The execution of update
update=$(curl -s -X PATCH https://api.cloudflare.com/client/v4/zones/$zone_identifier/dns_records/$record_identifier -H "X-Auth-Email: $auth_email
" -H "X-Auth-Key: $auth_key" -H "Content-Type: application/json" --data "{\"type\":\"A\",\"name\":\"$record_name\",\"content\":\"$ip\",\"ttl\":120
,\"proxied\":true}")
# The moment of truth
case "$update" in
*"\"success\":false"*)
  >&2 echo -e "[Cloudflare DDNS] Update failed for $record_identifier. DUMPING RESULTS:\n$update"
  exit 1;;
*)
  echo "[Cloudflare DDNS] IPv4 context '$ip' has been synced to Cloudflare.";;
esac
