# powwer-dns-pdns-clean-up-script
PowerDNS (PDNS) Clean-up script (For slave servers)

<pre><code>
#!/bin/bash
# Dependencies:
# bind-utils
# mysql-client
#### Config ################################
 
DBUSER="pdns"
DBPASS="****"
 
#### End of Config #########################
 
MYSQL="mysql -u $DBUSER -p$DBPASS --skip-column-name --silent -e"
 
check() {
AUTH=`dig @$1 $2 | grep "status" | awk -F , '{print $2}' | awk -F ': ' '{print $2}'`
if [ $AUTH = "REFUSED" ] || [ $AUTH = "NXDOMAIN" ]; then
echo "$1 $2: Server not AUTH or SERVfail - removing zone..."
DOMAIN_ID=`$MYSQL "USE pdns; SELECT id FROM domains WHERE name='$2' AND type='SLAVE' AND master='$1' LIMIT 1;"`
$MYSQL "USE pdns; DELETE FROM records WHERE domain_id='$DOMAIN_ID';"
$MYSQL "USE pdns; DELETE FROM domains WHERE id='$DOMAIN_ID';"
fi
}
 
MASTERS=(`$MYSQL "USE pdns; SELECT DISTINCT ip FROM supermasters;"`)
for m in "${MASTERS[@]}"; do
NAMES=(`$MYSQL "USE pdns; SELECT name FROM domains WHERE type = 'SLAVE' AND master = '${m}';"`)
for d in "${NAMES[@]}"; do
check ${m} ${d}
done
done
</pre></code>
