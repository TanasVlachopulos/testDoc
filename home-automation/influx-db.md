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

### Continuous queries for collecting data from sensors

```text
CREATE CONTINUOUS QUERY cq_10m_for_2w ON home BEGIN SELECT mean(value) INTO "10m_for_2w".meteo_data_10m_summary FROM meteo_data GROUP BY time(10m) END
CREATE CONTINUOUS QUERY cq_30m_forever ON home BEGIN SELECT mean(*) INTO home."30m_forever".meteo_data_30m_summary FROM home.meteo_data GROUP BY time(30m) END

CREATE CONTINUOUS QUERY cq_10m_for_2w ON home BEGIN SELECT mean(value) AS value INTO "10m_for_2w".meteo_data_10m_summary FROM meteo_data GROUP BY time(10m), data_type, device END
CREATE CONTINUOUS QUERY cq_30m_forever ON home BEGIN SELECT mean(*) INTO home."30m_forever".meteo_data_30m_summary FROM home.meteo_data GROUP BY time(30m) END

CREATE CONTINUOUS QUERY cq_10m_for_2w_2 ON home BEGIN SELECT mean(value) AS value INTO "10m_for_2w".meteo_data_10m_summary FROM meteo_data GROUP BY time(10m) END

CREATE CONTINUOUS QUERY cq_10m_for_2w ON home BEGIN SELECT mean(value) AS value INTO meteo_data_10m_summary FROM meteo_data GROUP BY time(10m), * END

CREATE CONTINUOUS QUERY cq_10m_for_2w ON home BEGIN SELECT mean(value) AS value INTO "10m_for_2w".meteo_data_10m_summary FROM meteo_data GROUP BY time(10m), * END
CREATE CONTINUOUS QUERY cq_30m_forever ON home BEGIN SELECT mean(value) AS value INTO "30m_forever".meteo_data_30m_summary FROM meteo_data GROUP BY time(30m), * END
```

### 







