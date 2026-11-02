# Running Switch Toolbox on Linux (with Bottles)

[Switch Toolbox](https://github.com/KillzXGaming/Switch-Toolbox/) is a Windows application, but it runs well on Linux using **Bottles**. This guide walks you through setting it up.

## 1. Create a New Bottle

1. Open **Bottles**.
2. Click **+** in the top-left to create a new bottle.
3. Give it a name (e.g. `SwitchModding`).
4. Choose **Custom**, leave Architecture as **64-bit** and Runner as `soda-9.0-1`.
5. Click **Create**.

![Creating a new bottle](images/switch-toolbox-linux/creating-bottle.png)

## 2. Install Dependencies

Once created, click on your new bottle in the list.

1. Scroll down to **Options** → **Dependencies** and click on it.
2. Install the following dependencies in order (this may take a while):
   - `dotnet48`
   - `vcredist2019`
   - `gdiplus`

![dotnet48 installed](images/switch-toolbox-linux/dotnet48-installed.png)

## 3. Install Switch Toolbox

1. Download the latest release of Switch Toolbox from [GitHub](https://github.com/KillzXGaming/Switch-Toolbox/releases).
2. Extract the archive to a location of your choice.
3. In Bottles, under **Quick Actions**, click **Browse** - this opens your bottle's `drive_c` directory.
4. Copy the extracted `Toolbox-Latest` folder into `drive_c/ModTools/`.

## 4. Add a Shortcut & Launch

1. In Bottles, under **Programs**, click **Add Shortcuts...**.
2. Navigate to the `Toolbox-Latest` folder inside `drive_c` and select `Toolbox.exe`.
3. Click the **Play** icon next to Toolbox to run it.

![Ready to launch](images/switch-toolbox-linux/ready-to-launch.png)
