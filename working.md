# Working with lib

## Working with single device
For each executed command with AdvancedSharpAdbClient, you need to specify for which device it should be performed. You can obtain the necessary device from the following code:
```csharp
AdbClient client = new AdbClient();
IEnumerable<DeviceData> devices = await client.GetDevicesAsync();
DeviceData device = devices.FirstOrDefault();
```

Next, you can use this device to execute commands, for example, to capture a screenshot:
```csharp
Framebuffer frameBuffer = await client.GetFrameBufferAsync(device);
Bitmap screenshot = frameBuffer.ToImage();
```

## Working with multiple devices
Working with multiple devices is no different from working with a single device. You just need to connect to all devices and send commands to each device one by one:
```csharp
AdbClient client = new AdbClient();
IEnumerable<DeviceData> devices = await client.GetDevicesAsync();
foreach (DeviceData device in devices)
{
    Framebuffer frameBuffer = await client.GetFrameBufferAsync(device);
    Bitmap screenshot = frameBuffer.ToImage();
}
```