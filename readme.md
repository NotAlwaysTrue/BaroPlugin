# Base Barotrauma Plugin

This repository provides a base template used by most Barotrauma plugins.

## Requirements

Before getting started, make sure you have an IDE that supports **.NET 8**:

- **Visual Studio 2022 or later** (with C# 10 support)  
  https://www.visualstudio.com/vs/community/
- **JetBrains Rider**  
  https://www.jetbrains.com/rider/

It is also recommended to have **Git** installed:

- **Windows**: Download from https://git-scm.com/install/windows  
- **Linux**: Install Git using your distribution’s package manager

## Getting the Project

There are two ways to obtain the project:

### Option 1: Use the GitHub Template (Recommended)

<img width="400" alt="image" src="https://github.com/user-attachments/assets/fbd5fb22-04cb-471f-9c4d-456736487b35" />


1. Create or sign in to a GitHub account.
2. Click the **Use this template** button in the top-right corner of this repository.
3. Clone your newly created repository locally:
   ```bash
   git clone https://github.com/your_username/your_repo_name
   ```

### Option 2: Download Directly

<img width="429" alt="image" src="https://github.com/user-attachments/assets/ce9acc22-b516-46aa-af85-7501ed8d942a" />

You can also download the repository as a ZIP file from GitHub and extract it locally, though this option does not include version control.

## Setting up the Project

Before opening the project, a few configuration steps are required.

> `.props` files can be edited using any text editor.

### Required Configuration

This repository **is** the mod folder: Barotrauma loads it directly, so it has to sit inside `LocalMods`. If your copy lives somewhere else (for example `LocalMods/.Templates`), move or re-clone it to:

```
<Barotrauma Installation>\LocalMods\<YourModName>
```

Then edit the file `CsProjectSource/BuildData.props` and replace `BaseMod` with your actual mod name. Keep the folder name and `ModName` in sync so `%ModDir%` and the assembly paths stay consistent.

Finally, copy `CsProjectSource/UserBuildData.props.example` to `CsProjectSource/UserBuildData.props` and set `BarotraumaExePath` to your own `Barotrauma.exe`. That file is ignored by git, so machine specific paths never end up in the repository.

> The `Launch Barotrauma` profile in `CsProjectSource/PluginToolbox/Properties/launchSettings.json` launches `$(BarotraumaExePath)`. When `UserBuildData.props` is missing it falls back to `<repo>\..\..\Barotrauma.exe`, which assumes the repository sits directly in `LocalMods`.

## Opening the Project

You can now open the solution file `CsProjectSource/WindowsSolution.sln` (or the appropriate solution for your platform, all of which live in `CsProjectSource`).

> If the `.sln` file does not open, ensure that you have a compatible .NET IDE installed.

Once the solution is open, start by editing `Plugin.cs`, located in the **SharedProject** in the solution explorer.

You may also rename existing `ExampleMod` namespaces in the code to whatever fits your mod.

## Project Structure

All C# projects and the shared MSBuild configuration live in the `CsProjectSource` folder:

- **Server-side code**: `CsProjectSource/ServerProject/ServerSource`
- **Client-side code**: `CsProjectSource/ClientProject/ClientSource`
- **Shared code** (used by both client and server): `CsProjectSource/SharedProject/SharedSource`
- **Build tooling**: `CsProjectSource/PluginToolbox`


Within the **SharedProject**, you can use the `CLIENT` and `SERVER` preprocessor symbols to conditionally compile code for the client or the server as needed.

```cs
#if SERVER
DebugConsole.NewMessage("Hello from server");
#endif

#if CLIENT
DebugConsole.NewMessage("Hello from client");
#endif
```

### Content

All mod content (XML files, sounds, textures, etc.) is recommended be placed in:

- `Content`

If you add new XML content, make sure to update the file list located at:

- `filelist.xml`

### Build Output

Compiled assemblies are copied to:

- `bin/<Platform>/<Client|Server>/<ModName><Client|Server>.dll`

`PluginInfo.xml` is regenerated on every build from `ModName`, pointing at those assemblies with `%ModDir%` paths, so it never needs to be edited by hand:

```xml
<MainAssemblyInfo assemblypath="%ModDir%/bin/Windows/Client/BaseModClient.dll" platforms="Windows" targets="Client" />
```

> The `name` attribute of `filelist.xml` is kept in sync with `ModName`; everything else in that file is preserved.

> If you rename `ModName` later, delete the `bin` folder once, otherwise the assemblies built under the old name stay behind.

## Testing Your Mod

<img width="1920" height="1032" alt="image" src="https://github.com/user-attachments/assets/56ba8402-7f1e-4f36-a28b-bca48c785c17" />

You can test your mod by pressing the "Launch Barotrauma" button on top, which will automatically launch Barotrauma with the debugger attached and your plugin built. 

Building the plugin writes the assemblies straight into `bin` inside this folder and updates `PluginInfo.xml` to match, so there is no separate deploy step - the mod can be enabled in-game right away.

You can also manually right-click **WindowsClient** and **WindowsServer** in the solution view, then select **Build**, which does the same process as the above, without the launching Barotrauma part.

> **Warning**  
> This method builds the mod in **Debug** configuration.  
> To properly publish or update your mod, follow the steps below.

## Building for Release

Change the launch option from **Launch Barotrauma** to **Build Release**:

<img width="1920" height="1032" alt="image" src="https://github.com/user-attachments/assets/94c8e5f1-2438-4d81-97c7-2c4ca7fc1d39" />

Now simply click **Build Release** and it should start building your plugin in release mode.

## Before Release

Before publishing, mark the `CsProjectSource` folder as hidden (Windows: right click → **Properties** → **Hidden**), so the project sources are not shipped together with the mod.

## Hot Reload

.NET supports Hot Reload, which lets you apply certain code changes while Barotrauma is running, without rebuilding the mod or restarting the game. This is useful for quickly iterating on gameplay logic.

To use Hot Reload in Visual Studio, launch Barotrauma with your plugin enabled. If your IDE is not attached to the Barotrauma process, you can attach the debugger to the running `Barotrauma.exe` process using **Debug → Attach to Process**.

Once attached, make changes to your C# code and apply them using **Hot Reload** (toolbar button) or **Alt + F10**. If the change is supported, it will be applied immediately and you can test it in-game.

Hot Reload works best for changes inside existing methods. Structural changes such as adding new classes, changing method signatures, or modifying fields usually require reloading the entire mod.
