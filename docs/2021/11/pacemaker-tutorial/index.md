# Pacemaker Tutorial


# Pacemaker Tutorial

## MISC CRM Commands

```console
$ # You can set a node to standby mode, 
$ # which can be used to simulate a node becoming unavailable,
$ # with this command:
$ sudo crm node standby NodeName
 
$ # You can change a node’s status from standby 
$ # to online with this command:

$ sudo crm node online NodeName
```
