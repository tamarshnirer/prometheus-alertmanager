# Prometheus Alertmanager Project

## Project Overview:
This project implements a monitoring and alerting solution using Prometheus and Alertmanager for an Amazon EC2 instance. It tracks CPU utilization and sends real-time alerts to a designated Slack channel if CPU usage exceeds 85%.

## Key Features:

- Monitoring: Prometheus collects metrics from the EC2 instance.
- Alerting: Alertmanager processes Prometheus alerts and triggers a notification when the threshold is breached.
- Integration: Slack integration alerts are delivered to the #alerts channel.

## Setup Instructions

### Prerequisites
- 2 Ubuntu servers with 20.04 or higher (One for serving as a target and one for running prometheus and alertmanager. 
- A Slack Workspace.

## Steps
### 1. Setting Up the Target:
- Create an EC2 or any physical or virtual machine with an Ubuntu server with a public IP.
  I used an EC2 of type t2,micro. with 22.04. I generated an elastic ip to have a consistent public ip address.
- Configure a security group to allow inbound traffic from port 9093 to expose its metrics and port 22 to SSH connection.
- Connect to the instance using SSH.
- Download Node Exporter using the following commands:
sudo apt update
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar xvfz node_exporter-1.8.2.linux-amd64.tar.gz
- Create a user to run the service.
sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
sudo chown -R node_exporter:node_exporter /home/ubuntu/node_exporter-1.8.2.linux-amd64/
sudo chmod +x /home/ubuntu/node_exporter-1.8.2.linux-amd64/node_exporter
sudo apt install acl
sudo setfacl -m u:node_exporter:x /home/ubuntu/node_exporter-1.8.2.linux-amd64/node_exporter
sudo getfacl /home/ubuntu/node_exporter-1.8.2.linux-amd64/node_exporter
sudo setfacl -m u:node_exporter:x /home/ubuntu/node_exporter-1.8.2.linux-amd64
- Set a service to run the Node Exporter.
vim /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/home/ubuntu/node_exporter-1.8.2.linux-amd64/node_exporter

[Install]
WantedBy=multi-user.target

- Run the service, make sure its active.                            
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter

- Download the stress command for testing purposes:
sudo apt install -y stress

### 2. Creating a Slack webhook.
- In your slack workspace, create a dedicated channel for the Prometheus notification. (e.g. #alerts)
- Create a Slack App by going to https://api.slack.com/apps
- Click "Create New App" and choose "From scratch"
- Name your app (e.g., "Prometheus Alerts") and select your workspace
- In your app's settings, find "Incoming Webhooks" in the features menu
- Toggle the switch to enable incoming webhooks
- Click "Add New Webhook to Workspace"
- Select the #alerts channel (or create it if it doesn't exist)
- Authorize the app to post to the selected channel
- Copy the webhook URL (it should look like https://hooks.slack.com/services/XXXX/XXXX/XXXX)
### 3. Setting Up the Monitoring Server
- Login or SSH to your 2nd Ubuntu machine (I used ubuntu 22.04 over wsl)
- clone this repository to your ubuntu machine.
```bash
git clone https://github.com/tamarshnirer/prometheus-alertmanager.git
cd prometheus-alertmanager
```
- Create two environment variables: The public IP of the EC2 and the slack webhook.
export TARGET_IP=<your_target_ip>
export SLACK_API_WEBHOOK=<your_slack_webhook>
Be sure to keep them secure, it can be done by using a 3rd party vault.
- Plugin those env var to the YAML files.
sed -i '' -e "s/TARGET_IP/$TARGET_IP/" prometheus.yml
sed -i '' -e "s/SLACK_API_WEBHOOK/$SLACK_API_WEBHOOK/" alertmanager.yml



### 4. Testing the system
