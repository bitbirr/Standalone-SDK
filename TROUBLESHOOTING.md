# Troubleshooting Guide

## Table of Contents

1. [Installation Issues](#installation-issues)
2. [Connection Problems](#connection-problems)
3. [COM Interop Issues](#com-interop-issues)
4. [Data Reading Issues](#data-reading-issues)
5. [Performance Issues](#performance-issues)
6. [Deployment Issues](#deployment-issues)
7. [Common Error Codes](#common-error-codes)
8. [Advanced Diagnostics](#advanced-diagnostics)

---

## Installation Issues

### ❌ "The system cannot execute the specified program"

**Symptoms:** Cannot register zkemkeeper.dll

**Causes:**
- Missing Visual C++ runtime
- Corrupted DLL file
- Wrong architecture (64-bit vs 32-bit)

**Solutions:**

1. **Install Visual C++ Redistributable:**
   ```
   Download and install:
   - Visual C++ 2010 Redistributable (x86)
   - Visual C++ 2015-2022 Redistributable (x86)
   ```

2. **Verify DLL integrity:**
   ```batch
   # Check file
   dir zkemkeeper.dll
   # Should be ~652 KB
   ```

3. **Run registration as Administrator:**
   ```batch
   # Right-click Command Prompt -> Run as Administrator
   cd "path\to\dll"
   regsvr32 zkemkeeper.dll
   ```

---

### ❌ "DllRegisterServer entry point was not found"

**Symptoms:** Registration fails with entry point error

**Solution:**
The DLL may be a dependency, not a COM server. This is normal for:
- commpro.dll
- comms.dll
- tcpcomm.dll
- usbcomm.dll
- etc.

**Only register:** `zkemkeeper.dll`

---

### ❌ "Access Denied" during installation

**Symptoms:** Cannot copy DLLs to System32

**Solution:**
```batch
# Option 1: Run as Administrator
Right-click Auto-install_sdk.bat -> Run as Administrator

# Option 2: Use local deployment (recommended)
# Copy DLLs to your application's bin folder instead
```

---

## Connection Problems

### ❌ Cannot Connect via TCP/IP

**Symptoms:** `Connect_Net()` returns false

**Diagnostic Steps:**

1. **Verify Network Connectivity:**
   ```batch
   ping 192.168.1.100
   # Replace with your device IP
   ```

2. **Check Device IP Address:**
   - On device: Menu → Comm → IP Address
   - Verify subnet matches your PC

3. **Test Port Connectivity:**
   ```batch
   telnet 192.168.1.100 4370
   # If command not found, install: dism /online /Enable-Feature /FeatureName:TelnetClient
   ```

4. **Check Firewall:**
   ```powershell
   # Allow port 4370 in Windows Firewall
   New-NetFirewallRule -DisplayName "ZKTeco Device" -Direction Inbound -LocalPort 4370 -Protocol TCP -Action Allow
   ```

5. **Verify Device Settings:**
   - Default port: 4370
   - Default password: 0
   - Check if device is in sleep mode

**Code to Diagnose:**
```csharp
CZKEMClass device = new CZKEMClass();

// Try connection with timeout
if (!device.Connect_Net("192.168.1.100", 4370))
{
    int errorCode = 0;
    device.GetLastError(ref errorCode);
    Console.WriteLine($"Connection failed. Error code: {errorCode}");
    
    // Error codes:
    // -10: Network error
    // -1: General error
}
```

---

### ❌ Cannot Connect via USB

**Symptoms:** `Connect_USB()` returns false

**Solutions:**

1. **Check USB Connection:**
   - Try different USB ports
   - Use USB 2.0 port (some devices don't support USB 3.0)
   - Check USB cable

2. **Install USB Drivers:**
   - Check Device Manager for unknown devices
   - Install manufacturer's USB drivers
   - Some devices show as "ZK USB Serial Port"

3. **Check Device Mode:**
   - Device may need to be in USB communication mode
   - Check device menu settings

4. **Try Different Machine Number:**
   ```csharp
   // Try machine numbers 1-255
   for (int i = 1; i <= 255; i++)
   {
       if (device.Connect_USB(i))
       {
           Console.WriteLine($"Connected with machine number: {i}");
           break;
       }
   }
   ```

---

### ❌ Cannot Connect via RS232/RS485

**Symptoms:** `Connect_Com()` returns false

**Solutions:**

1. **Verify COM Port:**
   ```batch
   # List available COM ports
   mode
   ```

2. **Check Settings:**
   ```csharp
   // Common configurations
   device.Connect_Com(1, 1, 115200);  // COM1, Machine 1, 115200 baud
   device.Connect_Com(1, 1, 57600);   // Try lower baud rate
   device.Connect_Com(1, 1, 38400);
   ```

3. **Check Physical Connection:**
   - Verify TX/RX connections
   - Check ground wire
   - Verify RS232 vs RS485 mode

---

### ❌ "Device is Offline" or Random Disconnections

**Symptoms:** Device disconnects randomly

**Solutions:**

1. **Network Issues:**
   - Check cable quality
   - Reduce distance to switch/router
   - Check for IP conflicts
   - Assign static IP to device

2. **Power Issues:**
   - Check power supply
   - Use UPS for stable power
   - Check voltage requirements

3. **Implement Reconnection Logic:**
   ```csharp
   public bool ConnectWithRetry(string ip, int port, int maxRetries = 3)
   {
       for (int i = 0; i < maxRetries; i++)
       {
           if (device.Connect_Net(ip, port))
               return true;
           
           Thread.Sleep(2000);  // Wait 2 seconds
       }
       return false;
   }
   ```

---

## COM Interop Issues

### ❌ "Unable to cast COM object"

**Symptoms:** Runtime error when calling methods

**Causes:**
- Platform mismatch (x64 vs x86)
- Incorrect Interop assembly
- COM threading issues

**Solutions:**

1. **Set Platform to x86:**
   ```
   Project Properties → Build → Platform target: x86
   ```

2. **Regenerate Interop Assembly:**
   ```batch
   # Developer Command Prompt
   tlbimp zkemkeeper.dll /out:Interop.zkemkeeper.dll
   ```

3. **Check COM Threading:**
   ```csharp
   // For WinForms/WPF
   [STAThread]
   static void Main()
   {
       Application.Run(new MainForm());
   }
   ```

---

### ❌ "Class not registered (Exception from HRESULT: 0x80040154)"

**Symptoms:** Cannot create CZKEMClass instance

**Solutions:**

1. **Register DLL:**
   ```batch
   regsvr32 "C:\path\to\zkemkeeper.dll"
   ```

2. **Check Registry:**
   ```
   HKEY_CLASSES_ROOT\zkemkeeper.CZKEM
   # Should exist after registration
   ```

3. **Use RegFree COM:**
   - Create manifest files (see Integration Guide)
   - Avoid system-wide registration

---

### ❌ "Type library not registered"

**Symptoms:** Cannot add COM reference in Visual Studio

**Solution:**
```batch
# Re-register with type library
regsvr32 zkemkeeper.dll

# If still fails, manually create Interop
tlbimp zkemkeeper.dll /out:Interop.zkemkeeper.dll
# Then add Interop.zkemkeeper.dll as regular reference
```

---

## Data Reading Issues

### ❌ No Data Returned from GetAllUserInfo

**Symptoms:** Loop exits immediately, no users retrieved

**Solution:**

1. **Call ReadAllUserID first:**
   ```csharp
   device.EnableDevice(machineNumber, false);
   
   // MUST call this first!
   if (device.ReadAllUserID(machineNumber))
   {
       string enrollNumber, name, password;
       int privilege;
       bool enabled;
       
       while (device.SSR_GetAllUserInfo(machineNumber,
           out enrollNumber, out name, out password,
           out privilege, out enabled))
       {
           Console.WriteLine($"User: {name}");
       }
   }
   
   device.EnableDevice(machineNumber, true);
   ```

2. **Check if device has users:**
   ```csharp
   int userCount = 0;
   device.GetDeviceStatus(machineNumber, 2, ref userCount);
   Console.WriteLine($"User count: {userCount}");
   ```

---

### ❌ No Attendance Logs Retrieved

**Symptoms:** ReadGeneralLogData returns true but no logs

**Solution:**

1. **Verify log count:**
   ```csharp
   int logCount = 0;
   device.GetDeviceStatus(machineNumber, 8, ref logCount);
   Console.WriteLine($"Log count: {logCount}");
   ```

2. **Use correct method sequence:**
   ```csharp
   device.EnableDevice(machineNumber, false);
   
   if (device.ReadGeneralLogData(machineNumber))
   {
       string enrollNumber;
       int verifyMode, inOutMode, year, month, day, hour, minute, second;
       int workCode = 0;
       
       while (device.SSR_GetGeneralLogData(machineNumber,
           out enrollNumber, out verifyMode, out inOutMode,
           out year, out month, out day,
           out hour, out minute, out second, ref workCode))
       {
           Console.WriteLine($"{enrollNumber}: {year}-{month}-{day}");
       }
   }
   
   device.EnableDevice(machineNumber, true);
   ```

---

### ❌ "Device is Busy" Error

**Symptoms:** Error code -100 when reading data

**Solutions:**

1. **Disable Device:**
   ```csharp
   device.EnableDevice(machineNumber, false);
   // Perform operations
   device.EnableDevice(machineNumber, true);
   ```

2. **Wait for Device:**
   ```csharp
   Thread.Sleep(1000);  // Wait 1 second
   device.ReadGeneralLogData(machineNumber);
   ```

3. **Check if Another Application Connected:**
   - Only one application can connect at a time
   - Close other software using the device

---

### ❌ Partial Data Retrieved

**Symptoms:** Only some users or logs retrieved

**Solutions:**

1. **Increase Buffer Size:**
   - SDK has internal buffers
   - Retrieve data in smaller batches

2. **Use Time-Range Queries:**
   ```csharp
   device.GetAttLogByTimeEx(machineNumber, "",
       2024, 1, 1,    // Start: Jan 1, 2024
       2024, 12, 31); // End: Dec 31, 2024
   ```

---

## Performance Issues

### ❌ Very Slow Data Retrieval

**Symptoms:** Takes several minutes to read logs

**Solutions:**

1. **Always Disable Device:**
   ```csharp
   // SLOW (device enabled):
   device.ReadGeneralLogData(machineNumber);
   
   // FAST (device disabled):
   device.EnableDevice(machineNumber, false);
   device.ReadGeneralLogData(machineNumber);
   device.EnableDevice(machineNumber, true);
   ```

2. **Use Efficient Methods:**
   ```csharp
   // Faster for bulk operations
   device.BeginBatchUpdate(machineNumber);
   // Make changes
   device.BatchUpdate(machineNumber);
   ```

3. **Network Optimization:**
   - Use wired connection instead of WiFi
   - Reduce network latency
   - Check network congestion

---

### ❌ High Memory Usage

**Symptoms:** Application memory grows continuously

**Solution:**

1. **Release COM Objects:**
   ```csharp
   void CleanupDevice(CZKEMClass device)
   {
       if (device != null)
       {
           try { device.Disconnect(); } catch { }
           
           System.Runtime.InteropServices.Marshal.ReleaseComObject(device);
           device = null;
       }
       
       GC.Collect();
       GC.WaitForPendingFinalizers();
       GC.Collect();
   }
   ```

2. **Process Data in Batches:**
   ```csharp
   // Don't store all logs in memory
   while (device.SSR_GetGeneralLogData(...))
   {
       SaveToDatabase(log);  // Save immediately
   }
   ```

---

## Deployment Issues

### ❌ "Works on Dev Machine, Fails on Production"

**Common Causes:**

1. **Missing Visual C++ Runtime:**
   ```
   Install on production:
   - vcredist_x86.exe (2010)
   - vcredist_x86.exe (2015-2022)
   ```

2. **DLL Not Registered:**
   ```batch
   # On production server
   regsvr32 "C:\Program Files (x86)\YourApp\zkemkeeper.dll"
   ```

3. **Platform Mismatch:**
   ```
   Ensure production app is compiled as x86
   ```

4. **Different .NET Framework:**
   ```
   Check .NET Framework version matches
   ```

---

### ❌ "Access Denied" in Production

**Solution:**

1. **Run with Elevated Privileges:**
   - Add manifest to require administrator
   - Or install as Windows Service

2. **Use Local Deployment:**
   - Don't install to System32
   - Use application directory

---

### ❌ Installer Fails to Register COM

**Solution:**

Create setup with proper registration:

```xml
<!-- WiX installer example -->
<Component Id="ZKemkeeper" Guid="YOUR-GUID">
    <File Id="zkemkeeper.dll" 
          Source="zkemkeeper.dll" 
          KeyPath="yes"
          SelfRegCost="1" />
</Component>
```

---

## Common Error Codes

| Code | Description | Solution |
|------|-------------|----------|
| 0 | Success | No error |
| -1 | General error | Check connection and parameters |
| -2 | Invalid parameter | Verify method arguments |
| -3 | Not supported | Feature not available on this device |
| -4 | Buffer too small | Increase buffer or use different method |
| -5 | Data not found | Verify data exists on device |
| -6 | Insufficient memory | Free up memory or restart device |
| -7 | Timeout | Increase timeout or check connection |
| -10 | Transmission error | Network issue, check cable/wifi |
| -100 | Device busy | Disable device or wait |
| -101 | Invalid handle | Reconnect to device |
| -102 | SDK not initialized | Create new CZKEMClass instance |

---

## Advanced Diagnostics

### Network Packet Capture

**Tool:** Wireshark

1. **Filter for device traffic:**
   ```
   ip.addr == 192.168.1.100 && tcp.port == 4370
   ```

2. **Look for:**
   - TCP handshake completion
   - Data packets
   - Retransmissions (indicates network issues)

---

### COM Object Inspection

**Tool:** OleView.exe (Windows SDK)

1. Find zkemkeeper.CZKEM
2. Check interfaces and methods
3. Verify CLSID registration

---

### Event Viewer Logs

Check Windows Event Viewer:
```
Windows Logs → Application
Look for .NET Runtime errors or COM+ errors
```

---

### Debugging Connection Issues

```csharp
public class DiagnosticHelper
{
    public static void DiagnoseConnection(string ip, int port)
    {
        Console.WriteLine("=== Connection Diagnostics ===");
        
        // 1. Ping test
        try
        {
            var ping = new System.Net.NetworkInformation.Ping();
            var reply = ping.Send(ip, 3000);
            Console.WriteLine($"Ping: {reply.Status}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Ping failed: {ex.Message}");
        }
        
        // 2. Port test
        try
        {
            using (var client = new System.Net.Sockets.TcpClient())
            {
                var result = client.BeginConnect(ip, port, null, null);
                var success = result.AsyncWaitHandle.WaitOne(3000);
                
                if (success)
                {
                    client.EndConnect(result);
                    Console.WriteLine("Port is open");
                }
                else
                {
                    Console.WriteLine("Port is closed or timeout");
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Port test failed: {ex.Message}");
        }
        
        // 3. SDK connection test
        var device = new CZKEMClass();
        if (device.Connect_Net(ip, port))
        {
            Console.WriteLine("SDK connected successfully");
            
            string serial = "";
            device.GetSerialNumber(1, out serial);
            Console.WriteLine($"Device Serial: {serial}");
            
            device.Disconnect();
        }
        else
        {
            int errorCode = 0;
            device.GetLastError(ref errorCode);
            Console.WriteLine($"SDK connection failed. Error: {errorCode}");
        }
    }
}
```

---

### Log Everything

```csharp
public class LoggingWrapper
{
    private CZKEMClass device;
    private ILogger logger;
    
    public bool Connect_Net(string ip, int port)
    {
        logger.LogInformation($"Attempting connection to {ip}:{port}");
        
        var sw = System.Diagnostics.Stopwatch.StartNew();
        bool result = device.Connect_Net(ip, port);
        sw.Stop();
        
        if (result)
        {
            logger.LogInformation($"Connected in {sw.ElapsedMilliseconds}ms");
        }
        else
        {
            int errorCode = 0;
            device.GetLastError(ref errorCode);
            logger.LogError($"Connection failed. Error: {errorCode}. Time: {sw.ElapsedMilliseconds}ms");
        }
        
        return result;
    }
}
```

---

## Getting Help

### Before Asking for Help

Collect this information:

1. **Environment:**
   - Windows version
   - .NET version
   - SDK version
   - Visual Studio version

2. **Device Info:**
   - Device model
   - Firmware version
   - Connection type (TCP/IP/USB/RS232)

3. **Error Details:**
   - Exact error message
   - Error code from GetLastError()
   - Stack trace

4. **What You've Tried:**
   - List troubleshooting steps attempted

### Support Channels

- 📧 GitHub Issues: [Repository Issues](../../issues)
- 📖 Documentation: See guides in this repository
- 🔧 ZKTeco Support: Contact hardware vendor

---

## Checklist for Common Issues

### Connection Issues Checklist

- [ ] Device is powered on
- [ ] Network cable is connected (for TCP/IP)
- [ ] IP address is correct
- [ ] Port 4370 is accessible
- [ ] Firewall allows connection
- [ ] Only one application connected to device
- [ ] Device is not in sleep mode
- [ ] zkemkeeper.dll is registered
- [ ] Application is compiled as x86
- [ ] Visual C++ runtime is installed

### Data Reading Issues Checklist

- [ ] Called ReadAllUserID before SSR_GetAllUserInfo
- [ ] Called ReadGeneralLogData before SSR_GetGeneralLogData
- [ ] Disabled device before bulk operations
- [ ] Re-enabled device after operations
- [ ] Checked device status for data count
- [ ] Used correct machine number

### Deployment Issues Checklist

- [ ] All DLLs included in deployment
- [ ] zkemkeeper.dll registered on target machine
- [ ] Visual C++ runtime installed on target
- [ ] Application runs with appropriate privileges
- [ ] Platform target set to x86
- [ ] .NET Framework version matches

---

**Last Updated**: 2024  
**SDK Version**: 6.2.4.11
