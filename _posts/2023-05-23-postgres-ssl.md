---
layout: post
title: Configuring Postgres with SSL.
subtitle: Playing with Monitoring.
date: 2023-05-12 18:30:00 +0000
tags:
- security
- postgres
---



So I started my desecent into the postgres world and I got fascinated with security. So I started testing around and started playing with security. Here is a simple guide to start hardening your postgres instance


# Setting Up PostgreSQL with SSL Encryption on Debian

In this guide, we'll walk through the steps to set up PostgreSQL with SSL encryption on a Debian machine. This ensures that the data transferred between your PostgreSQL server and clients is encrypted, providing an added layer of security.

## Step 1: Install PostgreSQL and OpenSSL

First, ensure that PostgreSQL and OpenSSL are installed on your Debian machine:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib openssl

```

## Step 2: Generate SSL Certificates

Create a directory to store the SSL certificates and generate a self-signed certificate and private key.

```bash
sudo mkdir /etc/ssl/postgresql
sudo openssl req -new -text -passout pass:yourpassword -subj /CN=yourhostname -out /etc/ssl/postgresql/server.req
sudo openssl rsa -in privkey.pem -passin pass:yourpassword -out /etc/ssl/postgresql/server.key
sudo openssl req -x509 -in /etc/ssl/postgresql/server.req -text -key /etc/ssl/postgresql/server.key -out /etc/ssl/postgresql/server.crt
```

Change the permissions to ensure only the PostgreSQL user can read the key file.


```bash
sudo chmod 600 /etc/ssl/postgresql/server.key
sudo chown postgres:postgres /etc/ssl/postgresql/server.key
sudo chown postgres:postgres /etc/ssl/postgresql/server.crt
```

## Step 3: Configure PostgreSQL for SSL

Edit the PostgreSQL configuration file to enable SSL and set the paths to your certificate and key files.

```bash
sudo nano /etc/postgresql/12/main/postgresql.conf
```
Ensure the following lines are present and correctly set:
```bash
ssl = on
ssl_cert_file = '/etc/ssl/postgresql/server.crt'
ssl_key_file = '/etc/ssl/postgresql/server.key'
```

## Step 4: Update pg_hba.conf for SSL Connections
Edit the `pg_hba.conf` file to allow SSL connections.
```bash
sudo nano /etc/postgresql/<version>/main/pg_hba.conf`
```
Add the following line to allow SSL connections:
```bash
'hostssl all all 0.0.0.0/0 md5'
```

## Step 5: Restart PostgreSQL
Restart the PostgreSQL service to apply the changes.
```bash
sudo systemctl restart postgresql
```

## Step 6: Verify SSL Configuration
Check if the server is running with SSL enabled:
```bash
sudo -u postgres psql -c "SHOW ssl;"
```

the result should be "on"
```bash
postgres@vultr:~$ psql -c "SHOW ssl;"
 ssl 
-----
 on
(1 row)


```

now lets create a new database and test are changes.

## Step 7: Create a Database and User

Create a test database and user to verify the SSL connection.

```bash
sudo -u postgres createuser testuser
sudo -u postgres createdb testdb -O testuser
sudo -u postgres psql
postgres=# ALTER USER testuser WITH ENCRYPTED PASSWORD 'yourpassword';

```

## Step 8: Connect Using SSL
Ensure your client connection string explicitly requires SSL. For example, using `psql`:
```bash 
psql "host=localhost dbname=testdb user=testuser password=yourpassword sslmode=require"
```

## Step 9: Verify SSL Connection
After connecting with SSL, check the `pg_stat_ssl` view:
```bash
SELECT * FROM pg_stat_ssl;
```

and you should see something like this:
```bash
testdb=> SELECT * FROM pg_stat_ssl;
  pid  | ssl | version |         cipher         | bits | client_dn | client_serial | issuer_dn 
-------+-----+---------+------------------------+------+-----------+---------------+-----------
 41427 | t   | TLSv1.3 | TLS_AES_256_GCM_SHA384 |  256 |           |               | 
(1 row)

```

and that's it regarding setting up encryption. In the next part of this primer we are going to setup client authentication and do some hardening on the database.
