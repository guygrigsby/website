---
title: "Open Source Flex"
date: 2020-03-03T18:42:44-07:00
categories: ["Personal", "Open Source"]

---

![Fancy Certificate from Comcast for Contributing to Trickster](/open-source-flex.png)

Just wanted to do a little bragging. I was helping a prev colleague from Comcast on a dashboard accelerator / reverse proxy server called [Trickster](https://github.com/Comcast/trickster). I cranked out some basic support for [OpenTelemetry](https://opentelemetry.io) tracing right before the dropped the official v1 and they gave me a certificate! Pretty sweet huh?

If you use Prometheus or another time series and have found that your Grafana dashboards are slow to load, I would recommend taking a look. It's a cache that sits between the dashboard and the time series DB. And it's *fast*.
