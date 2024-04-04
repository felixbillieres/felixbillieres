# 🕰️ Sigma

[Sigma](https://github.com/SigmaHQ/sigma) is an open-source generic signature language developed by Florian Roth & Thomas Patzke to describe log events in a structured format. This allows for quick sharing of detection methods by security analysts.

<figure><img src="../../../.gitbook/assets/image (790).png" alt=""><figcaption></figcaption></figure>

Sigma rules are written in YAML Ain't Markup Language ([YAML](http://yaml.org/)), a data serialisation language that is human-readable and useful for managing data.

### Sigma Syntax

<figure><img src="../../../.gitbook/assets/image (791).png" alt=""><figcaption></figcaption></figure>

```shell-session
title: WMI Event Subscription
id: 0f06a3a5-6a09-413f-8743-e6cf35561297
status: test
description: Detects creation of WMI event subscription persistence method.
logsource:
   product: windows    
   category: wmi_event 
detection:
  selection:
    EventID:  # This shows the search identifier value
      - 19    # This shows the search's list value
      - 20
      - 21
  condition: selection
falsepositives:
    - Exclude legitimate (vetted) use of WMI event subscription in your network
level: medium
tags:
  - attack.persistence # Points to the MITRE tactic.
  - attack.t1546.003   # Points to the MITRE technique.      
```

* **Status:** Describes the stage in which the rule maturity is at while in use. There are five declared statuses that you can use:
  * _Stable_: The rule may be used in production environments and dashboards.
  * _Test_: Trials are being done to the rule and could require fine-tuning.
  * _Experimental_: The rule is very generic and is being tested. It could lead to false results, be noisy, and identify interesting events.
  * _Deprecated_: The rule has been replaced and would no longer yield accurate results. The`related` field is used to create associations between the current rule and one that has been deprecated.
  * _Unsupported_: The rule is not usable in its current state (unique correlation log, homemade fields).
