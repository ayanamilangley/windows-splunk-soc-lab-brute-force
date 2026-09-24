# Brute-Force Authentication Investigation

-  1) Overview

This investigation was made in small home SOC lab using Windows 10 with Splunk Universal Forwarder and Debian with Splunk Enterprise.
The goal was to simulate repeated failed authentication attempts on a Windows endpoint, collect the related security events in Splunk, trigger an alert based on rule and investigate the activity using a standard SOC L1 triage process.
All activity in this investigation was generated for testing purposes and does not represent a real attack.

-  2) Lab Environment

The lab consisted of the following components:

* Windows 10 — source of Windows Security Events
* Splunk Universal Forwarder — log collection and forwarding
* Debian — host system for Splunk Enterprise
* Splunk Enterprise — SIEM platform
* Windows Security Event Logs — authentication telemetry
* Splunk Web — search, alerting, and investigation interface

The basic data flow is shown at:

`/screenshots/architecture.png`

-  3) Investigation Scenario

Several incorrect password attempts were made on the Windows 10 endpoint.
These attempts generated Windows Security Event ID 4625, which signs that logon attempt failed.
A Splunk detection rule was configured to trigger an alert when three failed logon events were observed. 
This was used to simulate a simple brute-force detection scenario and to practice transition from raw events to SOC alert.
Purpose of the test was to verify the complete process of collect authentication events, detecting repeated failures, generating an alert, and investigating the activity in the SIEM.

-  4) Detection

The initial Splunk search used to identify failed logons was:

```spl
index=* EventCode=4625
```

The search returned multiple failed logon events from the Windows endpoint.

The main fields reviewed during the investigation were:

* account name
* timestamp
* logon type
* failure reason

In addition to the search, a detection rule was configured with a rule of three failed logon attempts.

The rule generated an alert after the threshold was reached. This alert was then used as starting point for the investigation.

### Alert

The detection alert is shown in:

`/screenshots/splunk-brute-force-alert.png`

The alert indicated that configured threshold for failed authentication attempts had been reached.

-  5) Initial Findings

Few Event ID 4625 events were generated during the test.

The account name shown in the collected events was:

`DESKTOP-FHR4I96$`

The events occurred between approximately `14:00` and `15:30`.
The repeated 4625 events matched the authentication activity generated in the lab.

### Evidence

The original Windows Security Event can be seen here:

`/screenshots/windows-event-4625.png`

The corresponding events collected by Splunk are shown here:

`/screenshots/splunk-4625.png`

The alert generated after three failed logons is shown here:

`/screenshots/splunk-brute-force-alert.png`

-  6) Triage

The alert was treated as indication of potentially suspicious authentication activity rather than as automatic proof of a brute-force attack.

A failed authentication event can occur for legitimate reasons, such as:

* a user entering an incorrect password
* an outdated saved credential
* a misconfigured service
* repeated authentication attempts from an application

For this reason, the investigation focused on the event context and the activity that caused the alert.

The following questions were considered during triage:

1. Which account appeared in the failed logon events?
2. How many failed attempts were recorded?
3. When did the attempts occur?
4. What source information was available?
5. What logon type was involved?
6. Was there a successful logon after the failed attempts?
7. Did the activity match the expected behavior of the test environment?

-  7) Authentication Timeline

Successful logon events were also checked using Event ID 4624:

```spl
index=* EventCode=4624
```

The purpose was to determine is successful authentication occurred around the failed attempts?
The authentication events were reviewed as the timeline to get it what happened before and after the alert.

-  8) Assessment

The alert worked as expected: after three failed authentication events were detected, Splunk generated an alert for further investigation.
The observed activity consist simulated scenario because the failed logons were intentionally generated during the test.
However, alert does not prove that a real brute-force attack occurred, so in production environment, the alert would need additional context before being classified as malicious.

Relevant information for further investigation could include:

* source IP address and its reputation
* authentication history for the account
* frequency of the attempts
* successful authentication after repeated failures
* related activity on the endpoint
* additional endpoint or network telemetry

-  9) Conclusion

The main focus of the exercise was not only identifying Event ID 4625, but also understanding how a SOC analyst works with an alert generated from multiple events.
The lab showed how a simple threshold-based rule can turn repeated authentication failures into an alert that can then be reviewed.

-  10) Limitations

This was a small home lab with intentionally generated authentication activity.
The test did not include:

* a real external attacker
* multiple endpoints
* Active Directory
* enterprise-scale correlation
* EDR telemetry
* network-wide monitoring
* a production incident response process


Therefore, the project demonstrates the basic detection and investigation workflow rather than a complete enterprise brute-force detection solution.

