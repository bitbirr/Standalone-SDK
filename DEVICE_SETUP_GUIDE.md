# Device Setup and Compatibility Guide

## Table of Contents

1. [Supported Devices](#supported-devices)
2. [Device Setup](#device-setup)
3. [Network Configuration](#network-configuration)
4. [Device Settings](#device-settings)
5. [Multiple Device Setup](#multiple-device-setup)
6. [Best Practices](#best-practices)

---

## Supported Devices

The ZKTeco Standalone SDK supports a wide range of devices:

### Time Attendance Terminals

#### Fingerprint Devices
- **K Series**: K14, K20, K28, K30, K40, K50, K60
- **F Series**: F2, F7, F18, F19, F21, F22
- **X Series**: X628, X638, X628-C, X628-BT
- **ZK Series**: ZK4500, ZKFinger, ZKFinger VX
- **Standalone**: S922, S30, S60, S80

#### Face Recognition Devices
- **SpeedFace Series**: SpeedFace-V1L, SpeedFace-V3L, SpeedFace-V4L, SpeedFace-V5L
- **ProFace Series**: ProFace, ProFace X, ProFace Plus
- **UltraFace Series**: UltraFace 402, UltraFace 502, UltraFace 602, UltraFace 800
- **iFace Series**: iFace302, iFace402, iFace502, iFace702, iFace800, iFace990

#### Palm Recognition
- **PalmID Series**: PalmID II

#### Multi-Biometric
- **iClock Series**: iClock260, iClock360, iClock380, iClock560, iClock660, iClock680, iClock700, iClock880
- **Fusion Series**: Fusion, Fusion Plus

### Access Control Terminals

- **InBio Series**: InBio160, InBio260, InBio460
- **C3 Series**: C3-100, C3-200, C3-400
- **ProCapture Series**: ProCapture-X
- **PFACE Series**: PFACE202, PFACE302
- **Security Controllers**: SC103, SC203, SC403, SC503

### Specialized Devices

- **Visitor Management**: VF300, VF680
- **Turnstile Controllers**: TS1000, TS2000
- **Elevator Controllers**: EC10

> **Note**: This SDK works with most ZKTeco devices. If your specific model is not listed, it's likely still compatible. Verify with your device manual or contact ZKTeco support.

---

## Device Setup

### Initial Device Configuration

#### 1. Power Up the Device

1. Connect power adapter to device
2. Wait for device to boot (typically 30-60 seconds)
3. Device should display main screen

#### 2. Access Device Menu

**Standard Access:**
- Press `M/OK` or `Menu` button on device
- Enter admin password (default is usually empty or `0`)

**Default Admin Credentials:**
- User ID: `1` or `admin`
- Password: (empty) or `0`

#### 3. Set Device Language (if needed)

```
Menu → Options → Language → English
```

---

## Network Configuration

### TCP/IP Setup (Most Common)

#### Method 1: Device Keypad Configuration

1. **Access Network Settings:**
   ```
   Menu → Comm → Ethernet (or TCP/IP)
   ```

2. **Configure IP Address:**
   - **IP Address**: `192.168.1.100` (example)
   - **Subnet Mask**: `255.255.255.0`
   - **Gateway**: `192.168.1.1` (your router IP)
   - **Port**: `4370` (default)

3. **Save Settings:**
   - Press `OK` or `Enter`
   - Device may reboot

#### Method 2: Using ZKTeco Software

1. Download and install "Search Device" tool from ZKTeco
2. Connect device to network
3. Run tool to discover devices
4. Select device and configure IP settings

#### Method 3: DHCP (Dynamic IP)

1. **Enable DHCP on Device:**
   ```
   Menu → Comm → TCP/IP → DHCP → Enable
   ```

2. **Find Assigned IP:**
   - Check device screen: `Menu → Comm → TCP/IP → IP Address`
   - Or check your router's DHCP client list

### Recommended Network Settings

**For Single Device:**
```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0
Gateway:       192.168.1.1
Port:          4370
```

**For Multiple Devices:**
```
Device 1: 192.168.1.101
Device 2: 192.168.1.102
Device 3: 192.168.1.103
...
All using Port 4370
```

### Network Troubleshooting

**Can't Access Device:**
```bash
# 1. Find device on network
arp -a

# 2. Ping device
ping 192.168.1.100

# 3. Test port
telnet 192.168.1.100 4370
```

**Reset to Default IP:**
- Some devices: `192.168.1.201`
- Check device manual for default IP

---

## Device Settings

### Communication Settings

#### TCP/IP Settings

```
Menu → Comm → TCP/IP
├── IP Address: 192.168.1.100
├── Netmask: 255.255.255.0
├── Gateway: 192.168.1.1
├── Port: 4370
└── DHCP: Disable (for static IP)
```

#### RS232/RS485 Settings

```
Menu → Comm → RS232/RS485
├── Baud Rate: 115200 (or 57600, 38400, 19200)
├── Machine Number: 1
└── Mode: RS232 or RS485
```

#### USB Settings

- Usually auto-configured
- May need to enable USB mode:
  ```
  Menu → Comm → USB → Enable
  ```

### Time and Date Settings

```
Menu → System → Date/Time
├── Date Format: YYYY-MM-DD
├── Time Format: 24H
└── Time Zone: (your timezone)
```

**Sync with PC:**
```csharp
// In your application
DateTime now = DateTime.Now;
device.SetDeviceTime2(machineNumber, 
    now.Year, now.Month, now.Day, 
    now.Hour, now.Minute, now.Second);
```

### Access Control Settings

```
Menu → Access Control
├── Lock Delay: 5 seconds
├── Door Sensor: Enable/Disable
├── Alarm: Enable/Disable
└── Exit Button: Enable/Disable
```

### Attendance Settings

```
Menu → Attendance
├── Auto Clear: Disable (recommend manual clear)
├── Punch State: Enable
├── Work Code: Enable/Disable
└── Photo Capture: Enable/Disable (if supported)
```

### User Capacity Settings

Check device capacity:
```
Menu → Info → Capacity
```

Typical capacities:
- Users: 500 - 30,000 (device dependent)
- Fingerprints: 1,000 - 60,000
- Face templates: 500 - 10,000
- Attendance records: 50,000 - 500,000

---

## Multiple Device Setup

### Scenario 1: Multiple Devices on Same Network

**Configuration:**
```
Office Layout:
├── Main Entrance: 192.168.1.101
├── Back Door: 192.168.1.102
├── Warehouse: 192.168.1.103
└── Parking: 192.168.1.104

All using Port 4370
```

**Code Example:**
```csharp
public class MultiDeviceManager
{
    private Dictionary<string, DeviceConnection> devices = 
        new Dictionary<string, DeviceConnection>();
    
    public void ConnectAll()
    {
        var configs = new[]
        {
            new { Name = "Main Entrance", IP = "192.168.1.101" },
            new { Name = "Back Door", IP = "192.168.1.102" },
            new { Name = "Warehouse", IP = "192.168.1.103" },
            new { Name = "Parking", IP = "192.168.1.104" }
        };
        
        foreach (var config in configs)
        {
            var device = new CZKEMClass();
            if (device.Connect_Net(config.IP, 4370))
            {
                devices[config.Name] = new DeviceConnection
                {
                    Device = device,
                    IP = config.IP,
                    Name = config.Name
                };
                Console.WriteLine($"Connected to {config.Name}");
            }
        }
    }
    
    public List<AttendanceLog> GetAllLogs()
    {
        var allLogs = new List<AttendanceLog>();
        
        foreach (var kvp in devices)
        {
            var logs = GetLogsFromDevice(kvp.Value.Device);
            logs.ForEach(log => log.DeviceName = kvp.Key);
            allLogs.AddRange(logs);
        }
        
        return allLogs;
    }
}

public class DeviceConnection
{
    public CZKEMClass Device { get; set; }
    public string IP { get; set; }
    public string Name { get; set; }
}
```

### Scenario 2: Devices on Different Subnets

**Use VPN or Routing:**
```
Site A (192.168.1.0/24):
├── Device 1: 192.168.1.101

Site B (192.168.2.0/24):
├── Device 2: 192.168.2.101

Connect via VPN or configure routing
```

### Scenario 3: Remote Devices via Internet

**Setup Port Forwarding:**
```
Router Settings:
├── External Port: 4371
└── Internal: 192.168.1.101:4370
```

**Connect from Internet:**
```csharp
device.Connect_Net("your-public-ip", 4371);
```

**Security Recommendations:**
- Use VPN instead of port forwarding
- Change default port
- Enable firewall rules
- Use strong admin passwords

---

## Best Practices

### 1. Network Configuration

✅ **DO:**
- Use static IP addresses for devices
- Document all device IPs and locations
- Use consistent port numbers (4370)
- Keep devices on same subnet when possible
- Use quality network cables (Cat5e or better)

❌ **DON'T:**
- Use DHCP unless necessary
- Mix devices across many subnets
- Use WiFi for critical attendance devices
- Daisy chain network switches excessively

### 2. Device Placement

✅ **DO:**
- Place devices at comfortable height (1.2-1.5m)
- Ensure good lighting for face/palm devices
- Protect from weather (for outdoor units)
- Provide stable power supply (UPS recommended)
- Keep away from electromagnetic interference

❌ **DON'T:**
- Place in direct sunlight (affects face recognition)
- Install near powerful electromagnets
- Use unstable power sources
- Block device ventilation

### 3. User Management

✅ **DO:**
- Use meaningful enrollment numbers (employee IDs)
- Backup user data regularly
- Enroll multiple fingers per user (backup)
- Test biometric enrollment quality
- Keep user database synchronized

❌ **DON'T:**
- Use sequential numbers without meaning
- Delete users without backup
- Rely on single finger enrollment
- Ignore enrollment quality warnings

### 4. Attendance Data

✅ **DO:**
- Download attendance data daily
- Backup data before clearing device
- Validate data before clearing device
- Use time-range queries for large datasets
- Archive old data

❌ **DON'T:**
- Let device memory fill up completely
- Clear logs without downloading
- Keep years of data on device
- Ignore device capacity warnings

### 5. Maintenance

✅ **DO:**
- Clean fingerprint sensor regularly
- Update firmware periodically
- Test connections regularly
- Monitor device status
- Keep SDK updated

❌ **DON'T:**
- Ignore cleaning schedule
- Update firmware during peak hours
- Skip connection tests
- Ignore error messages

### 6. Security

✅ **DO:**
- Change default admin password
- Use encryption for network traffic (if supported)
- Restrict physical access to devices
- Audit admin actions
- Use VLAN for device network

❌ **DON'T:**
- Leave default passwords
- Expose devices to internet without VPN
- Give admin access to all users
- Ignore security updates

---

## Device Testing Checklist

### Initial Setup Testing

- [ ] Device powers on successfully
- [ ] Display shows correctly
- [ ] Network connectivity established
- [ ] Can ping device from PC
- [ ] SDK can connect to device
- [ ] Device time is correct

### Functionality Testing

- [ ] User enrollment works
- [ ] Fingerprint verification works
- [ ] Attendance logs are recorded
- [ ] Data can be downloaded
- [ ] Real-time events work
- [ ] Device time stays accurate

### Integration Testing

- [ ] Application connects successfully
- [ ] Data transfers correctly
- [ ] Multiple devices work together
- [ ] Error handling works
- [ ] Reconnection works after disconnect
- [ ] Performance is acceptable

---

## Configuration Templates

### Small Office (1-3 Devices)

**Device Config:**
```ini
[Device-MainEntrance]
IP=192.168.1.100
Port=4370
MachineNumber=1
Location=Main Entrance

[Device-BackDoor]
IP=192.168.1.101
Port=4370
MachineNumber=1
Location=Back Door
```

### Medium Office (4-10 Devices)

**Recommended Setup:**
- Dedicated network switch for devices
- Static IPs (192.168.1.100-110)
- Central management server
- Daily automatic data collection

### Large Enterprise (10+ Devices)

**Recommended Setup:**
- Multiple subnets by building/floor
- Redundant network connections
- Load-balanced management servers
- Real-time data synchronization
- 24/7 monitoring

---

## Quick Reference Card

### Default Settings

| Setting | Default Value |
|---------|--------------|
| IP Address | 192.168.1.201 (varies) |
| Port | 4370 |
| Admin Password | (empty) or `0` |
| Machine Number | 1 |
| Baud Rate (RS232) | 115200 |

### Menu Navigation

| Action | Path |
|--------|------|
| Set IP | Menu → Comm → TCP/IP |
| Set Time | Menu → System → Date/Time |
| View Users | Menu → User Manage |
| View Logs | Menu → Attendance |
| Device Info | Menu → Info |
| Access Control | Menu → Access Control |

### SDK Connection Test

```csharp
// Quick connection test
public static bool TestDevice(string ip)
{
    var device = new CZKEMClass();
    try
    {
        if (device.Connect_Net(ip, 4370))
        {
            string serial = "";
            device.GetSerialNumber(1, out serial);
            Console.WriteLine($"Connected! Serial: {serial}");
            device.Disconnect();
            return true;
        }
        return false;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
        return false;
    }
}
```

---

## Additional Resources

- [API Reference](API_REFERENCE.md)
- [Troubleshooting Guide](TROUBLESHOOTING.md)
- [.NET Integration Guide](DOTNET_INTEGRATION_GUIDE.md)
- [Device Manuals](https://www.zkteco.com/support)

---

**Last Updated**: 2024  
**SDK Version**: 6.2.4.11
