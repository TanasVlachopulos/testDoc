# InfluxDb

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





