# Using Intecre.Utilities

How to use each part of the library, with short examples. For more, read the TcUnit suites in the `TestRunner` project: each suite under `TestRunner/Tests` shows one block being set up, called and checked, and they all pass against this version.

Add the library to a PLC project as the placeholder `Intecre.Utilities`. Its names resolve without a prefix; if one clashes with a name in your project, qualify it with the namespace, for example `Intecre_Utilities.T_Timer`.

- [Before you start: defaults and fixed limits](#before-you-start-defaults-and-fixed-limits)
- [Logging](#logging)
- [Named objects](#named-objects)
- [Lists](#lists)
- [Dictionary](#dictionary)
- [Strings](#strings)
- [Parsing text into numbers](#parsing-text-into-numbers)
- [Values held as T_Arg](#values-held-as-t_arg)
- [Task information](#task-information)
- [Timer](#timer)
- [ADS](#ads)
- [Controllers and filter](#controllers-and-filter)
- [Terminal and OPC UA structures](#terminal-and-opc-ua-structures)
- [Diagnostics](#diagnostics)
- [Small helpers](#small-helpers)
- [Library version](#library-version)
- [Older blocks](#older-blocks)

## Before you start: defaults and fixed limits

**Logging is off until you turn it on.** `LogConfig.LogLevel` starts at `E_LogSeverity.None`, so nothing from the library or from your own `F_Log...` calls reaches the TwinCAT log until you set a level. See [Logging](#logging).

**The sizes below are constants compiled into the library.** They are not library parameters, so a project using the library cannot change them. Changing one means changing the constant in the library source and publishing a new version.

| Constant | Value | What it limits |
|---|---|---|
| `GVL_List.LIST_SIZE` | 100 | Items in every list, and entries in a dictionary |
| `StringsParams.MAX_STRING_LENGTH` | 300 | Size of `LongString`, which holds 299 characters |

`T_MaxString`, the result of `F_Concat3` to `F_Concat7` and the key type of the dictionary, holds 255 characters.

## Logging

### Turning it on

Set the level once at start-up. Messages at that severity and above are logged; `None` silences everything.

```pascal
LogConfig.LogLevel := E_LogSeverity.Info;
```

| Level | Logs |
|---|---|
| `Debug` | Debug, Info, Warning, Error |
| `Info` | Info, Warning, Error |
| `Warning` | Warning, Error |
| `Error` | Error |
| `None` | Nothing (the default) |

### Writing messages

```pascal
F_LogInfo('Conveyor started');
F_LogWarning(F_Concat3('Low pressure on ', lineName, ''));
F_LogError('Door opened while running');
F_LogDebug('Entering homing');
```

Each message goes to the TwinCAT log, which the XAE shows in the Error List. `F_LogDebug` adds `DEBUG ` to the front of its message. Each function returns the message when it was logged and an empty string when the level filtered it out.

`F_Logger(LogSeverity, Message)` does the same with the severity as a parameter.

### Logging once per edge

The edge blocks log when their `Event` input changes, not on every cycle while it stays set. `Q` is `TRUE` on the cycle the message was logged.

```pascal
VAR
  doorOpenedLog : FB_LogWarningEdge;
  doorClosedLog : FB_LogInfoEdge;
END_VAR

doorOpenedLog(Event := doorOpen, Message := 'Door opened');
doorClosedLog(Event := doorOpen, Message := 'Door closed', EventFallingEdge := TRUE);
```

`FB_LogInfoEdge`, `FB_LogWarningEdge` and `FB_LogErrorEdge` each log at their own severity. `FB_LogOnEdge` takes the severity as an input and accepts `Info`, `Warning` or `Error`. Given `Debug`, it logs an error saying the severity is wrong, with your message attached.

### Sending messages somewhere else as well

To copy every logged message to your own destination, such as a file, an HMI alarm list or a database, write a function block that implements `I_ExternalLogger` and hand an instance to `LogConfig.ExternalLogger`.

```pascal
FUNCTION_BLOCK T_HmiMessageLog IMPLEMENTS I_ExternalLogger
VAR
  lastMessage : T_MaxString;
  lastSeverity : E_LogSeverity;
END_VAR

METHOD AddToLog
VAR_IN_OUT
  Message : T_MaxString;
END_VAR
VAR_INPUT
  LogSeverity : E_LogSeverity;
END_VAR

lastMessage := Message;
lastSeverity := LogSeverity;
```

```pascal
PROGRAM MAIN
VAR
  hmiMessageLog : T_HmiMessageLog;
END_VAR

LogConfig.ExternalLogger := hmiMessageLog;
LogConfig.LogLevel := E_LogSeverity.Warning;
```

The external logger receives only the messages that pass the level. Declare its instance somewhere that lives as long as the PLC runs, such as a program or a global variable list. To stop forwarding, set `LogConfig.ExternalLogger := 0`.

## Named objects

Extend `T_NamedBase` to give a function block its own name and logging methods.

```pascal
FUNCTION_BLOCK T_Conveyor EXTENDS T_NamedBase

METHOD Start
LogInfo('Started');
```

```pascal
PROGRAM MAIN
VAR
  infeedConveyor : T_Conveyor;
END_VAR

infeedConveyor.Start();
```

| Member | For `MAIN.infeedConveyor` |
|---|---|
| `InstanceName` | `MAIN.infeedConveyor` (the instance path, without the project name) |
| `ObjectName` | `infeedConveyor` |
| `LogInfo('Started')` logs | `MAIN.infeedConveyor: Started` |

`LogDebug`, `LogInfo`, `LogWarning` and `LogError` follow `LogConfig.LogLevel` like the `F_Log...` functions.

## Lists

| Block | Holds |
|---|---|
| `FB_StringList` | `T_MaxString` |
| `FB_LongStringList` | `LongString` |
| `FB_UdintList` | `UDINT` |
| `FB_PointerList` | `PVOID` |
| `FB_TargList` | `T_Arg` |

All lists work the same way:

- Each holds up to `GVL_List.LIST_SIZE` items. `AddItem` returns `FALSE` once the list is full.
- Lists accept duplicates.
- Indexes start at 0. `FindIndex` returns -1 when the item is not there.
- `RemoveIndex` moves the later items down one place.
- `GetIndex` past the end returns an empty string, 0 or a null pointer, so prefer `TryGetIndex` when the index may be out of range.

```pascal
VAR
  activeAlarms : FB_StringList;
  alarmIndex : INT;
END_VAR

IF NOT activeAlarms.Contains('Door open') THEN
  activeAlarms.AddItem('Door open');
END_IF

alarmIndex := activeAlarms.FindIndex('Door open');
IF alarmIndex >= 0 THEN
  activeAlarms.RemoveIndex(TO_UDINT(alarmIndex));
END_IF
```

To go through every item, let `TryGetIndex` end the loop. A `FOR` loop to `Count - 1` runs away when the list is empty, because `Count` is unsigned.

```pascal
VAR
  index : UDINT;
  alarm : T_MaxString;
END_VAR

index := 0;
WHILE activeAlarms.TryGetIndex(index, alarm) DO
  F_LogWarning(alarm);
  index := index + 1;
END_WHILE
```

The string lists also have:

- `ContainsPartial` and `FindIndexPartialKey`, which match any item containing the given text.
- `CopyList` and `CopyListAndClear`, which copy every item into an `ARRAY[0..GVL_List.LIST_SIZE - 1]` of the list's item type.
- `GetAndRemoveIndex` on `FB_StringList`, which returns an item and removes it in one call.

`FB_TargList.Contains` and `FindIndex` compare the data each `T_Arg` points at, so two arguments for different variables holding the same value match.

## Dictionary

`FB_Dictionary` maps a text key to a `T_Arg`, which points at one of your variables. Use it to reach variables by name, for example settings that an HMI or a recipe file sends as text.

```pascal
VAR
  settings : FB_Dictionary;
  conveyorSpeed : LREAL := 1.5;
  operatingMode : INT := 2;
  setting : T_Arg;
  settingText : T_MaxString;
END_VAR

settings.AddItem('speed', F_LREAL(conveyorSpeed));
settings.AddItem('mode', F_INT(operatingMode));

IF settings.TryGet('speed', setting) THEN
  settingText := F_TargToString(setting);
  F_StringToTarg('2.0', setting);
END_IF
```

After this, `settingText` is `'1.5'` and `conveyorSpeed` is 2.0.

**The dictionary stores where the variable is, not a copy of its value.** Reading through the dictionary always gives the variable's current value, and writing through it changes the variable. Only add variables that live as long as the dictionary: never a function's local variable or a `VAR_TEMP`.

`AddItem` returns `FALSE` and adds nothing when:

- the key is empty,
- the value is empty (a `T_Arg` with no length),
- the key is already there,
- the dictionary already holds `GVL_List.LIST_SIZE` entries.

| Method | Does |
|---|---|
| `Contains(key)` | `TRUE` when the key is present |
| `TryGet(key, value)` | `TRUE` and the value when the key is present |
| `Get(key)` | The value, or an empty `T_Arg` |
| `TryGetPartialKey(partialKey, value)` | The value of the first key containing the text |
| `TryGetIndex(index, key, value)` | The key and value at a position, in the order they were added |
| `Remove(key)` | `TRUE` when the key was there and is now gone |
| `Clear()` | Empties the dictionary |
| `Count` | Number of entries |

### Noticing when any value changes

`F_DictionaryValuesCrc` returns a CRC-16 of every value in the dictionary, in the order they were added, and 0 for an empty dictionary. Compare it with the last one to see whether anything changed, for example before saving settings.

```pascal
settingsChecksum := F_DictionaryValuesCrc(settings);
IF settingsChecksum <> savedSettingsChecksum THEN
  SaveSettings();
  savedSettingsChecksum := settingsChecksum;
END_IF
```

## Strings

### Joining up to seven strings

```pascal
statusText := F_Concat4('Axis ', axisName, ' at ', TO_STRING(position));
```

`F_Concat3` to `F_Concat7` return a `T_MaxString`. Anything past 255 characters is cut off. Empty parts add nothing.

### Strings longer than 255 characters

`LongString` holds up to 299 characters. Join `LongString` variables with `F_ConcatLarge` for two, or `F_Concat3_Large` to `F_Concat7_Large` for more.

```pascal
VAR
  reportHeader, reportBody, reportFooter, report : LongString;
END_VAR

report := F_Concat3_Large(reportHeader, reportBody, reportFooter);
```

When the parts add up to more than 299 characters, the earlier parts are shortened so that the last part is kept whole.

`F_ClearLarge(report)` empties a `LongString` and zeroes its whole buffer.

The standard `LEN` stops counting at 255 characters. Measure a `LongString` with `LEN2(ADR(report))` from Tc2_Utilities.

## Parsing text into numbers

The `TryStringTo...` functions return `TRUE` when the whole text is a valid number for the type, and write the number into the variable you pass. When they return `FALSE`, the variable is set to 0.

```pascal
IF TryStringToDint(setpointText, setpoint) THEN
  F_LogInfo('Setpoint accepted');
ELSE
  F_LogWarning(F_Concat3('Not a whole number: ', setpointText, ''));
END_IF
```

| Function | Accepts | Rejects |
|---|---|---|
| `TryStringToDint` | Digits, with an optional leading minus: `123`, `-45` | `+1`, `1-2`, letters, spaces, empty text |
| `TryStringToUdint` | Digits up to 4294967295 | Any sign, spaces, empty text |
| `TryStringToWord` | Digits up to 65535 | 65536 and above, letters, empty text |
| `TryStringToLreal` | Digits with an optional point and leading minus: `3.25`, `-0.5`, `42` | Exponents such as `1e5`, letters, empty text |

`CharIsNumber`, `CharIsNumberOrMinus` and `CharIsNumberOrPeriod` test a single character, given as its byte value.

## Values held as T_Arg

`T_Arg` comes from Tc2_Utilities: it records where a variable is, its type and its size. Make one with the Tc2_Utilities functions such as `F_INT(variable)`, `F_LREAL(variable)` or `F_STRING(variable)`.

`F_TargToString(arg)` returns the variable's value as text. `F_StringToTarg(text, arg)` converts text and writes it into the variable.

Both handle `BOOL`, `BYTE`, `WORD`, `DWORD`, `USINT`, `SINT`, `UINT`, `INT`, `UDINT`, `DINT`, `REAL`, `LREAL` and `STRING`. For any other type they log a warning: `F_TargToString` returns an empty string and `F_StringToTarg` leaves the variable unchanged.

## Task information

`T_Task` reads the task information TwinCAT keeps for each PLC task. It looks at the first task until told otherwise, so in a project with more than one task, point it at the task it runs in.

```pascal
VAR
  task : T_Task;
END_VAR

task.SetToThisTask();
logFileName := F_Concat3('log_', task.DateTimeString, '.csv');
```

| Property | Gives |
|---|---|
| `Name` | Task name |
| `CycleTimeInMs`, `CycleTimeInSecs`, `CyclesPerSecond` | Configured cycle time |
| `CycleCount` | Cycles run so far |
| `IsFirstScan` | `TRUE` on the first cycle |
| `CycleTimeExceeded` | `TRUE` when the last cycle overran |
| `DistributedClockTimeInNs`, `...InMs`, `...InSecs` | Distributed clock time |
| `DateTimeFromDc` | Distributed clock time as `DATE_AND_TIME` |
| `DateTimeString` | `yyyy-mm-dd_hh-mm-ss`, safe to use in file names |
| `DateString` | `yyyy-mm-dd` |
| `TimeString` | `hh-mm-ss` |
| `Index` | Which task it reads; `SetToThisTask()` sets it to the calling task |

## Timer

`T_Timer` times from the distributed clock. It has no inputs and needs no cyclic call: set its properties and read its outputs whenever you like.

```pascal
VAR
  doorOpenTimer : T_Timer;
END_VAR

doorOpenTimer.SetPeriod := T#5S;
doorOpenTimer.Signal := doorOpen;

doorAlarm := doorOpenTimer.Q_On;
doorBeacon := doorOpenTimer.Q_PulseTrain;
```

| Property | Meaning |
|---|---|
| `Signal` | The input. Timing restarts whenever it changes; writing the same value every cycle does not restart it. |
| `SetPeriod` | The period. Anything under 1 ms is stored as 1 ms. |
| `Q_On` | `TRUE` once `Signal` has been on for longer than the period |
| `Q_Off` | `TRUE` once `Signal` has been off for longer than the period, after having been on at least once |
| `Q_PulseTrain` | While `Signal` is on: low for the first half of each period, high for the second half |
| `ElapsedTime`, `ElapsedTimeInMs` | Time since `Signal` came on; 0 while it is off |

## ADS

The ADS blocks run a request over several cycles. Each one:

- starts with a `Try...` method, which returns `FALSE` and logs a warning if a request is already busy,
- must be called every cycle (`myBlock();`) while the request runs,
- reports through `Busy`, `Done` and `Error`, which it also offers as the `I_TaskState` interface.

Addresses use `ST_AmsAddr`. Leave its `netId` all zero for the local system and set `port` to the target, for example 851 for the first PLC.

### Calling a method on a PLC by name

`T_InvokeRpcMethod` calls a method marked `{attribute 'TcRpcEnable'}` on any PLC it can reach over ADS. It gets a handle for the method, calls it and releases the handle in one request.

On the target PLC:

```pascal
FUNCTION_BLOCK T_Calculator

{attribute 'TcRpcEnable'}
METHOD AddNumbers : DINT
VAR_INPUT
  a : DINT;
  b : DINT;
END_VAR

AddNumbers := a + b;
```

On the calling PLC, a structure matching the method's inputs in order:

```pascal
TYPE ST_AddNumbersInputs :
STRUCT
  a : DINT;
  b : DINT;
END_STRUCT
END_TYPE
```

```pascal
VAR
  addNumbers : T_InvokeRpcMethod;
  addNumbersCall : ST_SymbolHandle;
  inputs : ST_AddNumbersInputs;
  sum : DINT;
END_VAR

addNumbers();

IF startCalculation THEN
  inputs.a := 20;
  inputs.b := 22;
  addNumbersCall.SymbolAddress.SymbolName := 'MAIN.calculator#AddNumbers';
  addNumbersCall.SymbolAddress.Ads.port := 851;
  addNumbersCall.SymbolAddress.WriteRef := F_BIGTYPE(pData := ADR(inputs), cbLen := SIZEOF(inputs));
  addNumbersCall.SymbolAddress.ReadRef := F_BIGTYPE(pData := ADR(sum), cbLen := SIZEOF(sum));
  addNumbers.TryInvokeMethodCall(symbolHandle := addNumbersCall);
  startCalculation := FALSE;
END_IF

calculationBusy := addNumbers.Busy;
calculationDone := addNumbers.Done;
calculationFailed := addNumbers.Error;
```

The symbol name is the instance path of the target function block, then `#`, then the method name. `WriteRef` points at the method's inputs and `ReadRef` at where to put its return value. `sum` holds 42 once `Done` is set. Set `ReadRef` and `WriteRef` to variables that live as long as the block, not to locals of a method or function.

### Working with a handle yourself

To keep a handle and use it more than once, take the steps `T_InvokeRpcMethod` takes for you:

1. `T_GetHandle.TryGetHandle(symbolHandle := handle)` fills in `handle.Handle` for `handle.SymbolAddress.SymbolName`.
2. `T_CallMethod.TryMethodCall(symbolHandle := handle)` writes `WriteRef` and reads `ReadRef` through the handle.
3. `T_ReleaseHandle.TryReleaseHandle(symbolHandle := handle)` frees it.

### Raw read and write

`T_AdsReadWrite.TryRequest(readWriteOptions)` and `T_AdsWrite.TryRequest(readWriteOptions)` send one ADS read-write or write to the index group and offset in `ST_AdsReadWriteOptions`. `WriteParams` and `ReadParams` are `T_Arg` values saying what to send and where to put the reply. The default timeout is 5 seconds.

## Controllers and filter

### P, I and PI

```pascal
VAR
  temperatureLoop : FB_PI_Controller;
END_VAR

temperatureLoop(
  ProcessVariable := measuredTemperature,
  Setpoint := targetTemperature,
  Kp := 2.0,
  IntegralConstant := 6.0,
  MaxLoopOutput := 100.0,
  MinLoopOutput := 0.0,
  Bias := 0.0
);
heaterPower := temperatureLoop.Output;
```

- `FB_P_Control`: `Term` is `Kp × (Setpoint − ProcessVariable)`.
- `FB_I_Controller`: each call adds `Ki × error × cycle time of the calling task` to `Term`, held between `MinIntegralComponent` and `MaxIntegralComponent` (±1000 by default). `Pause` holds the term, `Reset` restarts it from zero, and a `Ki` of 0 or less gives a term of 0.
- `FB_PI_Controller`: `Output` is the P term plus the I term plus `Bias`, held between `MinLoopOutput` and `MaxLoopOutput`. `IntegralConstant` is in repeats per minute, or per second when `RepeatsPerMinute` is `FALSE`. Integration pauses while the output is held at a limit and the error would push it further.

Call each controller exactly once per cycle.

**Set both output limits on `FB_PI_Controller`.** They default to 0, which holds the output at 0.

**Versions before 4.1.0.0 used the cycle time of the first PLC task**, whatever task called the controller. A controller in a task with a different cycle time integrated too fast or too slow. From 4.1.0.0 each controller uses its own task, so check the tuning of any loop that runs outside the first task.

### Alpha-beta filter

`FB_AlphaBetaFilter` smooths a measurement and tracks its rate of change.

```pascal
pressureFilter(
  measurement := rawPressure,
  deltaInSecond := task.CycleTimeInSecs,
  alpha := 0.3,
  beta := 0.01
);
smoothedPressure := pressureFilter.output;
```

`alpha` runs from 0 to 1: 1 passes the measurement straight through, smaller values smooth more. Keep `beta` small, between 0.001 and 0.1. With a `deltaInSecond` of 0 the rate term is left out.

## Terminal and OPC UA structures

Process-image structures to declare and link in the I/O tree:

| Structure | For |
|---|---|
| `stEL3058_Channel`, `stEL3152_Channel`, `stEL3318_Channel` | One channel of an EL3058, EL3152 or EL3318 analogue input |
| `stKL306x_Channel` | One channel of a KL3061 or KL3062 analogue input |
| `stKL33xx_Channel` | One channel of a KL33xx input such as the KL3314 |
| `stEL9410` | EL9410 supply undervoltage bits |
| `stEL9576` | EL9576 brake chopper status |
| `stWcState`, `stInfoData` | EtherCAT working counter and info data of a slave |

`F_GetKL306xState` turns a `stKL306x_ChannelStatus` into named outputs: underrange, overrange, error, and whether the value is equal to, above or below each limit or that limit is off.

```pascal
F_GetKL306xState(
  KL306xState := tankLevelInput.State,
  Overrange => tankLevelOverrange,
  ProcessData_GT_Limit1 => tankLevelHigh
);
```

The OPC UA structures match the control word, status word and data points of the TwinCAT OPC UA client I/O device: `ST_OpcUAServer_Control`, `ST_OpcUAServer_Status`, `ST_OpcUa_LrealPoint`, `ST_OpcUa_UintPoint` and their `...Write` forms.

## Diagnostics

### CPU temperature

`T_MdpReadCpuTemp` reads the IPC's CPU temperature through the device manager every 500 ms. Call it every cycle and read `CpuTemp`, in degrees Celsius.

```pascal
VAR
  cpuTemperature : T_MdpReadCpuTemp;
  cpuHotLog : FB_LogWarningEdge;
END_VAR

cpuTemperature();
cpuHotLog(Event := cpuTemperature.CpuTemp > 80, Message := 'CPU above 80 degrees');
```

`SetUpdateTimerValue(T#2S)` changes how often it reads. `Busy`, `Error` and `ErrorId` report the read in progress. It needs the Tc2_MDP library, which is only on Beckhoff's testing TcPkg feed.

### Saving persistent data regularly

`T_WritePersistentData` writes the persistent variables of a PLC at a fixed period. By default it writes on ADS port 851 every 10 seconds. Call it every cycle.

```pascal
VAR
  persistentWriter : T_WritePersistentData;
  persistentWriterConfigured : BOOL;
END_VAR

IF NOT persistentWriterConfigured THEN
  persistentWriterConfigured := persistentWriter.SetConfig(WritePeriod := T#1M, AdsPort := 852);
END_IF
persistentWriter();
```

`SetConfig` returns `TRUE` when it takes the new period and port. A zero period or port 0 is rejected: it returns `FALSE` and the current settings stay. `WritePeriod` and `AdsPort` read the settings back. Before 4.1.0.0, `SetConfig` did nothing.

## Small helpers

- `F_Gate(flag)` returns the flag and clears it, so a request flag is acted on once. The example after this list shows it.
- `F_GetAmsNetId(netIdBytes)` turns the six bytes of a `T_AmsNetIdArr` into text such as `5.6.7.8.1.1`, and all zeros into an empty string, which ADS reads as the local system.
- `NULL` is a null pointer constant.

```pascal
IF F_Gate(saveRequested) THEN
  SaveSettings();
END_IF
```

## Library version

```pascal
libraryVersion := F_GetVersion();
```

`F_GetVersion` returns an `ST_LibVersion` with the major, minor, build and revision numbers and the version as text. `F_GetTitle` and `F_GetCompany` return the library's title and company. The same version is in the constant `Global_Version.stLibVersion_Intecre_Utilities`.

## Older blocks

These remain so that existing projects keep compiling. Don't use them in new code:

- `FB_Assert` and `T_TestBase` were written before TcUnit. Use [TcUnit](https://tcunit.org) instead.
- `TGenericCollection` is unfinished and does nothing.
