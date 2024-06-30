# Device commands

## Install and uninstall applications

You can use several ways to install and uninstall applications

#### Using PackageManager
You can use `InstallPackageAsync` and `UninstallPackageAsync` methods from `PackageManager` to install and uninstall apps:
```csharp
....
PackageManager manager = new PackageManager(client, device);
// Install package
await manager.InstallPackageAsync(@"C:\mypackage.apk", new Action<InstallProgressEventArgs>(o => { }));
// Uninstall package
await manager.UninstallPackageAsync("com.android.app");
```

#### Using AdbClient
Or you can use `InstallAsync` and `UninstallAsync` methods from `AdbClient` to install and uninstall apps:
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
To install multiple packages you need to use `InstallMultiplePackageAsync` method from `PackageManager`:
```csharp
....
PackageManager manager = new PackageManager(client, device);
// Install split app whith base app
await manager.InstallMultiplePackageAsync(@"C:\base.apk", new[] { @"C:\split_1.apk", @"C:\split_2.apk" }, new Action<InstallProgressEventArgs>(a => { }));
// Add split app to base app which packagename is 'com.android.app'
await manager.InstallMultiplePackageAsync(new[] { @"C:\split_3.apk", @"C:\split_4.apk" }, "com.android.app", new Action<InstallProgressEventArgs>(a => { }));
```

#### Install multiple applications using AdbClient
Or you can use `InstallMultipleAsync` from `AdbClient` to install multiple applications:
```csharp
....
// Install split app whith base app
await client.InstallMultipleAsync(device, File.OpenRead("base.apk"), new[] { File.OpenRead("split_1.apk"), File.OpenRead("split_2.apk") });
// Add split app to base app which packagename is 'com.android.app'
await client.InstallMultipleAsync(device, new[] { File.OpenRead("split_3.apk"), File.OpenRead("split_4.apk") }, "com.android.app");
```


## Start and stop applications

`AdvancedSharpAdbClient` uses [monkeyrunner](https://developer.android.com/studio/test/monkeyrunner) to start and `force-stop` to stop applications.

You can use `StartAppAsync` and `StopAppAsync` from `AdbClient` to start and stop applications:
```csharp
....
// Start app 
await client.StartAppAsync(device, "com.android.app");
// Force stop app
await client.StopAppAsync(device, "com.android.app");
```

To check app status use `GetAppStatusAsync` from `AdbClient` or `IsAppRunningAsync` and `IsAppInForegroundAsync` from `DeviceClient`:
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
You can push and pull files to `Stream` such as `MemoryStream` or `FileStream` using `PullAsync` and `PushAsync` methods.

#### Using SyncService
Use `SyncService` to push and pull files.

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
Use `AdbClient` to push and pull files.

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
Use `ExecuteRemoteCommandAsync` to execute your own custom shell commands and `IShellOutputReceiver` to receive result:
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
To forward local to remote port device use `CreateForwardAsync` method:
```csharp
....
// Forward local TCP port to device port
int tcpres = await client.CreateForwardAsync(device, 6123, 7123);
// Forward local UDP port to device port
int udpres = await client.CreateForwardAsync(device, "udp:6123", "udp:7123", true);
```

#### Forward from remote to local connection
To forward remote to local port use `CreateReverseForwardAsync` method:
```csharp
....
// Forward from remote to local TCP port
int tcpres = await client.CreateReverseForwardAsync(device, "tcp:7123", "tcp:6123", true);
// Forward from remote to local UDP port
int udpres = await client.CreateReverseForwardAsync(device, "udp:7123", "udp:6123", true);
```

#### Remove forward
To remove forwards use `RemoveAllForwardsAsync`, `RemoveAllReverseForwardsAsync`, `RemoveForwardAsync` and `RemoveReverseForwardAsync` methods:
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