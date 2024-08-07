---
layout: post
title: Hardening Postgres 101!
subtitle: Locking down databases!.
date: 2023-02-30 18:30:00 +0000
tags:
- monitoring
- postgresql
---


If you have been following ym little series on postgres then you would have seen the littte intro to ssl certs for your database. I wanted to have a little primer for some of the basics and I think we should start with some things that I was told consist of the basics.

- A script for updating certs
- Locking down the pg config
- Enable monitoring and logging.

So let's get started with this.

## Regularly Update Certificates

Step 1: Create a Script to Renew Certificates

Create a script to renew the self-signed certificates.

```bash
sudo nano /usr/local/bin/renew_postgres_certificates.sh
```

Add the following script:
```bash
#!/bin/bash

# Variables
CA_KEY="/etc/ssl/postgresql/ca.key"
CA_CERT="/etc/ssl/postgresql/ca.crt"
SERVER_KEY="/etc/ssl/postgresql/server.key"
SERVER_CSR="/etc/ssl/postgresql/server.csr"
SERVER_CERT="/etc/ssl/postgresql/server.crt"
CLIENT_KEY="/etc/ssl/postgresql/client.key"
CLIENT_CSR="/etc/ssl/postgresql/client.csr"
CLIENT_CERT="/etc/ssl/postgresql/client.crt"

# Renew CA certificate (every 5 years for example)
# Ensure you renew before it expires
openssl req -new -x509 -days 1825 -key $CA_KEY -subj "/CN=MyPostgresCA" -out $CA_CERT

# Renew server certificate
openssl req -new -key $SERVER_KEY -subj "/CN=yourhostname" -out $SERVER_CSR
openssl x509 -req -in $SERVER_CSR -CA $CA_CERT -CAkey $CA_KEY -CAcreateserial -out $SERVER_CERT -days 365

# Renew client certificate
openssl req -new -key $CLIENT_KEY -subj "/CN=clienthostname" -out $CLIENT_CSR
openssl x509 -req -in $CLIENT_CSR -CA $CA_CERT -CAkey $CA_KEY -CAcreateserial -out $CLIENT_CERT -days 365

# Restart PostgreSQL
systemctl restart postgresql
```
Make the script executable:

```bash
sudo chmod +x /usr/local/bin/renew_postgres_certificates.sh
```

## Step 2: Automate the Renewal Process
Use `cron` to automate the script execution.

```bash
sudo crontab -e
```

Add the following line to run the script monthly (adjust the frequency as needed):
```bash
'0 0 1 * * /usr/local/bin/renew_postgres_certificates.sh'
```

This takes care of our certs let's move to locking down the hba_config.

## Harden PostgreSQL Configuration

#### Step 1: Restrict Access
Edit the `pg_hba.conf` file to restrict access:
```bash
'sudo nano /etc/postgresql/12/main/pg_hba.conf'
```
Ensure only trusted IPs can access the database:

```bash
# Example: Allow only localhost and specific IP range
host    all             all             127.0.0.1/32            md5
host    all             all             192.168.1.0/24          md5
```
### Step 2: Use Strong Passwords

Ensure all users have strong passwords. Change a user's password with:
```bash
ALTER USER youruser WITH PASSWORD 'newstrongpassword';
```
## Monitoring and Logging

### Step 1: Configure Logging

Edit the `postgresql.conf` file to enable detailed logging:
```bash
# Enable logging
logging_collector = on
log_directory = 'log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_rotation_age = 1d
log_rotation_size = 10MB

# Log connections and disconnections
log_connections = on
log_disconnections = on

# Log duration of each completed statement
log_duration = on

# Log all statements
log_statement = 'all'
```
then just restart postgres to apply changes.

Bonus Mentions: Install and Configure Monitoring Tools

You have a few options for this  I like pgbadger, pgadmin the best I think they are great tools.

Otherwise. you should have a solid foundation on hardening your databases the next post will include some more pg security information. 
