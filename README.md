# aws-splunk-linux-security-lab
Hands-on AWS security monitoring lab using Linux, Splunk Enterprise, Universal Forwarder, and centralized log analysis.
# AWS Splunk Linux Security Monitoring Lab

## Overview

This project is a hands-on security monitoring lab built in Amazon Web Services (AWS) to develop practical experience with Linux administration, Splunk, centralized logging, and cloud networking.

The lab uses multiple EC2 instances within an AWS VPC. An Ubuntu Linux server runs the Splunk Universal Forwarder to collect system and authentication logs and forward them to a Splunk Enterprise server for centralized indexing and analysis.

The environment will also include a Red Hat Enterprise Linux (RHEL) server for RHCSA administration practice and additional security monitoring exercises.

## Architecture

```text
                         AWS VPC
                            |
             +--------------+--------------+
             |                             |
             v                             v
       Ubuntu EC2                     Splunk EC2
     Linux Log Source              Splunk Enterprise
             |                             ^
             |                             |
             | Splunk Universal Forwarder  |
             +-------- TCP 9997 -----------+
```

### Planned Expansion

```text
                         AWS VPC
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
      RHEL EC2          Ubuntu EC2        Splunk EC2
     RHCSA Lab          Log Source     Splunk Enterprise
          |                 |                 ^
          |                 |                 |
          +-------- Splunk Forwarders -------+
                        TCP 9997
```

## Technologies

* Amazon Web Services (AWS)
* Amazon EC2
* AWS VPC
* AWS Security Groups
* Linux
* Ubuntu Server
* Splunk Enterprise
* Splunk Universal Forwarder
* rsyslog
* TCP/IP

## Current Lab Configuration

### Splunk Server

The Splunk Enterprise server is hosted on an Amazon EC2 Linux instance.

Splunk is configured to receive forwarded events on:

```text
TCP 9997
```

Splunk Web is used to search and analyze collected events.

A dedicated index was created for Linux events:

```spl
index=linux
```

### Ubuntu Log Source

An Ubuntu EC2 instance acts as a Linux log source.

Splunk Universal Forwarder was installed and configured to monitor:

```text
/var/log/syslog
/var/log/auth.log
```

The Universal Forwarder sends these events to the Splunk Enterprise server over TCP port 9997 using private AWS VPC networking.

## AWS Network Security

AWS Security Groups are used to restrict communication between systems.

The Splunk receiving port (TCP 9997) is accessible from the security group assigned to authorized forwarding instances rather than being exposed publicly.

Administrative services such as SSH and Splunk Web are restricted to trusted source addresses.

## Verification

Network connectivity between the Ubuntu forwarder and Splunk server was tested using:

```bash
nc -vz <splunk-private-ip> 9997
```

A successful TCP connection confirmed that the forwarding instance could reach the Splunk receiver.

The Splunk Universal Forwarder connection was verified with:

```bash
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

The Splunk server appeared under **Active forwards**.

## Log Ingestion

The following inputs were configured on the Universal Forwarder:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -index linux -sourcetype linux_secure

sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -index linux -sourcetype syslog
```

Successful ingestion was verified in Splunk using:

```spl
index=linux
```

The lab successfully ingested Linux system and authentication events from the Ubuntu EC2 instance.

## Example SPL Searches

View all Linux events:

```spl
index=linux
```

Count events by sourcetype:

```spl
index=linux
| stats count by sourcetype
```

Count events by log source:

```spl
index=linux
| stats count by source
```

Display event volume over time:

```spl
index=linux
| timechart count
```

Search authentication events:

```spl
index=linux sourcetype=linux_secure
```

## Troubleshooting Experience

Several issues were identified and resolved while building the lab.

### Splunk File Permissions

Splunk initially encountered Linux file permission errors because the installation had previously been executed under a different user context.

File ownership and service execution were corrected so Splunk could operate under a dedicated non-root account.

### Network Connectivity

Connectivity between EC2 instances was validated over the private AWS network using `nc`.

AWS Security Groups were configured to permit Splunk forwarding traffic on TCP 9997 between authorized instances.

### Splunk Licensing

The original Splunk Enterprise trial license had expired, preventing searches from executing.

The environment was converted to Splunk Free for continued non-production lab use.

### Log Forwarding

Splunk Universal Forwarder was installed on Ubuntu and configured to send Linux logs to the Splunk Enterprise instance.

Forwarder status and connectivity were verified before configuring log inputs.

## Skills Practiced

This project provides hands-on experience with:

* AWS EC2 deployment and administration
* AWS VPC networking
* AWS Security Groups
* Linux administration
* Linux permissions and ownership
* Linux system and authentication logging
* Network connectivity troubleshooting
* Splunk Enterprise administration
* Splunk Universal Forwarder
* Splunk indexes and sourcetypes
* Centralized log collection
* SPL searching and analysis
* Basic security monitoring

## Future Improvements

Planned additions to the lab include:

* Deploy a Red Hat Enterprise Linux server
* Practice RHCSA administration tasks
* Forward RHEL logs to Splunk
* Monitor SSH authentication activity
* Monitor sudo activity
* Detect failed authentication attempts
* Monitor user and group changes
* Monitor service configuration changes
* Monitor SELinux events
* Create Splunk dashboards
* Develop security-focused SPL searches and detections
* Document RHCSA exercises and troubleshooting scenarios

## Project Purpose

The goal of this project is to combine Linux administration, cloud infrastructure, and security monitoring into a single hands-on environment.

Rather than working only with pre-generated datasets, this lab generates and collects real Linux system activity from cloud-hosted servers and uses Splunk to investigate and analyze that activity.
