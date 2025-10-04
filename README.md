# Standalone-SDK

Standalone SDK is a middleware solution for building professional desktop HR/Payroll/Attendance software. It has been running with ZKTeco time attendance units and access control terminals for over 15 years. With this rock-steady SDK architecture, you can build any time-related software you want. The key benefit is that you don't need to worry about hardware compatibility—it works seamlessly with all ZKTeco devices.

## 📚 Documentation

### Getting Started
- **[⭐ AttWeb Integration Summary](ATTWEB_INTEGRATION.md)** - **Quick integration guide specifically for AttWeb project**
- **[Quick Start Guide](QUICK_START.md)** - Get up and running in 5 minutes
- **[.NET Integration Guide](DOTNET_INTEGRATION_GUIDE.md)** - Complete guide for integrating with .NET projects
- **[SDK Review](SDK_REVIEW.md)** - Comprehensive SDK analysis and architecture overview

### Reference
- **[API Reference](API_REFERENCE.md)** - Complete API documentation with examples
- **[Troubleshooting Guide](TROUBLESHOOTING.md)** - Solutions to common issues

## 🚀 Quick Start

### For .NET Developers (AttWeb Integration)

1. **Install the SDK:**
   ```batch
   cd "Communication Protocol SDK(32Bit Ver6.2.4.11)"
   Auto-install_sdk.bat  # Run as Administrator
   ```

2. **Add COM Reference in Visual Studio:**
   - Right-click project → Add → Reference
   - COM tab → Select "zkemkeeper 1.0 Type Library"

3. **Set Platform to x86:**
   - Project Properties → Build → Platform target: **x86**

4. **Basic Connection Code:**
   ```csharp
   using zkemkeeper;
   
   CZKEMClass device = new CZKEMClass();
   if (device.Connect_Net("192.168.1.100", 4370))
   {
       Console.WriteLine("Connected!");
       // Your code here
       device.Disconnect();
   }
   ```

👉 **See the [Quick Start Guide](QUICK_START.md) for a complete walkthrough**

## 📦 What's Included

### SDK Components

- **zkemkeeper.dll** - Main COM interface for device communication
- **zkemsdk.dll** - Core SDK library
- **Communication DLLs:**
  - tcpcomm.dll - TCP/IP communication
  - usbcomm.dll - USB communication  
  - rscomm.dll - RS232/RS485 communication
  - commpro.dll, comms.dll, rscagent.dll - Protocol handlers

### Installation Scripts

- **Auto-install_sdk.bat** - Automated installation (requires Admin)
- **Auto-Uninstall_sdk.bat** - Automated removal

## ✨ Key Features

- 🔌 **Multiple Connection Types**: TCP/IP, USB, RS232/RS485
- 👥 **User Management**: Add, update, delete users
- 📊 **Attendance Tracking**: Real-time and historical logs
- 🔐 **Access Control**: Door management, unlock/lock
- 📸 **Biometric Data**: Fingerprint, face, card support
- 🔄 **Real-time Events**: Live attendance notifications
- 📱 **Multi-device Support**: Connect to multiple devices

## 🛠️ Supported Technologies

### Platforms
- Windows 7/8/10/11
- Windows Server 2003/2008/2012/2016/2019/2022

### Development
- .NET Framework 4.6.1+
- .NET Core 3.1+ / .NET 5+ (Windows only)
- Visual Basic 6.0
- C/C++ (with COM support)
- Any COM-compatible language

### Devices
- All ZKTeco time attendance terminals
- All ZKTeco access control devices
- Biometric readers (fingerprint, face recognition)

## 📖 Integration Guides

### .NET / AttWeb Integration

The [.NET Integration Guide](DOTNET_INTEGRATION_GUIDE.md) covers:

- ✅ COM Interop configuration
- ✅ Connection patterns (TCP/IP, USB, RS232)
- ✅ User management examples
- ✅ Attendance log retrieval
- ✅ Real-time event handling
- ✅ ASP.NET Core integration
- ✅ Best practices and patterns
- ✅ Performance optimization
- ✅ Deployment strategies

### Code Examples

**Connect to Device:**
```csharp
var device = new CZKEMClass();
device.Connect_Net("192.168.1.100", 4370);
```

**Get Attendance Logs:**
```csharp
device.ReadGeneralLogData(machineNumber);
while (device.SSR_GetGeneralLogData(machineNumber, 
    out string enrollNumber, out int verifyMode, out int inOutMode,
    out int year, out int month, out int day,
    out int hour, out int minute, out int second, ref int workCode))
{
    var logTime = new DateTime(year, month, day, hour, minute, second);
    Console.WriteLine($"{enrollNumber}: {logTime}");
}
```

**Real-time Events:**
```csharp
device.OnAttTransactionEx += (enrollNumber, isInValid, attState, 
    verifyMethod, year, month, day, hour, minute, second, workCode) =>
{
    Console.WriteLine($"Real-time: {enrollNumber} at {hour}:{minute}");
};
device.RegEvent(machineNumber, 65535);
```

## 🔧 Troubleshooting

Having issues? Check the [Troubleshooting Guide](TROUBLESHOOTING.md) for solutions to:

- Connection problems (TCP/IP, USB, RS232)
- COM Interop issues
- Data reading errors
- Performance optimization
- Deployment issues

**Quick Fixes:**

- **"Class not registered"** → Run `regsvr32 zkemkeeper.dll` as Admin
- **"Cannot connect"** → Check IP, port 4370, and firewall
- **"Platform mismatch"** → Set project to x86 platform
- **No data returned** → Call `ReadAllUserID` before `SSR_GetAllUserInfo`

## 📋 Requirements

### For Development
- Visual Studio 2019 or later
- .NET Framework 4.6.1+ or .NET 6+
- Windows OS
- Administrator privileges (for SDK installation)

### For Runtime
- Windows 7 or later
- Visual C++ Runtime (2010, 2015-2022)
- Network connection to devices (for TCP/IP)

## 🏗️ Architecture

The SDK uses a COM-based architecture:

```
Your Application
      ↓
  zkemkeeper.dll (COM Interface)
      ↓
  zkemsdk.dll (Core SDK)
      ↓
  Communication Layer (tcpcomm/usbcomm/rscomm)
      ↓
  ZKTeco Device
```

See [SDK Review](SDK_REVIEW.md) for detailed architecture analysis.

## 📝 API Quick Reference

### Connection
- `Connect_Net(ip, port)` - TCP/IP connection
- `Connect_USB(machineNumber)` - USB connection
- `Connect_Com(port, machineNumber, baudRate)` - Serial connection
- `Disconnect()` - Close connection

### User Management
- `ReadAllUserID(machineNumber)` - Load users
- `SSR_GetAllUserInfo(...)` - Get user details
- `SSR_SetUserInfo(...)` - Add/update user
- `SSR_DeleteEnrollData(...)` - Delete user

### Attendance Logs
- `ReadGeneralLogData(machineNumber)` - Load logs
- `SSR_GetGeneralLogData(...)` - Get log details
- `ClearGLog(machineNumber)` - Clear logs

### Device Info
- `GetSerialNumber(...)` - Get device serial
- `GetFirmwareVersion(...)` - Get firmware version
- `GetDeviceStatus(...)` - Get device status
- `SetDeviceTime2(...)` - Set device time

👉 **See [API Reference](API_REFERENCE.md) for complete documentation**

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues, feature requests, or pull requests.

### Areas for Contribution
- Additional code examples
- Language bindings (Python, Java, etc.)
- Improved documentation
- Bug fixes and enhancements

## 📄 License

Please refer to ZKTeco licensing terms for SDK usage.

## 🔗 Resources

- [ZKTeco Official Website](https://www.zkteco.com)
- [⭐ AttWeb Integration Summary](ATTWEB_INTEGRATION.md) - **Start here for AttWeb project**
- [Quick Start Guide](QUICK_START.md)
- [.NET Integration Guide](DOTNET_INTEGRATION_GUIDE.md)
- [API Reference](API_REFERENCE.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)
- [SDK Review](SDK_REVIEW.md)
- [Device Setup Guide](DEVICE_SETUP_GUIDE.md)

## 💡 Support

- **Documentation**: Check the guides in this repository
- **Issues**: [GitHub Issues](../../issues)
- **Hardware Support**: Contact ZKTeco

---

**SDK Version**: 6.2.4.11 (32-bit)  
**Last Updated**: 2024

⭐ **Star this repository** if you find it helpful!
