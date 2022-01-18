---
title: "PowerDNS With Sqlite3 on Ubuntu 20.04"
date: 2021-10-23T18:57:14+01:00
draft: true
author: "M6WIQ"
---

## Objective

The following serves as a guide for the configuration of PowerDNS Authoritative Nameservers on Ubuntu 20.04. A lot of this was gathered from various sources (e.g. PowerDNS manuals, 3rd party tutorials) but I simply wanted to note the steps I have taken in configuring PowerDNS for future reference.

## Install Base Packages

```
sudo apt install pdns-backend-sqlite3 sqlite3
```

## Configure pdns.conf

In pdns.conf, you need to state what the backend source for PowerDNS is. There are numerous choices of backend available to suit use cases (e.g. MySQL, PostgreSQL etc). I have chosen SQLite for this tutorial as in small deployments (e.g. 2-3 zones) I believe this is the simplest and most effective solution.

In pdns.conf, the following lines need to be added:

```
launch=gsqlite3
gsqlite3-database=/var/lib/powerdns/pdns.sqlite3
```

Make sure any other enabled launch statements are removed or commented out.

## Creating the Sqlite Database

Outside of PowerDNS, we need to use sqlite directly to configure the *.db file and apply the schema for the database storing the zone configuration.

To create the database, run the following commands:

```
sudo mkdir /var/lib/powerdns
sudo sqlite3 /var/lib/powerdns/pdns.sqlite3 < /usr/share/doc/pdns-backend-sqlite3/schema.sqlite3.sql
sudo chown -R pdns:pdns /var/lib/powerdns
```

## Startup and Testing of PowerDNS

And start PowerDNS

```
sudo systemctl start pdns
```
or

```
sudo systemctl restart pdns
```

Make sure no error is reported, and use systemctl status pdns to make sure PowerDNS was started correctly.

A sample query sent to the server should now return quickly without data:

```
$ dig a www.example.com @127.0.0.1

; <<>> DiG 9.10.3-P4-Debian <<>> a www.example.com @127.0.0.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 40870
...
```

## Adding Zones and Records

Now, let’s add a zone and some records:
```
$ sudo -u pdns pdnsutil create-zone example.com ns1.example.com
Creating empty zone 'example.com'
Also adding one NS record
$ sudo -u pdns pdnsutil add-record example.com '' MX '25 mail.example.com'
New rrset:
example.com. 3005 IN MX 25 mail.example.com
$ sudo -u pdns pdnsutil add-record example.com. www A 192.0.2.1
New rrset:
www.example.com. 3005 IN A 192.0.2.1
```

This should be done as the pdns user (or root), as sqlite3 requires write access to the directory of the database file.

If we now requery our database, www.example.com should be present:

```
$ dig +short www.example.com @127.0.0.1
192.0.2.1

$ dig +short example.com MX @127.0.0.1
25 mail.example.com
```

## Setting up DNSSEC

Firstly, ensure the registrar supports DNSSEC.

