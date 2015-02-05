## XCP CCP Table Controls ##

**XCP CCP Table Controls** allows users to view and edit XCP and CCP tables while using the XCP and CCP Master Custom Device for NI VeriStand. This includes viewing the working point of the table while the ECU is running.

### LabVIEW Version ###

LabVIEW 2011

### Built Availability ###

This IP is built and available for NI VeriStand 2011 and later [here](https://decibel.ni.com/content/docs/DOC-19472) but will be taken down soon.

### Quality, Limitations ###

This IP should be considered mature, but of low quality. The code is kind of crazy and was the work of three different people. However, it works, and has worked in deployed systems for some time. Additionally, it wont be updated anymore as NI VeriStand moves to the UI manager.

1. These controls use an Xcontrol provided by Drivven that does all the table stuff. I will detail the issues with the xControls on a different #... but overall I recommend removing them entirely. We should integrate the functionality of the XControl directly into the workspace control VI. This would dramatically improve performance, ease implementation, and increase maintainability.

2. the xcontrols cause a lot of problems. Some fixable some not:
	- There is a significant load delay for these controls because of the xcontrol
	- Some XControl VIs are used in the Xcontrol and in the workspace control… causing lots of weird linking.
	- They each link to each other... causing lots of confusing linking. Ideally anything common between them would be in a different shared library.
	- They are single precision floats
	- The references to them are unique per xControl... so its impossible to make reusable subVIs for each of the table controls (1d, 2d). Instead you have make duplicate subVIs with different reference wire types
	- All the data that needs to be passed between the workspace control and the xcontrol has to go through property nodes that are very slow (lots of CPU)
	- Since the Xcontrol handles the right click menu… which handles the selection of XCP/CCP tables and working point mappings… the workspace control has to poll that mapping to see if it has changed and then save that data into the workspace control persistent data. This causes significant CPU load and is a weird programming flow.
	- Resizing the xcontrol when the workspace control is resized requires polling a property node (slow)

3.	These workspace controls don’t use a state machine. They just use an event structure. This makes them very hard to program efficiently. For example… the other controls have an init case where they check for connection status and if the connection status value changes or the workspace state changes (events), they go back to the init state and re-check the connection status. This means in their polling loops… they never need to check the connection status because that information is already known. These XCP/CCP table controls do not have that ability because they aren’t a state machine… so every single event needs to make lots of the same checks (for example, connection status). Ideally, these table controls would be refactored to use the same state machine flow as the other controls.

4.	The “view limits” feature of the xcontrol right click menu is there… but without “set limits” right click option it isn’t very useful. You can right click to view limits but you can’t right click to change the limits. The limits are hard coded to be +/- inf. Ideally we would:
	-	Set these limits from the meas/char max/min properties
	- Allow the user to narrow these limits if desired



5.	Error handling strategy needs work: The XControls set their title text coloring blue or black depending on if there was an error, and the user can right click “Display last error”. However this doesn’t work quite right and it isn’t in-line with how other controls work. They will show red if the connection is down… not if there is an error.


### License ###

*This repository and any materials provided by NI therein are provided AS IS. NI DISCLAIMS ANY AND ALL LIABILITIES FOR AND MAKES NO WARRANTIES, EITHER EXPRESS OR IMPLIED, INCLUDING WITHOUT LIMITATION ANY WARRANTIES OF MERCHANTABILITY, FITNESS FOR  PARTICULAR PURPOSE, OR NON-INFRINGEMENT OF INTELLECTUAL PROPERTY. NI shall have no liability for any direct, indirect, incidental, punitive, special, or consequential damages for your use of the repository or any materials contained therein.*