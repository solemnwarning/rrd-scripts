# rrd-scripts

This repository contains scripts I use for scraping various data sources into RRD (Round Robin Database) files for use in reporting/dashboards.

If you're looking for a way to feed this data into Grafana, consider [grafana-rrd-server](https://github.com/doublemarket/grafana-rrd-server).

## ifconfig-stats-to-rrd

Records the count of bytes/packets sent/received on an interface since the last invocation of the script. For example to record the statistics every minute and keep 6 months of history, create a database with the following command:

```
rrdtool create /srv/rrd/ppp0-stats.rrd -s 60 \
	DS:rx_packets:GAUGE:300:0:U \
	DS:rx_bytes:GAUGE:300:0:U \
	DS:tx_packets:GAUGE:300:0:U \
	DS:tx_bytes:GAUGE:300:0:U \
	RRA:AVERAGE:0.5:1m:6M
```

And put this in your crontab:

```
* * * * * ifconfig-stats-to-rrd ppp0 /srv/rrd/ppp0-stats.rrd
```

## ping-rtt-to-rrd

Records the rtt (round trip time) of pinging a given host. For example to keep a record of the latency of your Internet connection you could create a database like this:

```
rrdtool create /src/rtt-google.rrd -s 60 \
	DS:rtt:GAUGE:300:0:U \
	RRA:AVERAGE:0.5:1m:6M \
	RRA:AVERAGE:0.5:5m:2y
```

And put this in your crontab:

```
* * * * * ping-rtt-to-rrd google.com /srv/rrd/rtt-google.rrd
```

## shelly-v1-em-to-rrd

Records the current voltage and energy used/returned (in Wh) from one of the clamp meters on a Shelly EM (or compatible) device using the v1 HTTP API.

To create the database:

```
rrdtool create /srv/rrd/shelly0.rrd -s 60 \
	DS:wh:GAUGE:300:0:U \
	DS:wh_returned:GAUGE:300:0:U \
	DS:volts:GAUGE:300:0:U \
	RRA:AVERAGE:0.5:1m:2y
```

To record the data from meter 0 in your crontab:

```
* * * * * shelly-v1-em-to-rrd shellyem-XXXXXXXXXXXX 0 /srv/rrd/shelly0.rrd /srv/rrd/shelly0.state [<username>] [<password>]
```

The `shelly0.state` file is used to track the previous counter values reported by the device between invocations of the script.

**NOTE**: The password passed to this script may be leaked from the process table, so this script isn't suitable for accessing secured Shelly devices on a machine with other users or untrusted services.

## shelly-v2-pm-to-rrd

Records the current voltage/temperature and energy used over the past minute (as W averaged over the minute, or Wm) from a Shelly Plus 1PM (or compatible) device using the v2 HTTP API.

To create the database:

```
rrdtool create /srv/rrd/shelly.rrd -s 60 \
	DS:watts:GAUGE:300:0:U \
	DS:volts:GAUGE:300:0:U \
	DS:tempC:GAUGE:300:0:U \
	RRA:AVERAGE:0.5:1m:2y
```

To record the data from your crontab:

```
* * * * * shelly-v2-pm-to-rrd shellyplus1pm-XXXXXXXXXXXX /srv/rrd/shelly.rrd [<password>]
```

**NOTE**: The password passed to this script may be leaked from the process table, so this script isn't suitable for accessing secured Shelly devices on a machine with other users or untrusted services.
