# What is this ?

ZUpdater is a simple library to help you manage updates for your Qt application. It checks for new versions, downloads updates all based on **GitHub releases**

Currently being used in [iDescriptor](https://github.com/iDescriptor/iDescriptor)

**Expect breaking changes as this is still in early development stage. This readme will be kept to update**

# Features

- Check for updates
- Download updates
- Single click to install the update

# How to use ?

1. Add as a submodule to your project

```bash
git submodule add https://github.com/libZQT/ZUpdater lib/zupdater
git submodule update --init --recursive
```

2. Include ZUpdater in your project file
   CMakeLists.txt

```cmake
add_subdirectory(lib/zupdater)
target_link_libraries(your_target PRIVATE ZUpdater)
```

3. Initialize ZUpdater in your application

Here is how iDescriptor does it (exact code from [mainwindow.cpp](https://github.com/iDescriptor/iDescriptor/blob/main/src/mainwindow.cpp)):

```cpp
   #include "ZUpdater.h"

   UpdateProcedure updateProcedure;
    bool packageManagerManaged = false;
    bool isPortable = false;
    bool skipPrerelease = true;
#ifdef WIN32
    isPortable = !is_iDescriptorInstalled();
#endif

    /*
    struct UpdateProcedure {
        bool openFile;
        bool openFileDir;
        bool quitApp;
        QString boxInformativeText;
        QString boxText;
    };
    */
    switch (ZUpdater::detectPlatform()) {
    case Platform::Windows:
        updateProcedure = UpdateProcedure{
            !isPortable,
            isPortable,
            !isPortable,
            isPortable ? "New portable version downloaded, app location will "
                         "be shown after this message"
                       : "The application will now quit to install the update.",
            isPortable ? "New portable version downloaded"
                       : "Do you want to install the downloaded update now?",
        };
        break;
    case Platform::MacOS:
        updateProcedure = UpdateProcedure{
            true,
            false,
            true,
            "The application will now quit and open .dmg file downloaded to "
            "\"Downloads\" from there you can drag it to Applications to "
            "install.",
            "Update downloaded would you like to quit and install the update?",
        };
        break;
    case Platform::Linux:
        // currently only on linux (arch aur) is enabled
#ifdef PACKAGE_MANAGER_MANAGED
        packageManagerManaged = true;
#endif
        updateProcedure = UpdateProcedure{
            true,
            false,
            true,
            "AppImages we ship are not updateable. New version is downloaded "
            "to "
            "\"Downloads\". You can start using the new version by launching "
            "it "
            "from there. You can delete this AppImage version if you like.",
            "Update downloaded would you like to quit and open the new "
            "version?",
        };
        break;
    default:
        updateProcedure = UpdateProcedure{
            false, false, false, "", "",
        };
    }
// app version should be in format "x.y.z",where x,y,z are integers; example "1.0.3"
    m_updater = new ZUpdater(YOUR_REPO_NAME_HERE,YOUR_APP_VERSION_HERE,
                             YOUR_APP_NAME_HERE, updateProcedure, isPortable,
                             packageManagerManaged, skipPrerelease, this);
//  !OPTIONAL
// #if defined(PACKAGE_MANAGER_MANAGED) && defined(__linux__)
//     m_updater->setPackageManagerManagedMessage(
//         QString(
//             "You seem to have installed %1 using a package manager. "
//             "Please use %2 to update it.")
//             .arg(YOUR_APP_NAME_HERE)
//             .arg(PACKAGE_MANAGER_HINT));
// #endif


    qDebug() << "Checking for updates...";
    m_updater->checkForUpdates();
```

4. GitHub Release Entities

ZUpdater uses GitHub releases to manage updates. You need to create releases in your GitHub repository with the following naming conventions for assets:

- For Windows(portable): **\*-Windows\_$arch.portable.zip**
  <br>
  Example: **iDescriptor-Windows_x86_64.portable.zip**
  <br>
  or
  <br>
  Example: **iDescriptor-v1.0.0-Windows_x86_64.portable.zip**

- For Windows: **\*-Windows\_$arch.msi**
  <br>
  Example: **iDescriptor-Windows_x86_64.msi**
  <br>
  or
  <br>
  Example: **iDescriptor-v1.0.0-Windows_x86_64.msi**

- For MacOS: **\*-Apple\_$type.dmg**
  <br>
  Intel
  Example: **iDescriptor-Apple_Intel.dmg**
  <br>
  Apple Silicon
  Example: **iDescriptor-Apple_Silicon.dmg**
- For Linux: **\*-Linux\_$arch.AppImage.zip**
  <br>
  Example: **iDescriptor-Linux_x64.AppImage.zip**

<br>

## Full table example:

| Asset                                          | SHA256                                                           |   Size | Uploaded   |
| ---------------------------------------------- | ---------------------------------------------------------------- | -----: | ---------- |
| iDescriptor-v0.1.0-Apple_Intel.dmg             | f1de21171503e0242b3cc4b61d354a4b31c96d9f3d9fc05711c2f5c858667f5d | 181 MB | 5 days ago |
| iDescriptor-v0.1.0-Apple_Silicon.dmg           | 65989c3663bb90d8d24750cf74ec93d5e900c22317494038ea0b864bd0deef1c | 182 MB | 5 days ago |
| iDescriptor-v0.1.0-Linux_x86_64.AppImage.zip   | b64eca267760bbb812be151911127cb65adcce6efddc6b4d6c14dc475526bb94 | 169 MB | 5 days ago |
| iDescriptor-v0.1.0-Windows_x86_64.msi          | bb803b3e4cfaa75068c28d15808284ad7d3400ed2bb7e98cdf1a2223d0543b6e | 145 MB | 5 days ago |
| iDescriptor-v0.1.0-Windows_x86_64.portable.zip | 5af66c05d4550c84135492e645b6ecd2123422af87e7b8b46554794916f21525 | 161 MB | 5 days ago |

## Why AppImage.zip for Linux ?

Because GitHub strips the executable permission when uploading assets. However if you zip the AppImage before uploading, the permission is preserved when extracted. So the user will not have to manually set the executable permission after downloading. Just extract and double click to run.

Don't forget to `chmod +x your_AppImage` before zipping it though. ZUpdater will set the executable permission again after extracting (when updating), but for new users downloading for the first time, it must be set before zipping.

# License

ZUpdater is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

# Contributing

Contributions are welcome! Feel free to open issues or submit pull requests.

# Thanks

- [QSimpleUpdater](https://github.com/alex-spataru/QSimpleUpdater) - this project was originally forked from QSimpleUpdater and heavily modified to suit the needs.
