# Q-SYS-CyberPower-PDU41004-PDU81004 - SNMP
- CyberPower PDU41004 Networked PDU - SNMP
  - Written by Glen Gorton 26/08/2020

- Built with PDU41004 Firmware v1.2.0 (to upgrade firmware, use the CyberPower Upgrade and Configuration Utility, or USB)

- SNMPv3 Controls & Queries ******** you will need to browse to the HTTP interface of the PDU, select System->Network Service->SNMPv3. Allow Access and set up Access Control accounts/names.

- Code below allows you to configure the SNMPv3 authentication/connection on the UI. (eg. no authentication, no privacy protocol). Make sure these connection details match the credentials on the PDU.

- The PDU41004/PDU81004 will not allow multiple connections, so a Q-Sys telnet connection will fail if the HTTP interface is being accessed and vice versa. SNMP does work alongside telnet/HTTP.

- Commands that send strings (eg. Name, Location, Contact) will not accept 'spaces' between words/characters.
- ENVIROSENSOR Temperature and Humidity cannot be obtained via telnet commands in firmware v1.2.0. (Requires SNMP v1 or v3 queries/traps)



## Updated 01/12/2021
  - Added 'Outlet And Power Event Logging' toggle button per outlet so event logs can be turned on per outlet.

## Updated 11/11/2021

  - Added 'Outlet Load Amps' (indicator added per outlet).
  - Added 'Outlet And Power Event Logging' toggle button that must be True to enable below added functions:
  - Added 'OutletDrawingPowerStatus()' function which keeps track of outlet power state and power draw, logs events for off/on transitions and unexpected power loss.
  - Event Logs are now pushed to the EventLogs table in the output 'trigger' to Monitoring Proxy cannot handle multiple triggers at once. EventLogs table is interated through using EventLog().
  - Any future event logs need to be added to the table. Eg. table.insert(EventLogs, {log = "Outlet "..i.." with connected device ("..ControlName..") has been turned OFF." , severity = "warning"})
  - Found that the PDU cannot read a 10w load (eg. iPad charger - reads as 0 wattts). CyberPower have advised that "Due to the sensitivity of the sensor, the displayed load level will not be reliable unless it's higher than 10%~15% of the rated capacity of PDU. Otherwise both displayed load level and runtime will NOT be referable."
  - Controls["PowerWatts Off Threshold"].String (eg. 1) and Controls["PowerWatts On Threshold"].String (eg. 10) need to be populated. Based on the response from CyberPower the On Threshold may need to be 15 watts or above.

## Updated 02/07/2021

- Tested on PDU41004 and PDU81004 Firmware v1.2.1.

  - Added additional controls for PDU81004 which is a metered-by-outlet PDU ('Power Watts' indicator added per outlet).
  - Added 'Number Of Outlets', 'Switched Outlets', Metered Outlets', 'Date Of Manufacture', 'On Delay', 'Off Delay', 'Reboot Duration' commands.
  - Added Environment Sensor group box containing: Humidity, Temperature, Name, Location.
  - Added Monitoring Proxy, SetStatus, EventLog and ResetTextIndicators functions.
  - Added 'Connect' toggle, 'Connected' LED indicator, and 'Delay Between Commands' text input box.


