# Ruckus AP SNMP Exporter

SNMP Exporter for monitoring Ruckus Access Points with Prometheus and Grafana.

Tested with Ruckus R760 running firmware 7.0.0.300.

## Components

- `docker-compose.yml` - SNMP Exporter and Prometheus containers
- `snmp.yml` - Ruckus-specific SNMP OID configuration
- `prometheus.yml` - Prometheus scrape configuration
- `targets.yml.example` - AP target list template
- `ruckus-ap-dashboard.json` - Grafana dashboard

## Setup

### 1. Configure Targets

```bash
cp targets.yml.example targets.yml
# Edit targets.yml and add your AP IP addresses
```

### 2. Start the Stack

```bash
docker compose up -d
```

This starts both the SNMP Exporter and a dedicated Prometheus instance.

### 3. Add Prometheus Data Source in Grafana

1. Open Grafana (e.g., http://localhost:13000)
2. Go to Configuration → Data Sources → Add data source
3. Select **Prometheus**
4. Set URL to: `http://ruckus-prometheus:9090`
5. Click **Save & Test**

### 4. Import Dashboard

1. Go to Dashboards → Import
2. Upload `ruckus-ap-dashboard.json`
3. Select the Ruckus Prometheus data source you just added

## Adding APs

Edit `targets.yml`:
```yaml
- targets:
    - 192.168.1.100
    - 192.168.1.101
  labels:
    job: ruckus-ap
```

Prometheus will automatically pick up changes within 30 seconds.

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
