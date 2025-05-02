# honeynet-aws-tpot
Honeynet with T-Pot, Filebeat, Logstash, OpenSearch, and Terraform

# Project Summary
This project is a honeynet designed to attract, log, and analyze malicious internet traffic. Built using T-Pot CE and deployed to AWS EC2, it leverages Filebeat (v7.10) for log shipping, Logstash for parsing, and OpenSearch 2.19 for indexing. Terraform is used to automate infrastructure provisioning and enforce consistency across deployments.

# Architecture Diagram
<img width="1135" alt="Tpot flowchart" src="https://github.com/user-attachments/assets/6762cd66-6d54-4a67-8781-0d6f4b25ca46" />


# Infrastructure Overview
Platform: AWS EC2 (Ubuntu 22.04)

T-Pot: Community Edition (Dockerized honeypot sensors)

Filebeat: Version 7.10 (to ensure compatibility with OpenSearch)

Logstash: Latest version (as of 2025)

OpenSearch: Managed OpenSearch v2.19 domain (Amazon OpenSearch Service)

Automation: Terraform (AWS Provider)

# Deployment Steps
Provision EC2 Instance

    EC2 type: t3.xlarge

    OS: Ubuntu 22.04

    Open ports: 22, 5044, 443, 9200, 5601

Install T-Pot CE

    Cloned from the official GitHub

    Installed via install.sh

    Containers automatically run multiple honeypots (e.g., Cowrie, Dionaea, ElasticPot)



# Security Considerations

Filebeat 8+ requires Elastic license; not compatible with OpenSearch.

SSL is used for OpenSearch, but certificate verification is disabled for ease of setup.

All access is IP-restricted via AWS Security Groups.

Logs are harvested from Docker containers using JSON decode

# Sample Log Event

# Future Enhancements

Add OpenSearch Dashboards visualizations

Setup alerting for malicious behavior

Expand deployment via multi-region/multi-cloud Terraform

Create Ansible playbook for config automation

# ❌ What I Did Wrong

This project came with some pain—and lessons. Here's where things went sideways:

  Installed Filebeat 8.x first
    Initially used the latest Filebeat version, which broke communication with OpenSearch 2.19 due to x-pack /license checks. It required a downgrade to 7.10 to regain compatibility.

  Tried direct Filebeat → OpenSearch
    The plan was to keep it simple, but OpenSearch rejected Filebeat events due to missing license compatibility. This prompted the switch to a Logstash intermediary.

  Assumed OpenSearch would mimic Elasticsearch exactly
    Many community guides reference Elasticsearch behaviors (e.g., _license endpoint), but OpenSearch isn't a 1:1 replacement. Some endpoints and features don’t exist or behave differently.

  Didn’t open port 5044 for Logstash early on
    Filebeat kept failing to connect to Logstash (connection refused) until the port was allowed and Logstash was properly configured and running.

  Ran Filebeat without live harvester logs
    Several times, Filebeat appeared "healthy" but wasn’t actually sending any logs due to stale or inactive Docker log files. Took time to realize which containers were producing logs and when.
