# Locally-Managed Essentials --- Logging and Monitoring

> WatchGuard Firebox / Fireware training handbook

## 1. Learning objectives

By the end of this chapter you should understand:

-   Firebox logging settings
-   The main Firebox log message types
-   Internal Firebox log storage and its limitations
-   Log retention
-   External logging
-   WatchGuard Cloud, Dimension, and third-party Syslog
-   Practical logging configuration and troubleshooting
-   The relationship between logging volume, performance, and retention

------------------------------------------------------------------------

## 2. Logging overview

A Firebox generates log messages for activity that occurs on the device.
WatchGuard documents five major log message types:

1.  **Traffic**
2.  **Alarm**
3.  **Event**
4.  **Debug**
5.  **Statistic**

Logging gives an administrator evidence about what happened, when it
happened, and often which policy or security feature was involved.

``` text
                 FIREBOX LOGGING
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
 Firebox Log       Internal        External
   Settings        Storage          Logging
                                       |
                         +-------------+-------------+
                         |             |             |
                         v             v             v
                   WatchGuard       Dimension      Syslog
                      Cloud
```

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/log_message_types_c.html

------------------------------------------------------------------------

## 3. Firebox log settings

For a locally-managed Firebox, open the Fireware Web UI and go to:

**System → Logging**

Related settings can also be configured from Policy Manager.

WatchGuard recommends configuring a Firebox to send logs to at least one
external location. Internal storage is useful for recent
troubleshooting, but it is limited.

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/set_up_logging_on_device_wsm.html

### Internal storage option

When **Send log message to Firebox Internal storage** is enabled, the
Firebox keeps log messages on its own storage.

Useful for:

-   Quick troubleshooting
-   Checking recent traffic
-   Confirming that a policy is generating logs
-   Investigating a recent event
-   Lab exercises

It should **not** be treated as long-term log archival.

WatchGuard states that Firebox storage for logs is limited and
recommends an external logging destination when logs need to be retained
for later review.

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/logging/logging_settings_configure_web.html

------------------------------------------------------------------------

# 4. Types of Firebox logs

## 4.1 Traffic logs

Traffic logs are generated as the Firebox applies packet-filter and
proxy rules to traffic.

They can help answer:

-   What source IP generated the traffic?
-   What destination was contacted?
-   Which protocol/port was used?
-   Which policy handled the traffic?
-   Was it allowed or denied?

Example:

``` text
Client
10.10.10.25
      |
      | HTTPS / TCP 443
      v
   FIREBOX
      |
      | Allowed
      v
Internet
```

Traffic logging is useful, but logging every allowed connection can
create a very high volume of messages. WatchGuard notes that unnecessary
logging of allowed traffic can increase CPU load and reduce storage
available in logging systems.

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/logging/set_logging_notif_pref_pm_c.html

------------------------------------------------------------------------

## 4.2 Alarm logs

Alarm logs are generated when an alarm condition occurs.

WatchGuard documents categories including:

-   System
-   IPS
-   AV
-   Policy
-   Proxy
-   Counter
-   Denial of Service
-   Traffic

Example:

``` text
Traffic
   ↓
Security inspection
   ↓
Threat / alarm condition
   ↓
ALARM LOG
   ↓
Administrator investigation
```

Alarm logs are particularly useful when investigating security-related
events.

------------------------------------------------------------------------

## 4.3 Event logs

Event logs are associated with user or system activity.

Examples:

-   Device startup/shutdown
-   Device and VPN authentication
-   Process startup/shutdown
-   Hardware problems
-   Administrator actions

If a configuration or authentication event occurred, event logs can
provide useful context around the incident.

------------------------------------------------------------------------

## 4.4 Debug logs

Debug logs contain diagnostic information used for troubleshooting.

They can be useful when normal logs do not provide enough information to
identify a problem.

**Important:** Do not automatically run a production Firebox at maximum
diagnostic/debug logging. WatchGuard recommends not setting the
diagnostic level to Debug unless directed by WatchGuard Technical
Support.

Official references:\
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/log_message_types_c.html\
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/logging_and_logfiles_about_c.html

------------------------------------------------------------------------

## 4.5 Statistic logs

Statistic logs contain performance-related information.

Examples include:

-   External interface performance
-   VPN bandwidth statistics
-   Other performance information

These can help identify capacity or performance problems.

------------------------------------------------------------------------

# 5. Log retention

## What is log retention?

**Log retention** is how long log information remains available for
review.

``` text
Event occurs
    ↓
Log generated
    ↓
Log stored
    ↓
Retention period
    ↓
Older data eventually removed
```

A Firebox can generate logs continuously, but it has limited internal
storage.

Therefore:

> **Logging enabled does not mean unlimited historical logging.**

------------------------------------------------------------------------

## 5.1 Why local retention can be short

Local log history depends on factors such as:

-   Firebox model
-   Fireware version
-   Log volume
-   Enabled logging features
-   Traffic volume
-   Diagnostic logging level
-   Available storage

The training slide may show example retention such as less than 20
minutes or even 1--3 minutes. Treat those values as **illustrative
examples**, not a universal fixed retention period for every Firebox.

The important concept is:

> **Higher log volume can cause older local logs to disappear sooner.**

``` text
Low log volume
[████████░░░░░░░░░░]
        ↓
More recent history

High log volume
[██████████████████]
        ↓
Older history replaced sooner
```

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/logging/logging_settings_configure_web.html

------------------------------------------------------------------------

# 6. External logging

External logging means sending Firebox logs to another system.

``` text
                 FIREBOX
                    |
              Log messages
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
   WatchGuard    Dimension    Syslog
      Cloud
```

Benefits:

-   Longer retention
-   Centralized visibility
-   Historical investigation
-   Reporting
-   Searching across devices
-   Integration with monitoring/SIEM platforms

WatchGuard recommends configuring at least one external log destination.

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/set_up_logging_on_device_wsm.html

------------------------------------------------------------------------

# 7. WatchGuard Cloud

WatchGuard Cloud is WatchGuard's cloud platform for management,
visibility, logging, and reporting.

When logging to WatchGuard Cloud is enabled, the Firebox can send logs
to the cloud in addition to other configured log servers.

Typical flow:

``` text
Firebox
   |
   | Log data
   v
WatchGuard Cloud
   |
   +--> Log Search
   +--> Dashboards
   +--> Reports
   +--> Retention
```

Current Cloud retention depends on the Firebox subscription and
applicable Data Retention licensing.

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/WG-Cloud/Devices/drl_data_deletion_about.html

------------------------------------------------------------------------

# 8. Dimension

**WatchGuard Dimension** is a WatchGuard virtual visibility and
management solution that can receive Firebox log data.

Typical architecture:

``` text
             FIREBOX
                |
                | log messages
                v
             DIMENSION
                |
        +-------+-------+
        |               |
    Log search       Reports
```

Dimension is different from WatchGuard Cloud because Dimension is
deployed in an environment you control.

  Platform                   Location
  -------------------------- ---------------------------
  Firebox internal storage   Firebox
  WatchGuard Cloud           WatchGuard cloud
  Dimension                  Your deployed environment
  Syslog                     Your logging/SIEM server

WatchGuard documentation also notes that log messages sent to Dimension
are encrypted.

------------------------------------------------------------------------

# 9. Third-party Syslog

A Syslog server collects log messages from devices and servers across a
network.

Example:

``` text
              +----------------+
              |    Firebox     |
              +----------------+
                       |
                       | Syslog
                       v
              +----------------+
              | Syslog / SIEM  |
              +----------------+
                       |
                       v
               Search / SOC
```

A Syslog platform can centralize logs from:

-   Firewalls
-   Routers
-   Switches
-   Servers
-   Applications
-   Other security appliances

Current WatchGuard documentation states that a Firebox can send logs to
up to **three Syslog servers**. The default Syslog port shown in the
documentation is **514**, although the port can be changed.

Supported formats include:

-   Syslog
-   IBM LEEF

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/logging/send_logs_to_syslog_c.html

### Syslog security warning

Standard Syslog log messages are **not encrypted**.

WatchGuard recommends avoiding sending Syslog through the Firebox
external interface and recommends placing the Syslog server on the
trusted network when possible.

------------------------------------------------------------------------

# 10. WatchGuard Cloud vs Dimension vs Syslog

  ------------------------------------------------------------------------------------------------------------
  Feature             Internal         WatchGuard Cloud                      Dimension                  Syslog
  ------------- -------------- ------------------------ ------------------------------ -----------------------
  Runs on                  Yes                       No                             No                      No
  Firebox                                                                              

  External             Limited                      Yes                            Yes                     Yes
  retention                                                                            

  Centralized               No                      Yes                            Yes                     Yes
  logs                                                                                 

  Long-term            Limited   Subscription-dependent   Deployment/storage-dependent   Server/SIEM-dependent
  history                                                                              

  WatchGuard           Limited                      Yes                            Yes     Depends on platform
  dashboards                                                                           

  Own server                No                       No                            Yes                 Usually
  required                                                                             

  SIEM                 Limited       Platform-dependent             Platform-dependent                     Yes
  integration                                                                          
  ------------------------------------------------------------------------------------------------------------

------------------------------------------------------------------------

# 11. Policy logging

Logging is not simply "on" or "off" for the whole Firebox.

Individual policies can have logging settings.

For example:

``` text
HTTPS Policy
    |
    +--> Allow traffic
    |
    +--> Send log messages
    |
    +--> Reporting logging (where applicable)
```

Use policy logging deliberately.

### Good approach

Log:

-   Security events
-   Important administrative events
-   Important denied traffic
-   Traffic needed for troubleshooting or reporting

Avoid unnecessarily logging huge amounts of normal allowed traffic when
it has no operational value.

------------------------------------------------------------------------

# 12. Local management lab

## Step 1 --- Open the Fireware Web UI

From a management/trusted network:

``` text
https://<Firebox-IP>:8080
```

Log in with a Device Administrator account.

## Step 2 --- Open logging

Go to:

**System → Logging**

Review:

-   Settings
-   Internal storage
-   Log server configuration
-   Syslog configuration
-   Diagnostic/performance settings

## Step 3 --- Enable internal logging for the lab

Enable:

**Send log message to Firebox Internal storage**

This makes recent activity available for local troubleshooting.

Remember:

> Internal storage = recent visibility, not long-term archive.

------------------------------------------------------------------------

# 13. Hands-on lab --- generate traffic logs

1.  Connect a test client behind the Firebox.
2.  Generate normal web/DNS traffic.
3.  Open Traffic Monitor/logging.
4.  Find the corresponding message.
5.  Identify:
    -   Source IP
    -   Destination
    -   Protocol
    -   Port
    -   Policy
    -   Action
    -   Timestamp

Example:

``` text
Source:      192.168.1.20
Destination: 8.8.8.8
Protocol:    UDP
Port:        53
Policy:      DNS
Action:      Allow
```

The exact fields vary by message and Fireware version.

------------------------------------------------------------------------

# 14. Hands-on lab --- test external logging

If you have a Syslog/SIEM server:

``` text
Firebox
   |
   | logs
   v
Syslog/SIEM
   |
   v
Search / Dashboard
```

After configuration:

1.  Generate traffic.
2.  Wait for the log to arrive.
3.  Search for the Firebox.
4.  Confirm timestamp.
5.  Confirm source/destination.
6.  Confirm policy/action.
7.  Generate another test event.
8.  Confirm the second event arrives.

------------------------------------------------------------------------

# 15. Troubleshooting missing logs

Use this order:

### 1. Is logging enabled?

Check the Firebox logging settings.

### 2. Is the policy configured to log?

Traffic can pass without producing the particular log you expect.

### 3. Is traffic reaching the Firebox?

Test basic connectivity.

### 4. Is internal storage enabled?

If you expect a local copy, verify this setting.

### 5. Can the external log server be reached?

Check:

-   IP address
-   Routing
-   Firewall policy
-   Port
-   Server status

### 6. Is the format correct?

For Syslog/LEEF, make sure the receiver expects the selected format.

### 7. Is logging volume excessive?

High-volume logging can increase Firebox load and consume storage
quickly.

------------------------------------------------------------------------

# 16. Logging and performance

Logging consumes resources.

``` text
More logging
     ↓
More messages
     ↓
More processing
     ↓
More storage consumption
     ↓
Potentially shorter retention
```

This does **not** mean logging should be disabled.

It means logging should be designed around the information you actually
need.

WatchGuard notes that logging can affect Firebox performance and
recommends reviewing logging settings if performance decreases.

Official reference:\
https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/logging_and_logfiles_about_c.html

------------------------------------------------------------------------

# 17. Recommended lab design

For a training Firebox:

``` text
             FIREBOX
                |
        +-------+-------+
        |               |
        v               v
 Internal logs     External logs
   (recent)        (if available)
        |               |
        v               v
 Quick testing     Historical review
```

Practice:

-   Reading Traffic Monitor
-   Identifying policy matches
-   Reading alarm messages
-   Reading event messages
-   Testing Syslog
-   Comparing local vs external retention
-   Troubleshooting missing logs

Avoid maximum Debug logging unless it is required for troubleshooting.

------------------------------------------------------------------------

# 18. Production mental model

Think in layers:

### Layer 1 --- Local visibility

Firebox internal storage.

**Purpose:** immediate troubleshooting.

### Layer 2 --- Centralized retention

WatchGuard Cloud or Dimension.

**Purpose:** historical visibility, dashboards, and reporting.

### Layer 3 --- Security operations

Syslog/SIEM.

**Purpose:** correlation with other infrastructure and security events.

``` text
Firewall
   |
   +----> Local recent logs
   |
   +----> WatchGuard Cloud / Dimension
   |
   +----> Syslog / SIEM
```

------------------------------------------------------------------------

# 19. Key terminology

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  Log                                 Recorded event/message

  Log retention                       How long data remains available

  Traffic                             Traffic-processing activity

  Alarm                               Alarm/security condition

  Event                               User/system/admin activity

  Debug                               Diagnostic information

  Statistic                           Performance information

  Internal storage                    Storage on the Firebox

  External logging                    Sending logs to another system

  Syslog                              Common network logging method

  Dimension                           WatchGuard virtual
                                      visibility/logging platform

  WatchGuard Cloud                    WatchGuard cloud
                                      visibility/management/logging
                                      platform

  SIEM                                Platform for centralized
                                      security-event collection and
                                      correlation
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 20. Interview questions

### Q1. Why not rely only on Firebox internal storage?

Because internal storage is limited and is intended mainly for recent
visibility/troubleshooting. External logging is appropriate when
historical retention matters.

### Q2. What are the major Firebox log types?

Traffic, Alarm, Event, Debug, and Statistic.

### Q3. What is log retention?

The period for which log data remains available for review.

### Q4. Why can high log volume shorten local history?

Because limited storage fills faster, causing older data to be replaced
sooner.

### Q5. What external logging options are common?

WatchGuard Cloud, Dimension, and third-party Syslog.

### Q6. Is standard Syslog encrypted?

No. WatchGuard recommends securing the logging path and avoiding sending
Syslog through the external interface when possible.

### Q7. How many Syslog servers can a locally-managed Firebox send logs to?

Current WatchGuard documentation states up to **three** Syslog servers.

### Q8. Should Debug logging always be enabled?

No. Use diagnostic/debug logging carefully and follow WatchGuard Support
guidance when deeper diagnostics are required.

------------------------------------------------------------------------

# 21. Quick revision

``` text
FIREBOX LOGGING
│
├── Traffic
│   └── Network traffic activity
│
├── Alarm
│   └── Alarm/security conditions
│
├── Event
│   └── System/user/admin activity
│
├── Debug
│   └── Troubleshooting information
│
└── Statistic
    └── Performance information
```

### Storage

``` text
Internal Firebox Storage
        ↓
Recent / limited history

External Logging
        ↓
Longer retention + centralized visibility
```

### External options

``` text
WatchGuard Cloud
Dimension
Third-party Syslog
```

------------------------------------------------------------------------

# 22. Key takeaways

1.  **Firebox logging is essential for visibility and troubleshooting.**
2.  The major log types are **Traffic, Alarm, Event, Debug, and
    Statistic**.
3.  **Internal Firebox storage is limited.**
4.  Higher log volume can make local history shorter.
5.  Do not treat a slide's example number of minutes as a universal
    Firebox retention value.
6.  Use **external logging** when historical records matter.
7.  Common external options are **WatchGuard Cloud, Dimension, and
    Syslog**.
8.  Standard Syslog messages are **not encrypted**.
9.  Excessive logging can increase load and consume storage.
10. Debug logging should be used carefully.
11. A good design separates **recent local visibility** from **long-term
    centralized retention**.

------------------------------------------------------------------------

# 23. Uploaded lesson images

Use the supplied slides at these positions in the lesson:

-   **Overview slide:** after Section 3 --- shows Firebox log settings,
    internal storage, and external logging options.
-   **Log types slide:** after Section 4 --- shows the types/categories
    of logs.
-   **Log retention slide:** after Section 5 --- illustrates limited
    internal storage and the effect of log volume.
-   **Storage options slide:** after Section 6 --- shows WatchGuard
    Cloud, Dimension, and third-party Syslog.
-   **Key Takeaways slide:** after Section 22.

The uploaded Key Takeaways image is packaged with the companion ZIP.

------------------------------------------------------------------------

# 24. Official WatchGuard references

-   Types of Log Messages\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/log_message_types_c.html

-   Define Where the Firebox Sends Log Messages\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/set_up_logging_on_device_wsm.html

-   Configure Logging Settings and Performance Statistics\
    https://www.watchguard.com/help/docs/help-center/en-US/content/en-us/Fireware/logging/logging_settings_configure_web.html

-   About Firebox Logging and Notification\
    https://www.watchguard.com/help/docs/help-center/en-US/Content/en-US/Fireware/logging/logging_and_logfiles_about_c.html

-   Configure Syslog Server Settings\
    https://www.watchguard.com/help/docs/help-center/en-us/Content/en-US/Fireware/logging/send_logs_to_syslog_c.html

-   Set Logging and Notification Preferences\
    https://www.watchguard.com/help/docs/help-center/en-US/content/en-US/Fireware/logging/set_logging_notif_pref_pm_c.html

-   About Data Retention and Data Deletion\
    https://www.watchguard.com/help/docs/help-center/en-us/content/en-us/WG-Cloud/Devices/drl_data_deletion_about.html

------------------------------------------------------------------------

## Final mental model

> **The Firebox generates logs. Internal storage gives you recent
> visibility. External logging gives you history.**

And:

> **Good logging is not "log everything." Good logging is collecting the
> information you need, at the right level, and retaining it where you
> can actually use it.**
