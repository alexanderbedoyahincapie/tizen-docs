# Data Storages


You can access storage information and manage directories within certain parts of the file system, represented as [virtual root locations](#virtualroots). These virtual roots form a collection of locations that function as a single virtual device file system.

The main features of the `Tizen.System.Storage` class includes the following:

-   Storage management

    You can manage different storage locations on the device.

    You can [retrieve additional information about the storage locations](#storage), including which storage is supported on the device, using the [Tizen.System.StorageManager](/application/dotnet/api/TizenFX/latest/api/Tizen.System.StorageManager.html) class. You can also [monitor storage state changes](#state).

-   Storage space management

    You can [get the available and total size](#space) of the storage.

## Prerequisites

To use the methods and properties of the [Tizen.System.Storage](/application/dotnet/api/TizenFX/latest/api/Tizen.System.Storage.html) and [Tizen.System.StorageManager](/application/dotnet/api/TizenFX/latest/api/Tizen.System.StorageManager.html) classes, include the [Tizen.System](/application/dotnet/api/TizenFX/latest/api/Tizen.System.html) namespace in your application:

```csharp
using Tizen.System;
```

<a name="storage"></a>
## Retrieve storage information

To retrieve storage information, follow the steps below:

1.  Retrieve all storages on a device by using the `Storages` property of the [Tizen.System.StorageManager](/application/dotnet/api/TizenFX/latest/api/Tizen.System.StorageManager.html) class, which returns a list of all device storages as instances of the [Tizen.System.Storage](/application/dotnet/api/TizenFX/latest/api/Tizen.System.Storage.html) class:

    ```csharp
    var storages = StorageManager.Storages;
    var internalStorage = storages.Where(s => s.StorageType == StorageArea.Internal).FirstOrDefault();
    ```

2.  To get information about each storage instance, follow these steps:
    -   Get the ID of a specific storage by using the `Id` property:

        ```csharp
        var result = internalStorage.Id;
        ```

    -   Get the root directory of a specific storage by using the `RootDirectory` property:

        ```csharp
        var result = internalStorage.RootDirectory;
        ```

    -   Get the directory path for a storage of a specific type.

        The `GetAbsolutePath()` method of the `Tizen.System.Storage` class retrieves the absolute path to a storage of a particular type, which is defined by the values of the [Tizen.System.DirectoryType](/application/dotnet/api/TizenFX/latest/api/Tizen.System.DirectoryType.html) enumeration.

        To get the directories of all storage types:

        ```csharp
        foreach (DirectoryType directoryType in Enum.GetValues(typeof(DirectoryType)))
        {
            var result = internalStorage.GetAbsolutePath(directoryType);
        }
        ```

    -   Get the storage type of a specific storage by using the `StorageType` property, which takes values from the [Tizen.System.StorageArea](/application/dotnet/api/TizenFX/latest/api/Tizen.System.StorageArea.html) enumeration:

        ```csharp
        var result = internalStorage.StorageType;
        ```

    -   Get the mount state of a specific storage by using the `State` property, which takes values from the [Tizen.System.StorageState](/application/dotnet/api/TizenFX/latest/api/Tizen.System.StorageState.html) enumeration:

        ```csharp
        var result = internalStorage.State;
        ```

<a name="state"></a>
## Monitor storage state changes

To monitor storage state changes, follow the steps below:

1.  Define an event handler, which is called when the storage state changes:

    ```csharp
    EventHandler storageEventHandler = (s, e) =>
    {
        var storage = s as Storage;
        if (storage == null)
        {
            return;
        }
        if (storage.State == StorageState.Unmountable)
        {
            unmountableStateAchieved = true;
        }
    };
    ```

    The event handler checks whether storage can be mounted or is corrupted and unmountable.

2.  Register the event handler for the `StorageStateChanged` event of the [Tizen.System.Storage](/application/dotnet/api/TizenFX/latest/api/Tizen.System.Storage.html) class:

    ```csharp
    foreach (var storage in StorageManager.Storages)
    {
        storage.StorageStateChanged += storageEventHandler;
    }
    ```

3.  When no longer needed, deregister the event handler:

    ```csharp
    foreach (var storage in StorageManager.Storages)
    {
        storage.StorageStateChanged -= storageEventHandler;
    }
    ```

<a name="space"></a>
## Retrieve storage space information

To get the available and total size of the storage, use the `TotalSpace` and `AvailableSpace` properties in the [Tizen.System.Storage](/application/dotnet/api/TizenFX/latest/api/Tizen.System.Storage.html) class. For internal storage, they return the storage size, excluding the minimum memory size to launch the low memory pop-up in a low memory situation. Consequently, the available size must be less than the original available size, and you must use these properties to get the memory size.

To retrieve storage space information, follow the steps below:

1.  Get the total space of the storage by using the `TotalSpace` property, which returns the total space of the given storage in bytes:

    ```csharp
    var storages = StorageManager.Storages;
    var internalStorage = storages.Where(s => s.StorageType == StorageArea.Internal).FirstOrDefault();
    var result = internalStorage.TotalSpace;
    ```

2.  Get the available space of the storage by using the `AvailableSpace` property, which returns the available space of the given storage in bytes:

    ```csharp
    var storages = StorageManager.Storages;
    var internalStorage = storages.Where(s => s.StorageType == StorageArea.Internal).FirstOrDefault();
    var result = internalStorage.AvailableSpace;
    ```

<a name="virtualroots"></a>
## Virtual roots

The following table lists the supported virtual roots.

**Table: File system virtual roots**

| Virtual root | Description                              |
|------------|----------------------------------------|
| `Images`     | Location for storing images.             |
| `Sounds`     | Location for storing sound files.        |
| `Videos`     | Location for storing video files.        |
| `Camera`     | Location for storing photos.             |
| `Downloads`  | Location for storing downloaded items.   |
| `Music`      | Location for storing audio files.        |
| `Documents`  | Location for storing documents.          |
| `Others`     | Location for storing other files.        |
| `Ringtones`  | Location for ringtones (read-only location). |



## Related information
* Dependencies
  -   Tizen 4.0 and Higher
