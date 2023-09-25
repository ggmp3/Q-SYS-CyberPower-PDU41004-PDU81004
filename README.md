# CyberPower PDU41004 and PDU81004 Networked PDU (SNMP Control)
-- Written by Glen Gorton

## NOTES:
-- Originally built with PDU41004 Firmware v1.2.0 (to upgrade firmware, use the CyberPower Upgrade and Configuration Utility, or USB)
-- SNMPv3 Controls & Queries: you will need to browse to the HTTP interface of the PDU, select System->Network Service->SNMPv3. Allow Access and set up Access Control accounts/names.
-- Code below allows you to configure the SNMPv3 authentication/connection on the UI. (eg. no authentication, no privacy protocol). Make sure these connection details match the credentials on the PDU.
-- The PDU41004/PDU81004 will not allow multiple connections, so a Q-Sys telnet connection will fail if the HTTP interface is being accessed and vice versa. SNMP DOES work alongside telnet/HTTP.
-- Commands that send strings (eg. Name, Location, Contact) will not accept 'spaces' between words/characters.
-- ENVIROSENSOR Temperature and Humidity cannot be obtained via telnet commands in firmware v1.2.0. (Requires SNMP v1 or v3 queries/traps)

## 26th September 2023
-- Some changes made to the snmp_session.ErrorHandler as QSC have advised there are six possible errors that can be returned: "Timed out", "Send Failed", "Sec Error", "SNMP Error", "No Such Instance" and "No Such Object"
-- Created 'compromised' Event Logging for the errors: "Send Failed", "Sec Error", "No Such Instance" and "No Such Object"
-- CyberPower have also advised that the sensor on the outlets that provides the active Load (amps) and Power (watts) is only accurate if the load is above 1A/100Watts. This may have an effect in the OutletDrawingPowerStatus() function.

## 13th September 2023
-- QSC have fixed (with Designer v9.9.0) the snmp_session.ErrorHandler which will now provide a response if there is a NULL response. (ie. unsupported OID sent to device).
-- Designer v9.4.4 to v9.8.2 will crash (when emulating) or a running Core Design will eventually restart because it does not support a NULL response. Versions prior to v9.4.4 worked!
-- Found that script appeared to become unstable on QSC Firmware v9.4.4 and up, possibly also because of too many SNMP sessions running concurrently. snmp_session:startSession() now only runs once at Connect(), not for every OID sent.
-- StartSNMPSession() function added to start single the snmp_session at Connect().
-- Many other changes to how the ActiveQueryList OID table is used. Entries are not longer removed then added back into the table. The items number being polled now increments so the table is never modified.
-- sendDelaySet() function and associated Timers all removed.
-- Added Controls["Restart Script"]:Trigger() when IP Address is changed.
-- Removed Helper Function debugFormat() as it was not used.
-- Added Connect() function to SNMPnoresponse() function so that the device continues to try and re-establish a connection.
-- snmp_session.ErrorHandler will print errors and  SetStatus() depending on the error received (eg. "Timed out" will SetStatus to Compromised after a number of TimeOuts)

## 25th July 2023
-- Created new QID Query List (ModelQuery) which is a single OID query to obtain PDU model number. This OID is now sent first.
-- The first funciton within the requestSNMP() function will now check the PDU model number (Controls["Model"].String), then use the QueryList table specific to that model#. Eg. QueryListPDU81004
-- QueryList tables created (QueryListPDU41004 and QueryListPDU81004).
-- ActiveQueryList = {} script variable added to store which Query List table is being used. Clearing of the ActiveQueryList added to Initialize() function.

## 11th July 2023
-- Tested with PDU41004 and PDU81004 Firmware v1.2.6 which includes update MIB Library v2.10.
-- Added some additional OID's to the commented out "ePDU2 -- Do not work, or unnecessary" section for reference.
-- Added 'IntegerIndicators' table that contains integer controls to be reset when the script is Initialized.
-- Added function ConvertDateTime() to format date and time to: dd-mm-yyyy hh:mm:ss
-- Added function ConvertDate() to format date to: dd/mm/yy
-- Added 'Outlet Peak Load' (watts) text indicators.
-- Added 'Outlet Peak Load Date' text indicators.
-- Added 'Outlet Energy Consumption' (kWh) text indicators.
-- Added 'Outlet Energy Meter Date' text indicators.
-- Added 'Outlet Overload Threshold' (watts) integer inputs.
-- Added 'Outlet Near Overload Threshold' (watts) integer inputs.
-- Added 'Outlet Low Load Threshold' (watts) integer inputs.
-- Added 'Location' text box.
-- Added 'Contact' text box.
-- Added 'Overload Threshold' (amps) integer inputs.
-- Added 'Near Overload Threshold' (amps) integer inputs.
-- Added 'Low Load Threshold' (amps) integer inputs.
-- Added 'Device Power Limit' (watts) text indicator.
-- 'Cold Start State' is now a combo box with selectable controls. Was previously just a text indicator.
-- 'Cold Start Delay' is now a combo box with selectable controls. Was previously just a text indicator.
-- Added 'Cold Start Delay Time' (sec) integer input that works in conjunction with 'Cold Start Delay' control.
-- 'System Name', 'Device Rating', 'Number of Outlets', 'Switched Outlets', 'Metered Outlets', 'Firmware Version', 'Date Of Manufacture', 'Hardware Version', 'Model Number',  'Serial Number', 'Cold Start Delay', now using ePDU2 OID.

## 7th December 2022
-- PDU41004 model# is not capable of reading Watts. Added more "PowerWatts" string checks to avoid "attempt to compare number with nil" error within the 'OutletDrawingPowerStatus()' function.

## 2nd Decemeber 2022
-- 'Restart Script' Control (Trigger) added to script. Found SNMP requests would not work if the 'Connect' boolean was pressed, Initializing the script. As script restart is required.
-- 'Script Start' input pin exposed on the text controller.
-- Updated Controls["Connect"].EventHandler to include a Restart Script trigger if the Connect Boolean == false.

## 25th November 2022
-- EventLog() function has been updated as the faster ipairs interating through the EventLogs table was not reliable when wired to a Monitoring Proxy.

## 14th November 2022
-- Added "Log Entry" and "Log Severity" to TextIndicators to the reset when script Initializes.

## 1st December 2021
-- Added 'Outlet And Power Event Logging' toggle button per outlet so event logs can be turned on per outlet.

## 11th November 2021
-- Added 'Outlet Load Amps' (indicator added per outlet).
-- Added 'Outlet And Power Event Logging' toggle button that must be True to enable below added functions:
-- Added 'OutletDrawingPowerStatus()' function which keeps track of outlet power state and power draw, logs events for off/on transitions and unexpected power loss.
-- Event Logs are now pushed to the EventLogs table in the output 'trigger' to Monitoring Proxy cannot handle multiple triggers at once. EventLogs table is interated through using EventLog().
-- Any future event logs need to be added to the table. Eg. table.insert(EventLogs, {log = "Outlet "..i.." with connected device ("..ControlName..") has been turned OFF." , severity = "warning"})
-- Found that the PDU cannot read a 10w load (eg. iPad charger - reads as 0 wattts). CyberPower have advised that "Due to the sensitivity of the sensor, the displayed load level will not be reliable unless it's higher than 10%~15% of the rated capacity of PDU. Otherwise both displayed load level and runtime will NOT be referable."
-- Controls["PowerWatts Off Threshold"].String (eg. 1) and Controls["PowerWatts On Threshold"].String (eg. 10) need to be populated. Based on the response from CyberPower the On Threshold may need to be 15 watts or above.

## 2nd July 2021
-- Tested on PDU41004 and PDU81004 Firmware v1.2.1.
-- Added additional controls for PDU81004 which is a metered-by-outlet PDU ('Power Watts' indicator added per outlet).
-- Added 'Number Of Outlets', 'Switched Outlets', Metered Outlets', 'Date Of Manufacture', 'On Delay', 'Off Delay', 'Reboot Duration' commands.
-- Added Environment Sensor group box containing: Humidity, Temperature, Name, Location.
-- Added Monitoring Proxy, SetStatus, EventLog and ResetTextIndicators functions.
-- Added 'Connect' toggle, 'Connected' LED indicator, and 'Delay Between Commands' text input box.