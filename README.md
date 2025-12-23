# Ruckus AP SNMP Exporter

SNMP Exporter for monitoring Ruckus Access Points with Prometheus and Grafana.

Tested with Ruckus R760 running firmware 7.0.0.300.

## Components

- `docker-compose.yml` - SNMP Exporter container
- `snmp.yml` - Ruckus-specific SNMP OID configuration
- `targets.yml.example` - AP target list template
- `ruckus-ap-dashboard.json` - Grafana dashboard

## Setup

### 1. Configure Targets

```bash
cp targets.yml.example targets.yml
# Edit targets.yml and add your AP IP addresses
```

### 2. Start the SNMP Exporter

```bash
docker compose up -d
```

### 3. Configure Prometheus

Copy `targets.yml` to your monitoring folder:
```bash
cp targets.yml /path/to/monitoring/ruckus-targets.yml
```

Add to `prometheus.yml`:
```yaml
  - job_name: 'ruckus-ap'
    scrape_interval: 30s
    scrape_timeout: 20s
    file_sd_configs:
      - files:
          - /etc/prometheus/ruckus-targets.yml
        refresh_interval: 30s
    metrics_path: /snmp
    params:
      module: [ruckus]
      auth: [public_v2]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: ruckus-snmp-exporter:9116
```

Mount the targets file in Prometheus container and restart.

### 4. Import Dashboard

1. Open Grafana
2. Go to Dashboards → Import
3. Upload `ruckus-ap-dashboard.json`
4. Select your Prometheus datasource

## Adding APs

Edit `targets.yml`:
```yaml
- targets:
    - 192.168.1.100
    - 192.168.1.101
  labels:
    job: ruckus-ap
```

Then copy to monitoring folder and Prometheus will pick up changes automatically.

## Metrics Available

| Category | Metrics |
|----------|---------|
| System | CPU, Memory, Uptime, AP Model, Serial Number |
| Interface | ifHCInOctets, ifHCOutOctets (64-bit counters) |
| WLAN | Per-SSID client count, TX/RX bytes per WLAN |

### Metric Names
- `ruckusSystemCPUUtil` - CPU utilization percentage
- `ruckusSystemMemoryUtil` - Memory utilization percentage
- `ruckusAPModel` - AP model (e.g., R760)
- `ruckusAPSerialNumber` - AP serial number
- `ruckusWLANSSID` - SSID names with wlanIndex label
- `ruckusWLANNumSta` - Client count per WLAN
- `ruckusWLANRxBytes` - Received bytes per WLAN
- `ruckusWLANTxBytes` - Transmitted bytes per WLAN

## Test SNMP Connection

```bash
curl "http://localhost:19116/snmp?target=<AP_IP>&module=ruckus&auth=public_v2"
```

## Requirements

- SNMP enabled on Ruckus AP (SNMPv2c, community: public)
- Network access to AP on UDP port 161
