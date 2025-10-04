# .NET Integration Guide for AttWeb Project

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Installation & Setup](#installation--setup)
4. [COM Interop Configuration](#com-interop-configuration)
5. [Integration Patterns](#integration-patterns)
6. [Code Examples](#code-examples)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

## Overview

This guide provides step-by-step instructions for integrating the ZKTeco Standalone SDK with your .NET AttWeb project. The SDK uses COM technology, which requires COM Interop in .NET applications.

## Prerequisites

### Required Software
- .NET Framework 4.6.1+ or .NET 6+ (Windows only)
- Visual Studio 2019 or later
- Administrator privileges (for initial setup)
- Windows 7 or later (32-bit or 64-bit)

### Required Hardware
- ZKTeco biometric device (time attendance or access control terminal)
- Network connection (for TCP/IP) OR USB cable OR RS232/RS485 adapter

## Installation & Setup

### Option 1: Traditional Installation (Using System32)

**⚠️ Warning**: This method requires administrator privileges and affects system-wide settings.

```batch
# Run as Administrator
cd "Communication Protocol SDK(32Bit Ver6.2.4.11)"
Auto-install_sdk.bat
```

### Option 2: Private Deployment (Recommended for Modern Apps)

This method doesn't require administrator privileges and keeps your application isolated.

1. **Copy SDK DLLs to Your Project**:
   ```
   YourProject/
   ├── bin/
   │   ├── Debug/
   │   │   ├── zkemkeeper.dll
   │   │   ├── zkemsdk.dll
   │   │   ├── (other SDK DLLs)
   ```

2. **Register DLLs Locally** (one-time setup per development machine):
   ```batch
   # Run in your project's bin directory as Administrator
   regsvr32 zkemkeeper.dll
   ```

3. **For Deployment**: Use a setup/installer that registers the DLL or use registration-free COM (see Advanced section).

## COM Interop Configuration

### Step 1: Add COM Reference in Visual Studio

1. **Right-click on your project** → **Add** → **Reference**
2. Select **COM** tab
3. Find and check **"zkemkeeper 1.0 Type Library"**
4. Click **OK**

Visual Studio will automatically generate an Interop assembly (Interop.zkemkeeper.dll).

### Step 2: Alternative - Manual Type Library Import

If the COM reference doesn't appear, use tlbimp.exe:

```batch
# Developer Command Prompt for Visual Studio
tlbimp zkemkeeper.dll /out:Interop.zkemkeeper.dll /namespace:ZKEMKEEPERLib
```

Then add the generated Interop.zkemkeeper.dll as a reference to your project.

### Step 3: Configure Build Settings

For **32-bit SDK**, set your project to **x86**:

1. **Project Properties** → **Build**
2. Set **Platform target** to **x86**
3. Uncheck **Prefer 32-bit** (if using .NET Framework 4.5+)

## Integration Patterns

### Pattern 1: Direct Integration (Small Applications)

Use the SDK directly in your application code.

**Pros**: Simple, quick to implement
**Cons**: Tight coupling, hard to test, device-dependent

### Pattern 2: Service Layer (Recommended)

Create a service layer to abstract device communication.

**Pros**: Better separation of concerns, easier to test, maintainable
**Cons**: More initial setup

### Pattern 3: Windows Service + REST API (Enterprise)

Create a Windows Service that communicates with devices and exposes a REST API.

**Pros**: Scalable, can support web applications, centralized management
**Cons**: More complex architecture

### Pattern 4: Wrapper Library (Reusable)

Create a managed .NET wrapper library for the SDK.

**Pros**: Reusable across projects, cleaner API, better error handling
**Cons**: Requires initial development effort

## Code Examples

### Example 1: Basic Device Connection

```csharp
using zkemkeeper;
using System;

namespace AttWeb.Services
{
    public class BiometricDeviceService
    {
        private CZKEMClass zkemDevice;
        private int machineNumber = 1;
        
        public BiometricDeviceService()
        {
            zkemDevice = new CZKEMClass();
        }
        
        // Connect via TCP/IP
        public bool ConnectToDevice(string ipAddress, int port = 4370)
        {
            try
            {
                return zkemDevice.Connect_Net(ipAddress, port);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Connection failed: {ex.Message}");
                return false;
            }
        }
        
        // Connect via USB
        public bool ConnectViaUSB()
        {
            try
            {
                return zkemDevice.Connect_USB(machineNumber);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"USB connection failed: {ex.Message}");
                return false;
            }
        }
        
        // Disconnect from device
        public void Disconnect()
        {
            try
            {
                zkemDevice.Disconnect();
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Disconnect failed: {ex.Message}");
            }
        }
        
        // Get last error
        public string GetLastError()
        {
            int errorCode = 0;
            zkemDevice.GetLastError(ref errorCode);
            return $"Error Code: {errorCode}";
        }
    }
}
```

### Example 2: Retrieve Attendance Logs

```csharp
using zkemkeeper;
using System;
using System.Collections.Generic;

namespace AttWeb.Models
{
    public class AttendanceLog
    {
        public string EnrollNumber { get; set; }
        public int VerifyMode { get; set; }
        public int InOutMode { get; set; }
        public DateTime LogTime { get; set; }
        public int WorkCode { get; set; }
    }
}

namespace AttWeb.Services
{
    public class AttendanceService
    {
        private CZKEMClass zkemDevice;
        private int machineNumber = 1;
        
        public AttendanceService(CZKEMClass device)
        {
            zkemDevice = device;
        }
        
        // Read all attendance logs
        public List<AttendanceLog> GetAllAttendanceLogs()
        {
            var logs = new List<AttendanceLog>();
            
            try
            {
                // Disable device to speed up data reading
                zkemDevice.EnableDevice(machineNumber, false);
                
                // Read general log data
                if (zkemDevice.ReadGeneralLogData(machineNumber))
                {
                    string enrollNumber = "";
                    int verifyMode = 0;
                    int inOutMode = 0;
                    int year = 0, month = 0, day = 0;
                    int hour = 0, minute = 0, second = 0;
                    int workCode = 0;
                    
                    while (zkemDevice.SSR_GetGeneralLogData(machineNumber, 
                        out enrollNumber, out verifyMode, out inOutMode,
                        out year, out month, out day, 
                        out hour, out minute, out second, 
                        ref workCode))
                    {
                        var log = new AttendanceLog
                        {
                            EnrollNumber = enrollNumber,
                            VerifyMode = verifyMode,
                            InOutMode = inOutMode,
                            LogTime = new DateTime(year, month, day, hour, minute, second),
                            WorkCode = workCode
                        };
                        
                        logs.Add(log);
                    }
                }
                
                // Re-enable device
                zkemDevice.EnableDevice(machineNumber, true);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to read logs: {ex.Message}");
                zkemDevice.EnableDevice(machineNumber, true);
            }
            
            return logs;
        }
        
        // Clear attendance logs from device
        public bool ClearAttendanceLogs()
        {
            try
            {
                return zkemDevice.ClearGLog(machineNumber);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to clear logs: {ex.Message}");
                return false;
            }
        }
    }
}
```

### Example 3: User Management

```csharp
using zkemkeeper;
using System;
using System.Collections.Generic;

namespace AttWeb.Models
{
    public class BiometricUser
    {
        public string EnrollNumber { get; set; }
        public string Name { get; set; }
        public string Password { get; set; }
        public int Privilege { get; set; } // 0=User, 2=Administrator, 14=SuperAdmin
        public bool Enabled { get; set; }
    }
}

namespace AttWeb.Services
{
    public class UserManagementService
    {
        private CZKEMClass zkemDevice;
        private int machineNumber = 1;
        
        public UserManagementService(CZKEMClass device)
        {
            zkemDevice = device;
        }
        
        // Get all users from device
        public List<BiometricUser> GetAllUsers()
        {
            var users = new List<BiometricUser>();
            
            try
            {
                zkemDevice.EnableDevice(machineNumber, false);
                
                if (zkemDevice.ReadAllUserID(machineNumber))
                {
                    string enrollNumber = "";
                    string name = "";
                    string password = "";
                    int privilege = 0;
                    bool enabled = true;
                    
                    while (zkemDevice.SSR_GetAllUserInfo(machineNumber, 
                        out enrollNumber, out name, out password, 
                        out privilege, out enabled))
                    {
                        var user = new BiometricUser
                        {
                            EnrollNumber = enrollNumber,
                            Name = name,
                            Password = password,
                            Privilege = privilege,
                            Enabled = enabled
                        };
                        
                        users.Add(user);
                    }
                }
                
                zkemDevice.EnableDevice(machineNumber, true);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to read users: {ex.Message}");
                zkemDevice.EnableDevice(machineNumber, true);
            }
            
            return users;
        }
        
        // Add or update user
        public bool SetUser(BiometricUser user)
        {
            try
            {
                zkemDevice.EnableDevice(machineNumber, false);
                
                bool result = zkemDevice.SSR_SetUserInfo(machineNumber, 
                    user.EnrollNumber, user.Name, user.Password, 
                    user.Privilege, user.Enabled);
                
                zkemDevice.EnableDevice(machineNumber, true);
                
                return result;
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to set user: {ex.Message}");
                zkemDevice.EnableDevice(machineNumber, true);
                return false;
            }
        }
        
        // Delete user
        public bool DeleteUser(string enrollNumber)
        {
            try
            {
                zkemDevice.EnableDevice(machineNumber, false);
                
                bool result = zkemDevice.SSR_DeleteEnrollData(machineNumber, 
                    enrollNumber, machineNumber);
                
                zkemDevice.EnableDevice(machineNumber, true);
                
                return result;
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to delete user: {ex.Message}");
                zkemDevice.EnableDevice(machineNumber, true);
                return false;
            }
        }
    }
}
```

### Example 4: Real-Time Event Handling

```csharp
using zkemkeeper;
using System;

namespace AttWeb.Services
{
    public class RealtimeEventService
    {
        private CZKEMClass zkemDevice;
        private int machineNumber = 1;
        
        public event EventHandler<AttendanceEventArgs> OnAttendance;
        public event EventHandler<string> OnDeviceConnected;
        public event EventHandler<string> OnDeviceDisconnected;
        
        public RealtimeEventService(CZKEMClass device)
        {
            zkemDevice = device;
            RegisterEvents();
        }
        
        private void RegisterEvents()
        {
            // Register for real-time events
            zkemDevice.OnAttTransactionEx += ZkemDevice_OnAttTransactionEx;
            zkemDevice.OnConnected += ZkemDevice_OnConnected;
            zkemDevice.OnDisConnected += ZkemDevice_OnDisConnected;
        }
        
        public bool EnableRealTimeEvents()
        {
            try
            {
                return zkemDevice.RegEvent(machineNumber, 65535);
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Failed to register events: {ex.Message}");
                return false;
            }
        }
        
        private void ZkemDevice_OnAttTransactionEx(string enrollNumber, 
            int isInValid, int attState, int verifyMethod, 
            int year, int month, int day, 
            int hour, int minute, int second, int workCode)
        {
            var args = new AttendanceEventArgs
            {
                EnrollNumber = enrollNumber,
                IsValid = isInValid == 0,
                AttendanceState = attState,
                VerifyMethod = verifyMethod,
                Time = new DateTime(year, month, day, hour, minute, second),
                WorkCode = workCode
            };
            
            OnAttendance?.Invoke(this, args);
        }
        
        private void ZkemDevice_OnConnected()
        {
            OnDeviceConnected?.Invoke(this, "Device connected");
        }
        
        private void ZkemDevice_OnDisConnected()
        {
            OnDeviceDisconnected?.Invoke(this, "Device disconnected");
        }
    }
    
    public class AttendanceEventArgs : EventArgs
    {
        public string EnrollNumber { get; set; }
        public bool IsValid { get; set; }
        public int AttendanceState { get; set; }
        public int VerifyMethod { get; set; }
        public DateTime Time { get; set; }
        public int WorkCode { get; set; }
    }
}
```

### Example 5: Device Information

```csharp
using zkemkeeper;
using System;

namespace AttWeb.Services
{
    public class DeviceInfoService
    {
        private CZKEMClass zkemDevice;
        private int machineNumber = 1;
        
        public DeviceInfoService(CZKEMClass device)
        {
            zkemDevice = device;
        }
        
        // Get device serial number
        public string GetSerialNumber()
        {
            string serialNumber = "";
            zkemDevice.GetSerialNumber(machineNumber, out serialNumber);
            return serialNumber;
        }
        
        // Get device firmware version
        public string GetFirmwareVersion()
        {
            string firmwareVersion = "";
            zkemDevice.GetFirmwareVersion(machineNumber, ref firmwareVersion);
            return firmwareVersion;
        }
        
        // Get SDK version
        public string GetSDKVersion()
        {
            string sdkVersion = "";
            zkemDevice.GetSDKVersion(ref sdkVersion);
            return sdkVersion;
        }
        
        // Get device time
        public DateTime GetDeviceTime()
        {
            int year = 0, month = 0, day = 0;
            int hour = 0, minute = 0, second = 0;
            
            if (zkemDevice.GetDeviceTime(machineNumber, 
                ref year, ref month, ref day, 
                ref hour, ref minute, ref second))
            {
                return new DateTime(year, month, day, hour, minute, second);
            }
            
            return DateTime.MinValue;
        }
        
        // Set device time
        public bool SetDeviceTime(DateTime time)
        {
            return zkemDevice.SetDeviceTime2(machineNumber, 
                time.Year, time.Month, time.Day, 
                time.Hour, time.Minute, time.Second);
        }
        
        // Get platform information
        public string GetPlatform()
        {
            string platform = "";
            zkemDevice.GetPlatform(machineNumber, ref platform);
            return platform;
        }
        
        // Get device status
        public DeviceStatus GetDeviceStatus()
        {
            int adminCount = 0, userCount = 0, fpCount = 0, recordCount = 0;
            int dummy1 = 0, dummy2 = 0, dummy3 = 0;
            
            if (zkemDevice.GetDeviceStatus(machineNumber, 
                1, ref adminCount, // Admin count
                2, ref userCount,   // User count
                21, ref fpCount,    // Fingerprint count
                8, ref recordCount, // Record count
                22, ref dummy1,
                23, ref dummy2,
                24, ref dummy3))
            {
                return new DeviceStatus
                {
                    AdminCount = adminCount,
                    UserCount = userCount,
                    FingerprintCount = fpCount,
                    RecordCount = recordCount
                };
            }
            
            return null;
        }
    }
    
    public class DeviceStatus
    {
        public int AdminCount { get; set; }
        public int UserCount { get; set; }
        public int FingerprintCount { get; set; }
        public int RecordCount { get; set; }
    }
}
```

### Example 6: ASP.NET Core Integration (Dependency Injection)

```csharp
// Startup.cs or Program.cs (.NET 6+)
using Microsoft.Extensions.DependencyInjection;
using zkemkeeper;

namespace AttWeb
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            // Register SDK as singleton
            services.AddSingleton<CZKEMClass>(provider => new CZKEMClass());
            
            // Register services
            services.AddScoped<BiometricDeviceService>();
            services.AddScoped<AttendanceService>();
            services.AddScoped<UserManagementService>();
            services.AddScoped<DeviceInfoService>();
            services.AddScoped<RealtimeEventService>();
            
            // Other services...
            services.AddControllers();
        }
    }
}
```

```csharp
// Controller example
using Microsoft.AspNetCore.Mvc;
using AttWeb.Services;
using AttWeb.Models;
using System.Collections.Generic;

namespace AttWeb.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class AttendanceController : ControllerBase
    {
        private readonly BiometricDeviceService _deviceService;
        private readonly AttendanceService _attendanceService;
        
        public AttendanceController(
            BiometricDeviceService deviceService,
            AttendanceService attendanceService)
        {
            _deviceService = deviceService;
            _attendanceService = attendanceService;
        }
        
        [HttpGet("logs")]
        public ActionResult<List<AttendanceLog>> GetLogs()
        {
            var logs = _attendanceService.GetAllAttendanceLogs();
            return Ok(logs);
        }
        
        [HttpPost("connect")]
        public ActionResult Connect([FromBody] ConnectionRequest request)
        {
            bool connected = _deviceService.ConnectToDevice(
                request.IpAddress, 
                request.Port);
                
            if (connected)
                return Ok(new { message = "Connected successfully" });
            else
                return BadRequest(new { message = _deviceService.GetLastError() });
        }
    }
    
    public class ConnectionRequest
    {
        public string IpAddress { get; set; }
        public int Port { get; set; } = 4370;
    }
}
```

## Best Practices

### 1. Connection Management

```csharp
// Always use try-finally for connection management
public void ProcessAttendance(string ipAddress)
{
    var device = new CZKEMClass();
    
    try
    {
        if (device.Connect_Net(ipAddress, 4370))
        {
            // Process data
            var logs = GetLogs(device);
            SaveToDatabase(logs);
        }
    }
    finally
    {
        device.Disconnect();
        // COM object cleanup
        System.Runtime.InteropServices.Marshal.ReleaseComObject(device);
        device = null;
        GC.Collect();
        GC.WaitForPendingFinalizers();
    }
}
```

### 2. Error Handling

```csharp
public bool SafeDeviceOperation(Action operation)
{
    try
    {
        operation();
        return true;
    }
    catch (System.Runtime.InteropServices.COMException comEx)
    {
        // COM-specific errors
        _logger.LogError($"COM Error: {comEx.ErrorCode} - {comEx.Message}");
        return false;
    }
    catch (Exception ex)
    {
        _logger.LogError($"General Error: {ex.Message}");
        return false;
    }
}
```

### 3. Device State Management

```csharp
// Always disable device before bulk operations
public void BulkOperation()
{
    try
    {
        zkemDevice.EnableDevice(machineNumber, false);
        
        // Perform bulk operations
        // This speeds up data transfer significantly
        
        zkemDevice.EnableDevice(machineNumber, true);
    }
    catch
    {
        // Always re-enable device in case of error
        zkemDevice.EnableDevice(machineNumber, true);
        throw;
    }
}
```

### 4. Async Operations (for ASP.NET)

```csharp
using System.Threading.Tasks;

public class AsyncAttendanceService
{
    public async Task<List<AttendanceLog>> GetLogsAsync(string ipAddress)
    {
        return await Task.Run(() =>
        {
            var device = new CZKEMClass();
            
            try
            {
                if (device.Connect_Net(ipAddress, 4370))
                {
                    return GetAllAttendanceLogs(device);
                }
                return new List<AttendanceLog>();
            }
            finally
            {
                device.Disconnect();
            }
        });
    }
}
```

### 5. Configuration Management

```csharp
// appsettings.json
{
  "BiometricDevices": [
    {
      "Name": "Main Entrance",
      "IpAddress": "192.168.1.100",
      "Port": 4370,
      "MachineNumber": 1
    },
    {
      "Name": "Back Office",
      "IpAddress": "192.168.1.101",
      "Port": 4370,
      "MachineNumber": 1
    }
  ]
}
```

```csharp
// Configuration model
public class BiometricDeviceConfig
{
    public string Name { get; set; }
    public string IpAddress { get; set; }
    public int Port { get; set; } = 4370;
    public int MachineNumber { get; set; } = 1;
}

// Startup.cs
services.Configure<List<BiometricDeviceConfig>>(
    configuration.GetSection("BiometricDevices"));
```

## Advanced Topics

### Registration-Free COM (RegFree COM)

Avoid system-wide COM registration by using application manifests:

1. **Create app.manifest**:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity
    name="AttWeb"
    version="1.0.0.0"
    type="win32"/>
  
  <dependency>
    <dependentAssembly>
      <assemblyIdentity
        type="win32"
        name="zkemkeeper"
        version="1.0.0.0"/>
    </dependentAssembly>
  </dependency>
</assembly>
```

2. **Create zkemkeeper.manifest**:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <assemblyIdentity
    type="win32"
    name="zkemkeeper"
    version="1.0.0.0"/>
  
  <file name="zkemkeeper.dll">
    <comClass
      clsid="{00853A19-BD51-419B-9269-2DABE57EB61F}"
      threadingModel="Apartment"
      progid="zkemkeeper.CZKEM"/>
  </file>
</assembly>
```

### Multi-Device Management

```csharp
public class MultiDeviceManager
{
    private Dictionary<string, CZKEMClass> devices = new Dictionary<string, CZKEMClass>();
    
    public bool ConnectAll(List<BiometricDeviceConfig> configs)
    {
        foreach (var config in configs)
        {
            var device = new CZKEMClass();
            if (device.Connect_Net(config.IpAddress, config.Port))
            {
                devices[config.Name] = device;
            }
        }
        
        return devices.Count > 0;
    }
    
    public List<AttendanceLog> GetAllLogsFromAllDevices()
    {
        var allLogs = new List<AttendanceLog>();
        
        foreach (var kvp in devices)
        {
            var deviceLogs = GetLogsFromDevice(kvp.Value);
            // Add device identifier to logs
            deviceLogs.ForEach(log => log.DeviceName = kvp.Key);
            allLogs.AddRange(deviceLogs);
        }
        
        return allLogs;
    }
}
```

## Troubleshooting

### Common Issues and Solutions

#### 1. "Unable to cast COM object" Error

**Problem**: Type mismatch or wrong platform target
**Solution**: 
- Ensure project is set to x86 (32-bit)
- Verify the Interop assembly is correctly generated
- Try regenerating the Interop assembly

#### 2. "Class not registered" Error

**Problem**: zkemkeeper.dll not registered
**Solution**:
```batch
# Run as Administrator
regsvr32 "C:\Path\To\zkemkeeper.dll"
```

#### 3. Connection Timeout

**Problem**: Cannot connect to device
**Solution**:
- Verify device IP address and port
- Check network connectivity (ping the device)
- Ensure device is powered on
- Check firewall settings (port 4370 should be open)
- Try increasing connection timeout

#### 4. "Access Denied" When Reading Data

**Problem**: Device is locked or in use
**Solution**:
```csharp
// Disable device before operations
zkemDevice.EnableDevice(machineNumber, false);
// ... perform operations
zkemDevice.EnableDevice(machineNumber, true);
```

#### 5. Memory Leaks

**Problem**: COM objects not released
**Solution**:
```csharp
// Proper COM cleanup
System.Runtime.InteropServices.Marshal.ReleaseComObject(device);
device = null;
GC.Collect();
GC.WaitForPendingFinalizers();
```

#### 6. Real-Time Events Not Firing

**Problem**: Events not registered properly
**Solution**:
```csharp
// Register events before connecting
device.OnAttTransactionEx += Handler;
device.Connect_Net(ip, port);
device.RegEvent(machineNumber, 65535); // Register all events
```

### Error Codes

Common error codes from `GetLastError()`:

| Error Code | Description | Solution |
|-----------|-------------|----------|
| 0 | Success | No error |
| -1 | General error | Check connection and permissions |
| -2 | Invalid parameter | Verify input parameters |
| -5 | Data not found | Check if data exists on device |
| -10 | Transmission failed | Check network connection |
| -100 | Device is busy | Wait and retry |

## Performance Optimization

### 1. Bulk Data Reading

```csharp
// Faster: Read all data at once
zkemDevice.EnableDevice(machineNumber, false);
zkemDevice.ReadGeneralLogData(machineNumber);
// Process data
zkemDevice.EnableDevice(machineNumber, true);

// Slower: Read data one by one
```

### 2. Connection Pooling

```csharp
public class DeviceConnectionPool
{
    private static readonly ConcurrentBag<CZKEMClass> Pool = new ConcurrentBag<CZKEMClass>();
    
    public static CZKEMClass GetConnection(string ip, int port)
    {
        if (Pool.TryTake(out var device))
        {
            return device;
        }
        
        device = new CZKEMClass();
        device.Connect_Net(ip, port);
        return device;
    }
    
    public static void ReturnConnection(CZKEMClass device)
    {
        Pool.Add(device);
    }
}
```

### 3. Caching

```csharp
public class CachedDeviceService
{
    private IMemoryCache _cache;
    
    public List<BiometricUser> GetUsers(bool forceRefresh = false)
    {
        string cacheKey = "device_users";
        
        if (!forceRefresh && _cache.TryGetValue(cacheKey, out List<BiometricUser> users))
        {
            return users;
        }
        
        users = FetchUsersFromDevice();
        
        _cache.Set(cacheKey, users, TimeSpan.FromMinutes(10));
        
        return users;
    }
}
```

## Testing

### Unit Testing with Mock

```csharp
// Create interface for testability
public interface IBiometricDevice
{
    bool Connect(string ip, int port);
    List<AttendanceLog> GetLogs();
}

// Wrapper implementation
public class BiometricDeviceWrapper : IBiometricDevice
{
    private CZKEMClass _device;
    
    public bool Connect(string ip, int port)
    {
        _device = new CZKEMClass();
        return _device.Connect_Net(ip, port);
    }
    
    public List<AttendanceLog> GetLogs()
    {
        // Implementation
    }
}

// Mock for testing
public class MockBiometricDevice : IBiometricDevice
{
    public bool Connect(string ip, int port) => true;
    
    public List<AttendanceLog> GetLogs()
    {
        return new List<AttendanceLog>
        {
            new AttendanceLog { EnrollNumber = "1001", LogTime = DateTime.Now }
        };
    }
}

// Unit test
[TestMethod]
public void TestAttendanceProcessing()
{
    var mockDevice = new MockBiometricDevice();
    var service = new AttendanceService(mockDevice);
    
    var logs = service.ProcessLogs();
    
    Assert.AreEqual(1, logs.Count);
}
```

## Deployment Checklist

- [ ] Ensure all SDK DLLs are included in deployment package
- [ ] Register zkemkeeper.dll on target machine (or use RegFree COM)
- [ ] Set application to run as x86 (32-bit)
- [ ] Configure firewall to allow port 4370
- [ ] Test connection to all devices
- [ ] Implement proper error handling and logging
- [ ] Set up monitoring for device connectivity
- [ ] Document device IP addresses and configuration
- [ ] Create backup/restore procedures for device data
- [ ] Train support staff on troubleshooting

## Resources

- [Official ZKTeco Website](https://www.zkteco.com)
- SDK Documentation: Check vendor documentation (if available)
- COM Interop Documentation: [Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/cominterop)

## Support

For issues specific to:
- **SDK**: Contact ZKTeco support
- **Integration**: Refer to this guide or create an issue in the repository
- **Devices**: Check device manual or contact hardware vendor

---

**Last Updated**: 2024
**SDK Version**: 6.2.4.11
**Compatible .NET Versions**: .NET Framework 4.6.1+, .NET 6+ (Windows only)
