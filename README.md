# Checkmarx Health Monitor

The Checkmarx Health Monitor is a tool that monitors the following:
* Queue
    * Number of scans in queue
    * How long a scan remains in the 'Queued' state
* Scans
    * Slow running scan
    * Scan failure
* Engine
    * Responsiveness
    * Offline
* Portal
    * Responsiveness
* Audits
    * Project changes
    * Query changes
    * Preset changes
    * Results Severity, State, Assignment changes

## Getting Started

The tool consists of two files - a Powershell script (the monitor) and a JSON file (the configuration)
* CxHealthMonitor.ps1
* cx_health_mon_config.json


### Prerequisites

* Powershell V5 (Windows 10 has powershell 5.1 installed)
   * https://docs.microsoft.com/en-us/powershell/scripting/install/installing-windows-powershell?view=powershell-6
* Invoke-SqlCmd2 module
   * Run : Install-Module -Name Invoke-SqlCmd2
* Access to the Checkmarx Server and Database


## Usage

```
.\CxHealthMonitor.ps1 [-cxUser username] [-cxPass password] [-dbUser username] [-dbPass password]
```

The optional arguments will override the corresponding values provided in the configuration file. 

Note: If the optional db parameters are skipped and the corresponding entries in the config file are empty, the monitor will use SQLServer authentication.

## Configuration

The configuration file (cx_health_mon_config.json) consists of the following sections:
* cx
* monitor
* alerts
* alertingSystems

The **"cx"** section drives connectivity to the Checkmarx server and database. The Checkmarx Server URL, Checkmarx manager connection credentials, database instance and database connection credentials are configured here.
```json
"cx": {
        "host": "http://checkmarx.domain.com",
        "username": "your_account",
        "password": "your_password",
        "db": {
            "instance": "localhost\\SQLExpress",
            "username": "",
            "password": ""
        }
    }
```

The **"monitor"** section is used to configure thresholds and monitoring parameters. 
```json
"monitor": {
        "pollIntervalSeconds": 30,
        "thresholds": {
            "queuedScansThreshold": 5,
            "queuedTimeThresholdMinutes": 20,
            "scanDurationThresholdMarginPercent": 25.0,
            "scanRateAsLOCPerHour": 150000,
            "engineResponseThresholdSeconds": 60.0,
            "restResponseThresholdSeconds": 60.0
        },
        "apiResponseTimeoutSeconds": 120,
        "retries": 5
    }
```
* _pollIntervalSeconds_: Polling cadence - how often the monitor will connect to the Checkmarx server for monitoring purposes.
* _queuedScansThreshold_: Threshold for the maximum number of scans in the CxSAST Queue, beyond which alerts will be sent.
* _queuedTimeThresholdMinutes_: Threshold for the number of minutes a scan can remain in the CxSAST Queue, beyond which alerts will be sent.
* _scanDurationThresholdMarginPercent_: Additional duration (added as a percentage) to a scan's estimated duration, beyond which the scan will be marked 'slow'.
* _scanRateAsLOCPerHour_: The scan rate, in Lines Of Code per hour - to be used in estimating a scan's expected duration.
* _engineResponseThresholdSeconds_: Threshold (in seconds) for an engine to respond to the monitor's API call.
* _restResponseThresholdSeconds_: Threshold (in seconds) for the CxManager to respond to the monitor's API call. This is a proxy for Portal responsiveness.
* _apiResponseTimeoutSeconds_: Specified in seconds, this is how long the monitor will wait for a response from the monitored system, before timing out and trying again (see 'retries').
* _retries_: Number of times the monitor will attempt to connect to the monitored system before giving up.


The **"alerts"** section is used to configure values specific to Alerts.
 
```json
"alerts": {
        "waitingPeriodBetweenAlertsMinutes": 15
    }
```
* _waitingPeriodBetweenAlertsMinutes_: Period in minutes, to wait before sending out subsequent alerts arising from the same monitored subject and the same conditions. This configuration controls/prevents alert flooding.

The **"alertingSystems"** section is used to configure available Alerting Systems to be used by the monitor.
The monitor ships with multiple Alerting System implementations, such as Email(smtp), Syslog and Event Logs. When new implementations (such as SNMP etc.) are available, this is where they should be configured.

```json
"alertingSystems": {
        "smtp": [
            {
                "systemType": "smtp",
                "name": "Email",
                "host": "007-myemailserver.com",
                "port": 587,
                "user": "someuser@myemailserver.com",
                "password": "somepassword",
                "sender": "admin@myemailserver.com",
                "recipients": "list@of.com, email@addresses.com",
                "subject": "Checkmarx Health Monitor Alert"
            }
        ],
        "syslog": [
            {
                "systemType": "syslog",
                "name": "Kiwi",
                "host": "localhost",
                "port": 514
            },
            {
                "systemType": "syslog",
                "name": "Splunk",
                "host": "localhost",
                "port": 515
            }
        ]
    }
```

The **"log"** section is used to configure the JSON output directory to be used by the monitor.
The _jsonDirectory_ element specifies where the JSON files output by the monitor should be written.

```json
    "log": {
        "jsonDirectory": "json"
    }
```    
    
## Authors

* Checkmarx Professional Services - *Initial work*


## License

This project is licensed under **TBD**
