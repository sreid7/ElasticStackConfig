# ElasticStackConfig
The full Bash script for installing and configuring Elastic Stack on Kali Linux 


#!/bin/bash

# Elastic Stack Installation and Configuration Script for Kali Linux
# Author: Sheldon Reid
# Description: Script to automate Elastic Stack installation and configuration on Kali Linux.

# Update and Upgrade System
echo "[+] Updating and upgrading the system..."
sudo apt update && sudo apt upgrade -y

# Install Required Dependencies
echo "[+] Installing dependencies..."
sudo apt install -y apt-transport-https openjdk-11-jdk curl gnupg

# Add Elastic Stack GPG Key and Repository
echo "[+] Adding Elastic Stack GPG key and repository..."
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list

# Install Elasticsearch
echo "[+] Installing Elasticsearch..."
sudo apt update && sudo apt install -y elasticsearch

# Configure Elasticsearch
echo "[+] Configuring Elasticsearch..."
sudo sed -i '/^#network.host:/c\network.host: 0.0.0.0' /etc/elasticsearch/elasticsearch.yml
sudo sed -i '/^#http.port:/c\http.port: 9200' /etc/elasticsearch/elasticsearch.yml
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

# Install Logstash
echo "[+] Installing Logstash..."
sudo apt install -y logstash

# Configure Logstash
echo "[+] Configuring Logstash..."
LOGSTASH_CONF="/etc/logstash/conf.d/logstash-simple.conf"
sudo mkdir -p /etc/logstash/conf.d
cat <<EOL | sudo tee $LOGSTASH_CONF
input {
  beats {
    port => 5044
  }
}

filter {
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
EOL
sudo systemctl enable logstash
sudo systemctl start logstash

# Install Kibana
echo "[+] Installing Kibana..."
sudo apt install -y kibana

# Configure Kibana
echo "[+] Configuring Kibana..."
sudo sed -i '/^#server.host:/c\server.host: "0.0.0.0"' /etc/kibana/kibana.yml
sudo sed -i '/^#elasticsearch.hosts:/c\elasticsearch.hosts: ["http://localhost:9200"]' /etc/kibana/kibana.yml
sudo systemctl enable kibana
sudo systemctl start kibana

# Configure Firewall for Elastic Stack
echo "[+] Configuring firewall..."
sudo ufw allow 9200/tcp
sudo ufw allow 5601/tcp
sudo ufw allow 5044/tcp
sudo ufw enable

# Verify Services
echo "[+] Verifying Elastic Stack services..."
systemctl status elasticsearch
systemctl status logstash
systemctl status kibana

# Final Message
echo "[+] Elastic Stack installation and configuration is complete!"
echo "Access Kibana at: http://<your-ip>:5601"
echo "Ensure your system is secure before using this in production."
