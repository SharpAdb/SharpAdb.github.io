# Screen automation


## Overview screen elements
AdvancedSharpAdbClient uses [UIautomator](https://developer.android.com/training/testing/other-components/ui-automator) to get the screen xml.

The location of screen elements in xml format can be obtained with the [`DumpScreen`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L81) method:
```csharp
....
// Gets the current device UI hierarchy.
XmlDocument screenXml = await client.DumpScreenAsync(device);
```

!> Currently your UI needs to be idle (as in no [Accessibility Events](https://developer.android.com/reference/android/view/accessibility/AccessibilityEvent.html) sent) for at least 1000ms before the UIautomator will be able to produce the UI hierarchy dump, otherwise you will get the error «ERROR: could not get idle state».


## Find elements
You can find items with [xpath](https://www.w3schools.com/xml/xpath_syntax.asp) using [`FindElementAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L284) and [`FindElementsAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L323) methods:
```csharp
....
// Find single element
Element? element = await client.FindElementAsync(device, "hierarchy/node");

// Find all elements with given xpath
IEnumerable<Element> elements = await client.FindElementsAsync(device, "hierarchy/node");
```

> [UIautomatorviewer](https://developer.android.com/training/testing/other-components/ui-automator#inspect-ui) tool can be used to find xpath for elements


## Click on an element
To click on the screen use the [`ClickAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L119) method.

You can tap the screen by raw coordinates:
```csharp
....
// Click on (480, 560) coordinates
await client.ClickAsync(device, new Point(480, 560));
```

Or by a previously found element:
```csharp
....
Element element = ....
// Click on an elements
await element.ClickAsync();
```


## Swipe
To swipe from one coordinate to another, use the [`SwipeAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L212) method:
```csharp
....
// Swipes from(0, 0) to (350, 0) within 1 second (1000 ms)
await client.SwipeAsync(device, new Point(0, 0), new Point(350, 0), 1000);
```


## Type text

#### Using standart adb keyboard
You can use [`SendTextAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L387) method to type text.

To current device state:
```csharp
....
// Send text to current device state
await client.SendTextAsync(device, "hello world!");
```

To the input element:
```csharp
....
Element = ....
// Focus on the elements and send text
await element.SendTextAsync("hello world!");
```

!> This method does not support unicode characters (such as cyryllic and chinese).

#### Using ADBKeyBoard
This method supports unicode characters, but it need [ADBKeyBoard](https://github.com/senzhk/ADBKeyBoard) to be installed on the device:
```csharp
...
// Send unicode text via ADBKeyBoard
string message = "Привет мир!";
await client.ExecuteRemoteCommandAsync($"am broadcast -a ADB_INPUT_TEXT --es msg '{message}'", device);
```

#### Clear input
Use the [`ClearInputAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L408) method to clear an input with a specified length:
```csharp
....
// Remove 250 symbols of current input
await client.ClearInputAsync(device, 250);
```


## Send key events
You can send key events using [`SendKeyEventAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/DeviceCommands/DeviceClient.cs#L366) method:
```csharp
....
// Send contacts key event
await client.SendKeyEventAsync(device, "KEYCODE_CONTACTS");
```

Or using predefined key events:
```csharp
....
// Send KEYCODE_BACK key event
await client.ClickBackButtonAsync(device);
// Send KEYCODE_HOME key event
await client.ClickHomeButtonAsync(device);
```

## Create screenshots
Use the [`GetFrameBufferAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/AdbClient.cs#L575) method to obtain screenshot:
```csharp
....
// Get the current screen state
Framebuffer framebuffer = await client.GetFrameBufferAsync(device);
// Convert to bitmap
Bitmap? image = framebuffer.ToImage();
```
> **Note** to have access to the [`ToImage`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/Models/Framebuffer.cs#L212) (windows) or [`ToBitmapAsync`](https://github.com/SharpAdb/AdvancedSharpAdbClient/blob/main/AdvancedSharpAdbClient/Models/Framebuffer.cs#L244) (UWP) method, you must specify the target platform of the project