#!/bin/bash

echo "########### Install WazuhA -  Please Wait  ###########"

AGENT_NAME=$(hostname)
WAZUH_USER=''
WAZUH_PASSWORD=''
WAZUH_SERVER=''
AGENT_IP=$1

echo "Install Wazuh"
sudo WAZUH_MANAGER="$WAZUH_SERVER" rpm -hiv 
echo "Request Wazuh API to get key"
echo "curl -u *****:****** -k -X POST -d 'name=${AGENT_NAME}&ip=${AGENT_IP}' http://${WAZUH_SERVER}:55000/agents"
GET_ID=$(curl -s -u "$WAZUH_USER":"$WAZUH_PASSWORD" -k -X POST -d 'name='"$AGENT_NAME"'&ip='"$AGENT_IP" https://"$WAZUH_SERVER":55000/agents)
ERROR=$(echo "$GET_ID" | grep -Po '(?<="error":)[\d^"]*')
if [[ $ERROR -ne 0 ]]; then
  echo "$GET_ID" | grep -oP '(?<="message":")[^"]*'
  exit
fi
API_KEY=$(echo "$GET_ID" | grep -oP '(?<="key":")[^"]*')

echo "Add agent key and restart service."
echo "y" | /var/ossec/bin/manage_agents -i "$API_KEY"
/var/ossec/bin/ossec-control restart

echo "########### Install WazuhA -  Done  ###########"

mv /etc/audit/audit.rules /etc/audit/audit.rules.bk
cp audit.rules 	/etc/audit/audit.rules

service auditd start
service auditd status

chkconfig wazuh-agent on
service wazuh-agent start
chkconfig --list | grep wazuh-agent

