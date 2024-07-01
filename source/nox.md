# Automate multiple Nox emulators
This tutorial will show you how to work with multiple devices in [Nox emulator](https://en.bignox.com)

## Start adb server
You can use `adb.exe` from the Nox directory located at the `bin/adb.exe`:
```csharp
if (!AdbServer.Instance.GetStatus().IsRunning)
{
    AdbServer server = new AdbServer();
    StartServerResult result = await server.StartServerAsync(@"F:\Nox\bin\adb.exe", false);
    if (result != StartServerResult.Started)
    {
        Console.WriteLine("Can't start adb server");
        return;
    }
}
```

## Start emulators
First you need to create and configure the required number of emulators in the Nox control panel.
After that, you can run them using the code:
```csharp
int emulatorCount = 5;
// Starting 5 emulators
for (int i = 0; i < emulatorCount; i++)
{
    Process process = new Process();
    process.StartInfo.FileName = @"F:\Nox\bin\Nox.exe";
    process.StartInfo.Arguments = $"-clone:Nox_{i}";
    process.Start();
}
```
!> You must wait for emulators to start and load

## Connect to emulators
Nox has each port starting at 620, for example 62001, 62025, 62040.
To find ports, you can use the [`netstat`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) utility, which listens for active ports and connections:
```csharp
List<int> ports = new List<int>();
Process[] processes = Process.GetProcessesByName("NoxVMHandle");
foreach (Process process in processes)
{
    ProcessStartInfo startInfo = new ProcessStartInfo("cmd", "/c netstat -a -n -o | find \"" + process.Id + "\" | find \"127.0.0.1\" | find \"620\"");
    startInfo.UseShellExecute = false;
    startInfo.CreateNoWindow = true;
    startInfo.RedirectStandardOutput = true;

    Process proc = new Process();
    proc.StartInfo = startInfo;
    proc.Start();
    proc.WaitForExit();

    MatchCollection matches = Regex.Matches(proc.StandardOutput.ReadToEnd(), "(?<=127.0.0.1:)62.*?(?= )");
    foreach (Match match in matches)
    {
        if (!string.IsNullOrEmpty(match.Value))
        {
            string rawPort = match.Value;
            int port = int.Parse(rawPort);
            ports.Add(port);
        }
    }
}
```

Next, you need to connect to them:
```csharp
....
AdbClient client = new AdbClient();
foreach (int port in ports)
{
    await client.ConnectAsync("127.0.0.1", port);
}
```

Now you can use all connected devices to automate things like taking screenshots:
```csharp
....
IEnumerable<DeviceData> devices = await client.GetDevicesAsync();
foreach (DeviceData device in devices)
{
    Framebuffer framebuffer = await client.GetFrameBufferAsync(device);
    Bitmap screenshot = framebuffer.ToImage();
    string filename = device.Name + ".png";
    screenshot.Save(filename);
}
```

## Multithreading
If you want to use multithreading, you need to create an [`AdbClient`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L33) for each device and work independently of each other:
```csharp
....
List<Thread> threads = new List<Thread>();
// Start threads
foreach (int port in ports)
{
    Thread thread = new Thread(async () =>
    {
        AdbClient client = new AdbClient();
        await client.ConnectAsync("127.0.0.1", port);
        IEnumerable<DeviceData> devices = await client.GetDevicesAsync();
        DeviceData device = devices.First();
        Framebuffer framebufffer = await client.GetFrameBufferAsync(device);
    });
    thread.Start();
    threads.Add(thread);
}
....
// Stop threads
foreach (Thread thread in threads)
{
    thread.Abort();
}
```