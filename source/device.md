# Device commands

## Install and uninstall applications

You can use several ways to install and uninstall applications

#### Using PackageManager
You can use [`InstallPackageAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/PackageManager.cs#L156) and [`UninstallPackageAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/PackageManager.cs#L478) methods from [`PackageManager`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/PackageManager.cs#L18) to install and uninstall apps:
```csharp
....
PackageManager manager = new PackageManager(client, device);
// Install package
await manager.InstallPackageAsync(@"C:\mypackage.apk", new Action<InstallProgressEventArgs>(o => { }));
// Uninstall package
await manager.UninstallPackageAsync("com.android.app");
```

#### Using AdbClient
Or you can use [`InstallAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L687) and [`UninstallAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L1084) methods from [`AdbClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L33) to install and uninstall apps:
```csharp
....
using (FileStream stream = File.OpenRead("Application.apk"))
{
    // Installing applcation
    await client.InstallAsync(device, stream);
    // Uninstalling application
    await client.UninstallAsync(device, "com.android.app");
}
```

#### Install multiple applications using PackageManager
To install multiple packages you need to use [`InstallMultiplePackageAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/PackageManager.cs#L219) method from [`PackageManager`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/PackageManager.cs#L18):
```csharp
....
PackageManager manager = new PackageManager(client, device);
// Install split app whith base app
await manager.InstallMultiplePackageAsync(@"C:\base.apk", new[] { @"C:\split_1.apk", @"C:\split_2.apk" }, new Action<InstallProgressEventArgs>(a => { }));
// Add split app to base app which packagename is 'com.android.app'
await manager.InstallMultiplePackageAsync(new[] { @"C:\split_3.apk", @"C:\split_4.apk" }, "com.android.app", new Action<InstallProgressEventArgs>(a => { }));
```

#### Install multiple applications using AdbClient
Or you can use [`InstallMultipleAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L756) from [`AdbClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L33) to install multiple applications:
```csharp
....
// Install split app whith base app
await client.InstallMultipleAsync(device, File.OpenRead("base.apk"), new[] { File.OpenRead("split_1.apk"), File.OpenRead("split_2.apk") });
// Add split app to base app which packagename is 'com.android.app'
await client.InstallMultipleAsync(device, new[] { File.OpenRead("split_3.apk"), File.OpenRead("split_4.apk") }, "com.android.app");
```


## Start and stop applications

`AdvancedSharpAdbClient` uses [monkeyrunner](https://developer.android.com/studio/test/monkeyrunner) to start and `force-stop` to stop applications.

You can use [`StartAppAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L428) and [`StopAppAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L434) from [`AdbClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L33) to start and stop applications:
```csharp
....
// Start app 
await client.StartAppAsync(device, "com.android.app");
// Force stop app
await client.StopAppAsync(device, "com.android.app");
```

To check app status use [`GetAppStatusAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L263) from [`AdbClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L33) or [`IsAppRunningAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L234) and [`IsAppInForegroundAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L249) from [`DeviceClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L22):
```csharp
....
// Get app status using AdbClient
AppStatus appStatus= await client.GetAppStatusAsync(device, "com.android.app");

// Get app status using DeviceClient
DeviceClient deviceClient = device.CreateDeviceClient();
bool isAppRunning = await deviceClient.IsAppRunningAsync("com.android.app");
bool isAppForeground = await deviceClient.IsAppInForegroundAsync("com.android.app");
```

## Push and pull files
You can push and pull files to [`Stream`](https://learn.microsoft.com/en-us/dotnet/api/system.io.stream) such as [`MemoryStream`](https://learn.microsoft.com/en-us/dotnet/api/system.io.memorystream) or [`FileStream`](https://learn.microsoft.com/en-en/dotnet/api/system.io.filestream) using [`PullAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/SyncService.cs#L247) and [`PushAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/SyncService.cs#L149) methods.

#### Using SyncService
Use [`SyncService`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/SyncService.cs#L45) to push and pull files.

Pull files:
```csharp
....
using (SyncService syncService = new SyncService(device)) 
{
    // Write data to C:\MyFile.txt
    using (FileStream stream = File.OpenWrite(@"C:\MyFile.txt"))
    {
        // Download file from device
        await syncService.PullAsync("/data/local/tmp/MyFile.txt", stream, null);
    }
}
```

Push files:
```csharp
....
using (SyncService service = new SyncService(device))
{
    // Read data from C:\MyFile.txt
    using (FileStream stream = File.OpenRead(@"C:\MyFile.txt"))
    {
        // Push data to device
        await service.PushAsync(stream, "/data/local/tmp/MyFile.txt", UnixFileStatus.DefaultFileMode, DateTimeOffset.Now, null);
    }
}
```

#### Using AdbClient
Use [`AdbClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L33) to push and pull files.

Pull:
```csharp
....
using (FileStream stream = File.OpenWrite(@"C:\MyFile.txt"))
{
    // Download file from device
    await client.PullAsync(device, "/data/local/tmp/MyFile.txt", stream, new Action<SyncProgressChangedEventArgs>(o => { }));
}
```

Push:
```csharp
....
using (FileStream stream = File.OpenRead(@"C:\MyFile.txt"))
{
    // Push data to device
    await client.PushAsync(device, "/data/local/tmp/MyFile.txt", stream, UnixFileStatus.DefaultFileMode, DateTimeOffset.Now, new Action<SyncProgressChangedEventArgs>(o => { }));
}
```


## Run shell commands
Use [`ExecuteRemoteCommandAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L345) to execute your own custom shell commands and [`IShellOutputReceiver`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/Receivers/IShellOutputReceiver.cs#L14) to receive result:
```csharp
....
IShellOutputReceiver receiver = new ConsoleOutputReceiver();
// Execute shell command with receiver
await client.ExecuteShellCommandAsync(device, "adb shell am start -a android.intent.action.CALL -d tel:+972527300294", receiver);
// Shell command output
string allOutput = receiver.ToString();

// Execute shell command without receiver
await client.ExecuteShellCommandAsync(device, "pm reset-permissions -p com.android.app");
```

## Forward connection
Forward connection allows you to redirect connection from a port on the local device to a port on the mobile device and back again.

#### Forward local to remote connection
To forward local to remote port device use [`CreateForwardAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L216) method:
```csharp
....
// Forward local TCP port to device port
int tcpres = await client.CreateForwardAsync(device, 6123, 7123);
// Forward local UDP port to device port
int udpres = await client.CreateForwardAsync(device, "udp:6123", "udp:7123", true);
```

#### Forward from remote to local connection
To forward remote to local port use [`CreateReverseForwardAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L232) method:
```csharp
....
// Forward from remote to local TCP port
int tcpres = await client.CreateReverseForwardAsync(device, "tcp:7123", "tcp:6123", true);
// Forward from remote to local UDP port
int udpres = await client.CreateReverseForwardAsync(device, "udp:7123", "udp:6123", true);
```

#### Remove forward
To remove forwards use [`RemoveAllForwardsAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L284), [`RemoveAllReverseForwardsAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L262), [`RemoveForwardAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L274) and [`RemoveReverseForwardAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L250) methods:
```csharp
....
// Remove all forwards from the device
await client.RemoveAllForwardsAsync(device);
// Remove all reverse forwards from the device
await client.RemoveAllReverseForwardsAsync(device);
// Remove forward from specified port
await client.RemoveForwardAsync(device, 6123);
// Remove reverse forward from specified port
await client.RemoveReverseForwardAsync(device, "tcp:7123");
```