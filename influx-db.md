# Influx DB

### Manage database 

For management from machine there is CLI client called **influx**. Simple run `influx` command and client will automatically connect to default database localhost on port 8086.

#### Commands

```sql
SHOW DATABASES
CREATE DATABASE mydb
USE mydb

-- create admin user 
CREATE USER admin WITH PASSWORD '123' WITH ALL PRIVILEGES

-- create non admin user 
CREATE USER nodered WITH PASSWORD '123'
GRANT READ ON home TO nodered
GRANT WRITE ON home TO nodered

```

### Influx links

{% embed url="https://docs.influxdata.com/influxdb/v1.7/introduction/getting-started/" caption="Create DB and writing data" %}

{% embed url="https://docs.influxdata.com/influxdb/v1.7/administration/authentication\_and\_authorization/\#authorization" caption="Add DB users" %}









