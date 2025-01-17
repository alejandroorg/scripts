# Scripts

## Elasticsearch Indices Reindex Tool: reindex.sh
**USAGE:** `reindex.sh (-a|--all yes | -f|--file /path/file) -s|--server https://elastic_address:port -u|--user user -p|--pass password`
**Note:** The script is not yet complete, I have to develop the `ALL` feature

## Archived Logs Removal Tool: alrt.py

## Windows Event Log converter
It converts windows Event logs from the EventViewer or from a file containing the exported Event (see picture) to a one-line JSON string to test it in `ossec-logtest` (or `wazuh-logtest`).

![image](https://user-images.githubusercontent.com/37050249/129812061-7bc1e2ed-b081-441b-8260-78a0f4bd789f.png)

**USAGE:**
```
F:\Wazuh\Scripts\Event-Converter.ps1 -LogName Security -EventID 4690
F:\Wazuh\Scripts\Event-Converter.ps1 -FilePath ./event.evt
```

**EXAMPLE:**
```
{"win":{"system":{"level":"0","systemTime":"2021-08-17T22:00:43.6300632Z","opcode":"0","providerGuid":"{54849625-5478-4994-a5ba-3e3b0328c30d}","security":"","computer":"DESKTOP-Q1TV95Q","eventID":"4690","channel":"Security","task":"12807","version":"0","correlation":"","severityValue":"Information","providerName":"Microsoft-Windows-Security-Auditing","keywords":"0x8020000000000000","eventRecordID":"426523486","message":"An attempt was made to duplicate a handle to an object.\\r\\n\\r\\nSubject:\\r\\n\\tSecurity ID:\\t\\tDESKTOP-Q1TV95Q\\\\Dario\\r\\n\\tAccount Name:\\t\\tDario\\r\\n\\tAccount Domain:\\t\\tDESKTOP-Q1TV95Q\\r\\n\\tLogon ID:\\t\\t0x122458\\r\\n\\r\\nSource Handle Information:\\r\\n\\tSource Handle ID:\\t0xf60\\r\\n\\tSource Process ID:\\t0x2e54\\r\\n\\r\\nNew Handle Information:\\r\\n\\tTarget Handle ID:\\t0x5a1c\\r\\n\\tTarget Process ID:\\t0x4\\r\\nEvent Xml:\\r\\n"},"eventData":{"subjectDomainName":"DESKTOP-Q1TV95Q","subjectLogonId":"0x122458","targetProcessId":"0x4","sourceHandleId":"0xf60","subjectUserName":"Dario","sourceProcessId":"0x2e54","subjectUserSid":"S-1-5-21-2470304919-1303594442-3450856006-1007","targetHandleId":"0x5a1c"}}}
```

**Configurations needed:**
Configure the Parent rule to link it to JSON decoder and not EventChannel
`/var/ossec/ruleset/rules/0575-win-base_rules.xml`
```
  <rule id="60000" level="2">
    <!-- category>ossec</category -->
    <!-- decoded_as>windows_eventchannel</decoded_as -->
    <decoded_as>json</decoded_as>
    <field name="win.system.providerName">\\.+</field>
    <options>no_full_log</options>
    <description>Group of windows rules</description>
  </rule>
```

## Wazuh Cluster Monitoring Tool: monitor-cluster.py
This tool can run in automatic or manual mode, in automatic mode, there is no need to pass arguments to it. In manual mode you need to specify only one argument.
Tool parameters:
```
# Usage:
#       monitor-cluster.py update|status
# Parameters:
#       update     Use this to create, or update the baseline file
#       status     Gets the current status of the cluster
# No parameters: The tool runs in automatic mode, if the baseline file is not crated
#                it creates it, if not it runs the status check.
```

## Wazuh Fields Manager Tool: fields-manager.py
This tool is intended to be used as a tool for inserting (update mode) new fields in the `wazuh-template.json` file. Getting the fields from an input YAML file, and converting it to a python dictionary and then dumping it to the `wazuh-template.json`. Also it is capable of get the differences (getdiff mode) between the local (modified) template, and the template allocated in the Wazuh repositories.
```
usage: fields-manager.py [-h] (-d | -u) [-f filename]

Manage Custom Fields for Wazuh Template

optional arguments:
  -h, --help            show this help message and exit
  -d, --getdiff         Obtain differences between Default Template and local template.
  -u, --update          Updates local template with new/modified fields.
  -f filename, --inputfile filename
                        Specify the custom fields file
```
**Note:** This script uses the module `deepdiff`, so it is needed to install the module before using it: `pip install deepdiff`

## Tool to dump all Wazuh Indices in Elasticsearch to JSON: elastic-dump.sh

## Groups Inventory tool: groups-inventory.py
This script that takes the information about inventory from the Wazuh Manager’s databases through API calls and it inserts the information as alerts into Wazuh (and then Elasticsearch).
It must run locally on the Wazuh Manager and you can execute it manually or automatically with a wodle command.
```
<wodle name="command">
  <tag>groups-inventory</tag>
  <disabled>no</disabled>
  <command>/path/to/script/groups-inventory.py --group linux winsrv winwks --endpoint packages os</command>
  <interval>24h</interval>
  <ignore_output>yes</ignore_output>
  <run_on_start>yes</run_on_start>
  <timeout>0</timeout>
</wodle>
```
Remember to give execution permissions to the script: `chmod +x groups-inventory.py`

You need to pass two arguments to the script, `group` and `endpoint`. They can have one or more values:
```
# ./groups-inventory.py -h
usage: groups-inventory.py [-h] [--group GROUP [GROUP ...]] [--endpoint ENDPOINT [ENDPOINT ...]]

Get inventory information from agents and inject it in Wazuh as an alert.

optional arguments:
  -h, --help            show this help message and exit
  --group GROUP [GROUP ...]
                        Group name to query, use ALL for all the groups
  --endpoint ENDPOINT [ENDPOINT ...]
                        Specific syscollector endpoint to query. Use ALL for all the endpoints
```

**Examples:**
1. Getting Packages and OS information from groups linux, winsrv and winwks.
```
./groups-inventory.py --group linux winsrv winwks --endpoint packages os
```
2. Getting all the inventory information from all the groups:
```
./groups-inventory.py --group ALL --endpoint ALL
```
Note: This script does not compare if changes were made on the inventory, it will bring all the inventory information requested, all the times it is executed. This means that it will duplicate data in the Elasticsearch, that’s why it is recommended to not run it frequently. Weekly it would be a good choice, but you can run it also daily.

Once it runs, it inserts the data directly to the Wazuh Manager through a linux socket, and you should start seeing the information on your Kibana server UI:
![image](https://user-images.githubusercontent.com/37050249/147351670-61e7096c-0741-407d-a845-0d5806718e5f.png)
As you can see, you can get all the inventory information and filter it with groups.
