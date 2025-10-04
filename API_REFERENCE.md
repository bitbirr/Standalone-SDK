# ZKTeco SDK API Reference

## Overview

This document provides a comprehensive reference for the ZKTeco Standalone SDK API methods. The SDK is exposed through the `CZKEMClass` COM interface.

## Table of Contents

1. [Connection Methods](#connection-methods)
2. [User Management](#user-management)
3. [Attendance Logs](#attendance-logs)
4. [Device Management](#device-management)
5. [Access Control](#access-control)
6. [Advanced Features](#advanced-features)
7. [Event Handling](#event-handling)
8. [Error Handling](#error-handling)

---

## Connection Methods

### Connect_Net
Connects to device via TCP/IP network.

```csharp
bool Connect_Net(string IPAddr, int Port)
```

**Parameters:**
- `IPAddr` (string): IP address of the device (e.g., "192.168.1.100")
- `Port` (int): Communication port (default: 4370)

**Returns:** `true` if connection successful, `false` otherwise

**Example:**
```csharp
CZKEMClass device = new CZKEMClass();
if (device.Connect_Net("192.168.1.100", 4370))
{
    Console.WriteLine("Connected successfully");
}
```

---

### Connect_USB
Connects to device via USB.

```csharp
bool Connect_USB(int MachineNumber)
```

**Parameters:**
- `MachineNumber` (int): Machine number (usually 1)

**Returns:** `true` if connection successful, `false` otherwise

**Example:**
```csharp
if (device.Connect_USB(1))
{
    Console.WriteLine("Connected via USB");
}
```

---

### Connect_Com
Connects to device via RS232/RS485.

```csharp
bool Connect_Com(int ComPort, int MachineNumber, int BaudRate)
```

**Parameters:**
- `ComPort` (int): COM port number (e.g., 1 for COM1)
- `MachineNumber` (int): Machine number (usually 1)
- `BaudRate` (int): Communication speed (9600, 19200, 38400, 57600, 115200)

**Returns:** `true` if connection successful, `false` otherwise

**Example:**
```csharp
if (device.Connect_Com(1, 1, 115200))
{
    Console.WriteLine("Connected via COM1");
}
```

---

### Disconnect
Disconnects from the device.

```csharp
void Disconnect()
```

**Example:**
```csharp
device.Disconnect();
```

---

## User Management

### SSR_GetAllUserInfo
Retrieves all user information from device.

```csharp
bool SSR_GetAllUserInfo(
    int dwMachineNumber,
    out string dwEnrollNumber,
    out string Name,
    out string Password,
    out int Privilege,
    out bool Enabled
)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `dwEnrollNumber` (out string): User enrollment number
- `Name` (out string): User name
- `Password` (out string): User password
- `Privilege` (out int): User privilege (0=User, 2=Administrator, 14=SuperAdmin)
- `Enabled` (out bool): User enabled status

**Returns:** `true` while more users available, `false` when done

**Example:**
```csharp
device.ReadAllUserID(machineNumber);
string enrollNumber, name, password;
int privilege;
bool enabled;

while (device.SSR_GetAllUserInfo(machineNumber, 
    out enrollNumber, out name, out password, 
    out privilege, out enabled))
{
    Console.WriteLine($"User: {name} ({enrollNumber})");
}
```

---

### SSR_SetUserInfo
Sets or updates user information.

```csharp
bool SSR_SetUserInfo(
    int dwMachineNumber,
    string dwEnrollNumber,
    string Name,
    string Password,
    int Privilege,
    bool Enabled
)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `dwEnrollNumber` (string): User enrollment number (unique ID)
- `Name` (string): User name
- `Password` (string): User password (optional, can be empty)
- `Privilege` (int): User privilege level
  - `0` = Regular user
  - `2` = Administrator (can enroll users)
  - `14` = Super administrator
- `Enabled` (bool): Enable/disable user

**Returns:** `true` if successful, `false` otherwise

**Example:**
```csharp
bool result = device.SSR_SetUserInfo(1, "1001", "John Doe", "", 0, true);
if (result)
{
    Console.WriteLine("User created/updated successfully");
}
```

---

### SSR_DeleteEnrollData
Deletes user enrollment data.

```csharp
bool SSR_DeleteEnrollData(
    int dwMachineNumber,
    string dwEnrollNumber,
    int dwBackupNumber
)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `dwEnrollNumber` (string): User enrollment number
- `dwBackupNumber` (int): Backup number (use machine number)

**Returns:** `true` if successful, `false` otherwise

**Example:**
```csharp
bool deleted = device.SSR_DeleteEnrollData(1, "1001", 1);
```

---

### ReadAllUserID
Loads all user IDs into memory for retrieval.

```csharp
bool ReadAllUserID(int dwMachineNumber)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number

**Returns:** `true` if successful, `false` otherwise

**Note:** Call this before using `SSR_GetAllUserInfo()`

---

### SetUserInfo
Alternative method to set user info (older API).

```csharp
bool SetUserInfo(
    int dwMachineNumber,
    int dwEnrollNumber,
    string Name,
    string Password,
    int Privilege,
    bool Enabled
)
```

**Note:** Similar to `SSR_SetUserInfo` but uses int for enrollment number

---

## Attendance Logs

### ReadGeneralLogData
Reads all attendance logs from device.

```csharp
bool ReadGeneralLogData(int dwMachineNumber)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number

**Returns:** `true` if successful, `false` otherwise

**Note:** Call this before retrieving individual log entries

**Example:**
```csharp
device.EnableDevice(machineNumber, false);
if (device.ReadGeneralLogData(machineNumber))
{
    Console.WriteLine("Logs loaded successfully");
}
device.EnableDevice(machineNumber, true);
```

---

### SSR_GetGeneralLogData
Retrieves individual attendance log entries.

```csharp
bool SSR_GetGeneralLogData(
    int dwMachineNumber,
    out string dwEnrollNumber,
    out int dwVerifyMode,
    out int dwInOutMode,
    out int dwYear,
    out int dwMonth,
    out int dwDay,
    out int dwHour,
    out int dwMinute,
    out int dwSecond,
    ref int dwWorkCode
)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `dwEnrollNumber` (out string): User enrollment number
- `dwVerifyMode` (out int): Verification method
  - `0` = Password
  - `1` = Fingerprint
  - `2` = Card
  - `3` = Face
- `dwInOutMode` (out int): Check in/out status
  - `0` = Check in
  - `1` = Check out
  - `2` = Break out
  - `3` = Break in
  - `4` = Overtime in
  - `5` = Overtime out
- `dwYear, dwMonth, dwDay, dwHour, dwMinute, dwSecond` (out int): Timestamp
- `dwWorkCode` (ref int): Work code

**Returns:** `true` while more logs available, `false` when done

**Example:**
```csharp
string enrollNumber;
int verifyMode, inOutMode, year, month, day, hour, minute, second;
int workCode = 0;

while (device.SSR_GetGeneralLogData(machineNumber,
    out enrollNumber, out verifyMode, out inOutMode,
    out year, out month, out day,
    out hour, out minute, out second, ref workCode))
{
    DateTime logTime = new DateTime(year, month, day, hour, minute, second);
    Console.WriteLine($"{enrollNumber}: {logTime} - Mode: {verifyMode}");
}
```

---

### GetGeneralLogData
Alternative method to get attendance logs (older API).

```csharp
bool GetGeneralLogData(
    int dwMachineNumber,
    ref int dwEnrollNumber,
    ref int dwVerifyMode,
    ref int dwInOutMode,
    ref int dwYear,
    ref int dwMonth,
    ref int dwDay,
    ref int dwHour,
    ref int dwMinute,
    ref int dwSecond
)
```

**Note:** Similar to `SSR_GetGeneralLogData` but uses int for enrollment number

---

### ClearGLog
Clears all attendance logs from device.

```csharp
bool ClearGLog(int dwMachineNumber)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number

**Returns:** `true` if successful, `false` otherwise

**Warning:** This permanently deletes all attendance logs from the device

**Example:**
```csharp
if (device.ClearGLog(1))
{
    Console.WriteLine("All logs cleared");
}
```

---

### GetAttLogByTimeEx
Gets attendance logs within a time range.

```csharp
bool GetAttLogByTimeEx(
    int dwMachineNumber,
    string dwEnrollNumber,
    int dwStartYear, int dwStartMonth, int dwStartDay,
    int dwEndYear, int dwEndMonth, int dwEndDay
)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `dwEnrollNumber` (string): User enrollment number (empty for all users)
- Start/End date components

**Returns:** `true` if successful, `false` otherwise

---

## Device Management

### EnableDevice
Enables or disables device operations.

```csharp
bool EnableDevice(int dwMachineNumber, bool bFlag)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `bFlag` (bool): `true` to enable, `false` to disable

**Returns:** `true` if successful, `false` otherwise

**Note:** Disable device before bulk data operations for better performance

**Example:**
```csharp
device.EnableDevice(1, false);  // Disable for data operations
// ... perform operations
device.EnableDevice(1, true);   // Re-enable
```

---

### GetDeviceStatus
Gets device status information.

```csharp
bool GetDeviceStatus(
    int dwMachineNumber,
    int dwStatusType,
    ref int dwValue
)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number
- `dwStatusType` (int): Status type to query
  - `1` = Administrator count
  - `2` = User count
  - `8` = Attendance log count
  - `21` = Fingerprint count
  - `22` = Password count
  - `23` = Card count
  - `24` = Face count
- `dwValue` (ref int): Returned value

**Returns:** `true` if successful, `false` otherwise

**Example:**
```csharp
int userCount = 0, logCount = 0;
device.GetDeviceStatus(1, 2, ref userCount);
device.GetDeviceStatus(1, 8, ref logCount);
Console.WriteLine($"Users: {userCount}, Logs: {logCount}");
```

---

### GetDeviceTime
Gets current time from device.

```csharp
bool GetDeviceTime(
    int dwMachineNumber,
    ref int dwYear,
    ref int dwMonth,
    ref int dwDay,
    ref int dwHour,
    ref int dwMinute,
    ref int dwSecond
)
```

**Returns:** `true` if successful, `false` otherwise

**Example:**
```csharp
int year = 0, month = 0, day = 0, hour = 0, minute = 0, second = 0;
if (device.GetDeviceTime(1, ref year, ref month, ref day, 
    ref hour, ref minute, ref second))
{
    DateTime deviceTime = new DateTime(year, month, day, hour, minute, second);
    Console.WriteLine($"Device time: {deviceTime}");
}
```

---

### SetDeviceTime
Sets device time.

```csharp
bool SetDeviceTime(int dwMachineNumber)
```

**Parameters:**
- `dwMachineNumber` (int): Machine number

**Returns:** `true` if successful, `false` otherwise

**Note:** Sets device time to current PC time

---

### SetDeviceTime2
Sets device time to specific value.

```csharp
bool SetDeviceTime2(
    int dwMachineNumber,
    int dwYear,
    int dwMonth,
    int dwDay,
    int dwHour,
    int dwMinute,
    int dwSecond
)
```

**Example:**
```csharp
DateTime now = DateTime.Now;
device.SetDeviceTime2(1, now.Year, now.Month, now.Day, 
    now.Hour, now.Minute, now.Second);
```

---

### GetSerialNumber
Gets device serial number.

```csharp
bool GetSerialNumber(int dwMachineNumber, out string dwSerialNumber)
```

**Example:**
```csharp
string serialNumber;
if (device.GetSerialNumber(1, out serialNumber))
{
    Console.WriteLine($"Serial: {serialNumber}");
}
```

---

### GetFirmwareVersion
Gets firmware version.

```csharp
bool GetFirmwareVersion(int dwMachineNumber, ref string strVersion)
```

**Example:**
```csharp
string version = "";
device.GetFirmwareVersion(1, ref version);
Console.WriteLine($"Firmware: {version}");
```

---

### GetSDKVersion
Gets SDK version.

```csharp
bool GetSDKVersion(ref string strVersion)
```

**Example:**
```csharp
string sdkVersion = "";
device.GetSDKVersion(ref sdkVersion);
Console.WriteLine($"SDK Version: {sdkVersion}");
```

---

### GetPlatform
Gets device platform information.

```csharp
bool GetPlatform(int dwMachineNumber, ref string strPlatform)
```

**Example:**
```csharp
string platform = "";
device.GetPlatform(1, ref platform);
Console.WriteLine($"Platform: {platform}");
```

---

### PowerOffDevice
Powers off the device.

```csharp
bool PowerOffDevice(int dwMachineNumber)
```

**Warning:** Use with caution

---

### RestartDevice
Restarts the device.

```csharp
bool RestartDevice(int dwMachineNumber)
```

---

### ClearData
Clears all data from device.

```csharp
bool ClearData(int dwMachineNumber, int dwDataFlag)
```

**Parameters:**
- `dwDataFlag` (int): Data type to clear
  - `1` = Attendance logs
  - `2` = User data
  - `3` = SMS
  - `4` = Photo
  - `5` = All data

**Warning:** This is destructive and cannot be undone

---

## Access Control

### GetDoorState
Gets door state.

```csharp
bool GetDoorState(
    int MachineNumber,
    ref int State
)
```

**Parameters:**
- `State` (ref int): Door state
  - `0` = Closed
  - `1` = Open

---

### SetDoorStatus
Sets door status (lock/unlock).

```csharp
bool SetDoorStatus(
    int MachineNumber,
    int DoorNo,
    int Status
)
```

**Parameters:**
- `DoorNo` (int): Door number
- `Status` (int): 
  - `0` = Locked
  - `1` = Unlocked

---

### ControlDevice
Generic device control command.

```csharp
bool ControlDevice(
    int dwMachineNumber,
    int dwOperationID,
    int dwParam
)
```

**Operation IDs:**
- `1` = Restart device
- `2` = Power off
- `3` = Enable/disable device

---

## Advanced Features

### GetUserTmpExStr
Gets user fingerprint template as string.

```csharp
bool GetUserTmpExStr(
    int dwMachineNumber,
    string dwEnrollNumber,
    int dwFingerIndex,
    out int dwFlag,
    out string TmpData,
    out int TmpLength
)
```

**Parameters:**
- `dwFingerIndex` (int): Finger index (0-9)
- `dwFlag` (out int): Template flag/valid status
- `TmpData` (out string): Template data
- `TmpLength` (out int): Template length

---

### SetUserTmpExStr
Sets user fingerprint template.

```csharp
bool SetUserTmpExStr(
    int dwMachineNumber,
    string dwEnrollNumber,
    int dwFingerIndex,
    int Flag,
    string TmpData
)
```

---

### SSR_GetUserInfo
Gets specific user information.

```csharp
bool SSR_GetUserInfo(
    int dwMachineNumber,
    string dwEnrollNumber,
    out string Name,
    out string Password,
    out int Privilege,
    out bool Enabled
)
```

---

### BeginBatchUpdate
Begins batch update mode.

```csharp
bool BeginBatchUpdate(int dwMachineNumber)
```

**Note:** Improves performance when making multiple changes

---

### BatchUpdate
Commits batch updates.

```csharp
bool BatchUpdate(int dwMachineNumber)
```

---

### CancelBatchUpdate
Cancels batch updates.

```csharp
bool CancelBatchUpdate(int dwMachineNumber)
```

---

## Event Handling

### RegEvent
Registers for real-time events.

```csharp
bool RegEvent(int dwMachineNumber, int EventMask)
```

**Parameters:**
- `EventMask` (int): Event types to register
  - `65535` = All events
  - `1` = Attendance transaction

**Example:**
```csharp
device.OnAttTransactionEx += Device_OnAttTransactionEx;
device.Connect_Net("192.168.1.100", 4370);
device.RegEvent(1, 65535);

void Device_OnAttTransactionEx(string EnrollNumber, int IsInValid, 
    int AttState, int VerifyMethod, int Year, int Month, int Day,
    int Hour, int Minute, int Second, int WorkCode)
{
    Console.WriteLine($"Real-time: {EnrollNumber} at {Hour}:{Minute}:{Second}");
}
```

---

### OnAttTransactionEx Event
Real-time attendance transaction event.

```csharp
event OnAttTransactionExEventHandler OnAttTransactionEx;
```

---

### OnConnected Event
Device connected event.

```csharp
event OnConnectedEventHandler OnConnected;
```

---

### OnDisConnected Event
Device disconnected event.

```csharp
event OnDisConnectedEventHandler OnDisConnected;
```

---

### OnFingerEvent
Fingerprint detected event.

```csharp
event OnFingerEventHandler OnFinger;
```

---

### OnVerifyEvent
Verification event.

```csharp
event OnVerifyEventHandler OnVerify;
```

---

### OnNewUserEvent
New user enrolled event.

```csharp
event OnNewUserEventHandler OnNewUser;
```

---

## Error Handling

### GetLastError
Gets the last error code.

```csharp
bool GetLastError(ref int dwErrorCode)
```

**Common Error Codes:**
- `0` = Success
- `-1` = General error
- `-2` = Invalid parameter
- `-3` = Not supported
- `-4` = Buffer too small
- `-5` = Data not found
- `-10` = Transmission failed
- `-100` = Device busy

**Example:**
```csharp
int errorCode = 0;
device.GetLastError(ref errorCode);
Console.WriteLine($"Error Code: {errorCode}");
```

---

## Best Practices

### 1. Always Disable Device for Bulk Operations
```csharp
device.EnableDevice(machineNumber, false);
// Perform operations
device.EnableDevice(machineNumber, true);
```

### 2. Proper Error Handling
```csharp
try
{
    if (!device.Connect_Net(ip, port))
    {
        int errorCode = 0;
        device.GetLastError(ref errorCode);
        Console.WriteLine($"Connection failed. Error: {errorCode}");
    }
}
catch (Exception ex)
{
    Console.WriteLine($"Exception: {ex.Message}");
}
```

### 3. Clean Up COM Objects
```csharp
device.Disconnect();
System.Runtime.InteropServices.Marshal.ReleaseComObject(device);
device = null;
GC.Collect();
```

### 4. Use Batch Updates for Multiple Changes
```csharp
device.BeginBatchUpdate(machineNumber);
// Make multiple changes
device.SetUserInfo(...);
device.SetUserInfo(...);
device.BatchUpdate(machineNumber);
```

---

## Version Information

- **SDK Version**: 6.2.4.11
- **Architecture**: 32-bit (x86)
- **Platform**: Windows

## Additional Resources

- [Quick Start Guide](QUICK_START.md)
- [.NET Integration Guide](DOTNET_INTEGRATION_GUIDE.md)
- [SDK Review](SDK_REVIEW.md)

---

**Note**: This reference is based on SDK version 6.2.4.11. Some methods may vary in different versions.
