# Sending markers with E-Prime to LPT port

The sample files can be downloaded [here](https://downgit.github.io/#/home?url=https://github.com/solo-fsw/eprime-lpt-sample-tasks) (download starts immediately with DownGit). The sample files include a sample in which markers are sent through Task Events, this is not explained below.

Markers can be sent from E-Prime to the LPT port in InLines. There are two general ways of sending markers. One way is to use the WritePort command:

```VBScript
WritePort <PortAddress>, <MarkerValue>
```

When using this command, the marker value is being sent, until a 0 is sent to the port address:

```VBScript
WritePort <PortAddress>, 0
```

Thus, to send a marker with a value of 10 to port address DFF8 for 100 ms, the following code can be used:

```VBScript
WritePort "&HDFF8", 10
sleep 100
WritePort "&HDFF8", 0
```

A different way of sending markers, is to link the sending of the marker to the onset or offset of an object. For this, the following code should be placed in an InLine that is located before the onset of the relevant object.

```VBScript
'Enable onset signal:
Fixation.OnsetSignalEnabled = True

'Set the parallel port address:
Fixation.OnsetSignalPort = "&HDFF8"

'Set the marker value that is sent at object onset:
Fixation.OnsetSignalData = 30

'Enable offset signal:
Fixation.OffsetSignalEnabled = True

'Set the parallel port address:
Fixation.OffsetSignalPort = "&HDFF8"

'Set the marker value that is sent at object offset:
Fixation.OffsetSignalData = 0
```

With this code, a marker with a value of 30 is sent at the onset of an object called **Fixation **and the value 0 is sent at the offset of the Fixation object. **When using this method of sending markers, make sure that the PreRelease of the object (in this case the Fixation object) is set to 0.** The **PreRelease** can be set in the **Durations/Input** tab in the **Properties** of the object.

In the above examples the port address was repeated for each command. Instead of having to write out the same port address each time a marker is sent, **it is highly advised to define the port address as a constant in the User Script**. Constants are variables that cannot change value during script execution. Thus, by defining the port as constant, you can be sure that it will not be adjusted anywhere else in the task.

To access the User Script in E-Prime 3, press **Alt + 7** or double-click on the **User Script** icon in the top of the **Experiment Explorer**. To access the User Script in E-Prime 2, press **Alt + 5**. In the User Script, define the LPT port as string:

```VBScript
Const LPTAddress As String = "&HDFF8" 'Address LPT Port
```

**In the labs of FSW Leiden the LPT port address of the stimulus PCs can be found on a sticker on the PC.**

Note that in E-Prime the characters “&H” need to be placed in front of the address, because the address is given as a hexadecimal.

When sending markers, it is possible to refer to an attribute for the marker value. For example, when the value 21 needs to be sent for congruent stimuli and 22 for incongruent stimuli, these values can be specified in a list and when sending markers, one can refer to the relevant column:

![E-Prime_list_markers](https://user-images.githubusercontent.com/56065641/217825442-08bd04c4-c4ca-47cd-bdad-c9a1445851e7.png)

```VBScript
Stimulus.OnsetSignalEnabled = True
Stimulus.OnsetSignalPort = LPTAddress
Stimulus.OnsetSignalData = CInt(C.GetAttrib("StimulusMarker")
Stimulus.OffsetSignalEnabled = True
Stimulus.OffsetSignalPort = LPTAddress
Stimulus.OffsetSignalData = 0
```

In the above example, a marker with the value **21** when the Stimulus is **congruent* *and **22** when the Stimulus is **incongruent** is sent to the port address stored in the variable **LPTAddress** at the onset of the **Stimulus** object. The marker is sent until the offset of the object (sending value 0 signals the end of the previous marker). Note that the marker value is converted to integer using **CInt**.

**Make sure that the duration of the marker is at least a few samples.** For example when recording physiological signals with Biopac at a sampling rate of 500 Hz (500 samples per second), each sample has a duration of 2 ms. In this case, when sending a marker with a duration of 1 ms, it is possible that it will be missed by the Biopac. In this case, we would advise to send a marker with a duration of at least 10 ms. Always check that all markers are present in your data.

**Always include an inline at the start of your experiment in which you set the LPT port to 0.** It is possible that one of the bits of the LPT port is high when the computer is started. To make sure that at the start of your experiments all bits are low, it is important to ‘reset’ the LPT port by sending a 0. The following code is advised to be added to an inline at the start of your experiment. In this code first a 0 is sent, then a 255 (all bits high) to check whether all bits are working correctly, and then a 0 again.

```VBScript
'Reset port
WritePort LPTAddress, 0
sleep 100
WritePort LPTAddress, 255
sleep 100
WritePort LPTAddress, 0
```

**Make sure to not run your task in windowed mode.** Some issues have been encountered when sending markers with E-Prime in windowed mode (some markers would not arrive).
