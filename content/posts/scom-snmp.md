---
title: "SCOM and SNMP"
date: 2020-04-29T10:50:52+08:00
lastmod: 2020-04-29T10:51:12+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "scom",
    "snmp",
]

comment: true
toc: true
auto_collapse_toc: true
---

## References

1. SCOM 2019 installation: https://www.prajwaldesai.com/scom-1801-install-guide/

## Integration with Unity SNMP Agent

### Steps
1. Install snmp agent on a Windows 10 machine: 10.245.54.154. The snmp agent is configured to manage Unity: 10.245.101.39.
2. The SCOM is deployed on a Windows Server 2019 machine: 10.245.54.155.
3. Configure SCOM to discover snmp device.
   
    3.1 Create a `Discovery Rule`. 
![](/forgetful/images/scom-discover-rules.png)

    3.2 After a rule is created, a new network device could be discovered.
![](/forgetful/images/scom-discover-network-devices.png)

4. Receive snmp trap.

    4.1 Create a event collection rule.
![](/forgetful/images/scom-event-collect-rule-0.png)
![](/forgetful/images/scom-event-collect-rule-1.png)
![](/forgetful/images/scom-event-collect-rule-2.png)
    
    4.2 Create a event view.
![](/forgetful/images/scom-event-view-0.png)
![](/forgetful/images/scom-event-view-1.png)
![](/forgetful/images/scom-event-view-2.png)

5. Try to send a trap from Unity. **SCOM didn't handle it although it received the package from Unity.**
![](/forgetful/images/scom-unity-send-snmp-trap.png)

    Wireshark on SCOM server.
![](/forgetful/images/scom-wireshark-server-receive-package-from-unity.png)

6. Try to send a trp from snmp agent. **SCOM handled it successfully.**

    6.1 Install `net-snmp` binary on Windows 10 machine `10.245.54.154`. Default installation path is `C:\usr\bin`.

    6.2 Run command to send trap.
    ```powershell
    PS C:\usr\bin> .\snmptrap.exe -v 2c -c public 10.245.54.155 2404 1.3.6.1.4.1.1139.103.1.18.2.6 1.3.6.1.4.1.1139.103.1.18.1.1 s "spa"
    PS C:\usr\bin>
    ```
![](/forgetful/images/scom-received-trap-event-from-snmp-agent.png)

### Conclusion
1. SCOM probably handles the trap event by the source IP address and drops all the trap event it doesn't know.
2. It has nothing to do with the engine id. Because Unity sends traps with it engine id in, but SCOM didn't handle it. And snmp agent didn't send the engine id (`-e` option of `snmptrap` command), SCOM handled it.
3. We could run snmp-agent in a container on Unity. Then SCOM could snmpget to Unity and handle the trap from Unity perfectly.


### Misc
1. Code change in branch https://github.com/emc-openstack/snmp-agent/tree/scom-integration
2. `snmptrap` command reference:
    ```console
    # Here is an example (To simplify, no auth protocol and encryption is used here.):
    sudo snmptrap --clientAddr=<mgmt_ip> -v 3 -l noAuthNoPriv -u <user name> -e 0x800004730508001BFF771C <host>:<port> \
        2404 \
        1.3.6.1.4.1.1139.103.1.18.2.6 \
        1.3.6.1.4.1.1139.103.1.18.1.1 s "spa" \
        1.3.6.1.4.1.1139.103.1.18.1.2 s "AlertServicesSnmpTest" \
        1.3.6.1.4.1.1139.103.1.18.1.3 s "14:100001" \
        1.3.6.1.4.1.1139.103.1.18.1.4 s "This is a test message to be sent in an SNMP trap."

    parameters:
        -v <snmp version>: 1/2c/3
        -a <auth protocol>: MD5/SHA
        -A <password>: if specified -a
        -x <encrypt protocol>: AES/DES
        -X <password>: if specified -x
        -l <security level>: noAuthNoPriv|authNoPriv|authPriv
        -u <user name>: The same as what configured on GUI.
        -e <engineID>: 0x800004730508001BFF771C (two parts: engineID (0x8000047305) + MACAddress of mgmt port)
        <host>:<port>: Host and port of the specified SNMP trap destination.
        -n <context name>: always empty, not specified.
        <uptime>: Using command “papi_get.sh uptime” to get the current up time.
        <trapOID>: TrapOID, actually is the severity of the alert, refer to above definitions “Severity”. Now almost all our alerts use the following severities: critical, error, warning, notice, info.
        <payload>: The data of the SNMP trap. Contains following fields: Node, Component(Alert Component), SymptomID(Alert Message ID), SymptomText(Alert Message).
            Each field contains of 3 parts: oid, type and data.
            OID is the defined “Data Types” above, type is always ‘s’ means string, data is specified to every fields.
    ```
    The payload in the example:

        Node: 1.3.6.1.4.1.1139.103.1.18.1.1 s "spa"
        Component: 1.3.6.1.4.1.1139.103.1.18.1.2 s "AlertServicesSnmpTest"
        SymptomID: 1.3.6.1.4.1.1139.103.1.18.1.3 s "14:100001"
        SymptomText: 1.3.6.1.4.1.1139.103.1.18.1.4 s "This is a test message to be sent in an SNMP trap."

    Date types references:

        Node = 1.3.6.1.4.1.1139.103.1.18.1.1
        Component = 1.3.6.1.4.1.1139.103.1.18.1.2
        SymptomID = 1.3.6.1.4.1.1139.103.1.18.1.3
        SymptomText = 1.3.6.1.4.1.1139.103.1.18.1.4
        TimeStamp = 1.3.6.1.4.1.1139.103.1.18.1.5

    Severity references:

        emergency = 1.3.6.1.4.1.1139.103.1.18.2.0
        alert     = 1.3.6.1.4.1.1139.103.1.18.2.1
        critical  = 1.3.6.1.4.1.1139.103.1.18.2.2
        error     = 1.3.6.1.4.1.1139.103.1.18.2.3
        warning   = 1.3.6.1.4.1.1139.103.1.18.2.4
        notice    = 1.3.6.1.4.1.1139.103.1.18.2.5
        info      = 1.3.6.1.4.1.1139.103.1.18.2.6
        debug     = 1.3.6.1.4.1.1139.103.1.18.2.7



### Troubleshooting
#### 1. Now snmp agent receives the request from SCOM, but it don't know these OIDs.

**It was resolved by adding OIDs: `1.3.6.1.2.1.1.1.0` to `snmp-agent`. Refer to the change of `scom-integration` branch.**

```console
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    Execution point: rfc3412.receiveMessage:request
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportDomain: 1.3.6.1.6.1.1.0
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportAddress: 10.245.54.155@55486 (local 0.0.0.0@11161)
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityModel: 2
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityName: public
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityLevel: 1
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * PDU: GetRequestPDU:
 request-id=16140311
 error-status=noError
 error-index=0
 variable-bindings=VarBindList:
  VarBind:
   name=1.3.6.1.2.1.1.1.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.2.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.4.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.5.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.6.0
   =_BindValue:
    unSpecified=


2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    Execution point: rfc3412.returnResponsePdu
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportDomain: 1.3.6.1.6.1.1.0
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportAddress: 10.245.54.155@55486 (local 0.0.0.0@11161)
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityModel: 2
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityName: public
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityLevel: 1
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * PDU: ResponsePDU:
 request-id=16140311
 error-status=noAccess
 error-index=1
 variable-bindings=VarBindList:
  VarBind:
   name=1.3.6.1.2.1.1.1.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.2.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.4.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.5.0
   =_BindValue:
    unSpecified=

  VarBind:
   name=1.3.6.1.2.1.1.6.0
   =_BindValue:
    unSpecified=


2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    Execution point: rfc3412.receiveMessage:request
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportDomain: 1.3.6.1.6.1.1.0
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportAddress: 10.245.54.155@55488 (local 0.0.0.0@11161)
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityModel: 1
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityName: public
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityLevel: 1
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * PDU: GetRequestPDU:
 request-id=16140312
 error-status=noError
 error-index=0
 variable-bindings=VarBindList:
  VarBind:
   name=1.3.6.1.2.1.1.1.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.2.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.4.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.5.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.6.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    Execution point: rfc3412.returnResponsePdu
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportDomain: 1.3.6.1.6.1.1.0
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * transportAddress: 10.245.54.155@55488 (local 0.0.0.0@11161)
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityModel: 1
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityName: public
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * securityLevel: 1
2020-05-07 09:49:01,521 snmpagent_unity.agent Thread-1 DEBUG    * PDU: GetResponsePDU:
 request-id=16140312
 error-status=noSuchName
 error-index=1
 variable-bindings=VarBindList:
  VarBind:
   name=1.3.6.1.2.1.1.1.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.2.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.4.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.5.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=


  VarBind:
   name=1.3.6.1.2.1.1.6.0
   value=ObjectSyntax:
    simple=SimpleSyntax:
     empty=
```

#### 2. SCOM could fail to discover the network device with error `No Response Ping`.

This happens mostly after the SCOM server reboots.

**Solution: Go to firewall. Enable all rules, including Inbound and Outbound ones, related to `Operation Manager`.**