# Quick Start Guide - AttWeb Integration

## 5-Minute Setup

This guide will get you connected to a ZKTeco device from your .NET AttWeb project in 5 minutes.

### Prerequisites
- Visual Studio 2019+
- .NET Framework 4.6.1+ or .NET 6+
- ZKTeco device connected to network
- Administrator rights (for one-time setup)

## Step 1: Install SDK (2 minutes)

### Option A: Quick Install (Windows)
```batch
# Open Command Prompt as Administrator
cd "Communication Protocol SDK(32Bit Ver6.2.4.11)"
Auto-install_sdk.bat
```

### Option B: Manual Install
1. Copy all DLLs from `Communication Protocol SDK(32Bit Ver6.2.4.11)/sdk/` to your project's `bin` folder
2. Open Command Prompt as Administrator in the bin folder
3. Run: `regsvr32 zkemkeeper.dll`

## Step 2: Configure Project (1 minute)

1. **Set Platform to x86**:
   - Right-click project → Properties → Build
   - Platform target: **x86**

2. **Add COM Reference**:
   - Right-click project → Add → Reference
   - COM tab → Find **"zkemkeeper 1.0 Type Library"** → OK

## Step 3: Write Connection Code (2 minutes)

Create a new file `BiometricService.cs`:

```csharp
using zkemkeeper;
using System;

public class BiometricService
{
    private CZKEMClass device = new CZKEMClass();
    
    // Connect to device
    public bool Connect(string ipAddress, int port = 4370)
    {
        return device.Connect_Net(ipAddress, port);
    }
    
    // Get attendance logs
    public void GetLogs()
    {
        int machineNumber = 1;
        device.EnableDevice(machineNumber, false);
        
        if (device.ReadGeneralLogData(machineNumber))
        {
            string enrollNumber = "";
            int verifyMode = 0, inOutMode = 0;
            int year = 0, month = 0, day = 0;
            int hour = 0, minute = 0, second = 0;
            int workCode = 0;
            
            while (device.SSR_GetGeneralLogData(machineNumber, 
                out enrollNumber, out verifyMode, out inOutMode,
                out year, out month, out day, 
                out hour, out minute, out second, ref workCode))
            {
                Console.WriteLine($"{enrollNumber} - {year}-{month:D2}-{day:D2} {hour:D2}:{minute:D2}:{second:D2}");
            }
        }
        
        device.EnableDevice(machineNumber, true);
    }
    
    // Disconnect
    public void Disconnect()
    {
        device.Disconnect();
    }
}
```

## Step 4: Test It

```csharp
class Program
{
    static void Main(string[] args)
    {
        var service = new BiometricService();
        
        // Replace with your device's IP address
        if (service.Connect("192.168.1.100"))
        {
            Console.WriteLine("Connected!");
            service.GetLogs();
            service.Disconnect();
        }
        else
        {
            Console.WriteLine("Connection failed!");
        }
    }
}
```

## Common Connection Issues

### Can't Find Device IP?
```batch
# Find device on network (Windows)
arp -a
# Or use device's keypad to check IP (usually Menu → Comm → IP)
```

### Connection Fails?
1. **Check Network**: `ping 192.168.1.100` (replace with your device IP)
2. **Check Port**: Default is 4370
3. **Check Firewall**: Allow port 4370
4. **Check Device**: Ensure it's powered on and on same network

### "Class not registered" Error?
```batch
# Re-register DLL as Administrator
regsvr32 zkemkeeper.dll
```

### "Platform mismatch" Error?
- Set project to **x86** (not Any CPU or x64)

## Next Steps

1. Read the full [.NET Integration Guide](DOTNET_INTEGRATION_GUIDE.md)
2. Review [SDK Documentation](SDK_REVIEW.md)
3. Check [API Reference](API_REFERENCE.md)

## Support

- 📧 Repository Issues: [GitHub Issues](../../issues)
- 📖 Full Documentation: See integration guide
- 🔧 Troubleshooting: See troubleshooting section in integration guide

---

**Success?** Star this repository and share your experience!
