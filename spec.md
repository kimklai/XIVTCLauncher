# XIVTCLauncher Directory Spec

## Project Summary

XIVTCLauncher, also named FFXIVSimpleLauncher in the project file and code namespace, is a .NET 8 WPF launcher for the Taiwan version of Final Fantasy XIV. It provides account login, OTP support, game update checking and patch installation, launcher update checking, and optional Dalamud setup/injection.

The application uses an MVVM-style layout:

- `Views/` contains WPF windows and XAML UI.
- `ViewModels/` contains UI state and command orchestration.
- `Services/` contains login, update, credential, runtime, Dalamud, and settings logic.
- `Models/` contains persisted settings and data transfer models.
- `Patching/` contains patch file parsing and installation support.

## Build And Runtime

- Project file: `FFXIVSimpleLauncher.csproj`
- Output type: `WinExe`
- Target framework: `net8.0-windows`
- UI framework: WPF
- Nullable reference types: enabled
- Implicit usings: enabled
- Application icon: `Resources/app.ico`
- Current project version: `1.14.17`

NuGet dependencies:

- `CommunityToolkit.Mvvm` for MVVM source generators and commands.
- `MaterialDesignThemes` for WPF styling.
- `Microsoft.Web.WebView2` for embedded web login.
- `Newtonsoft.Json` for JSON handling.
- `SevenZipExtractor` for archive extraction support.

## Top-Level Files

- `App.xaml` and `App.xaml.cs`: WPF application entry point and resource setup.
- `AssemblyInfo.cs`: assembly-level WPF/theme metadata.
- `FFXIVSimpleLauncher.csproj`: project definition, framework, package references, and version metadata.
- `README.md`: Traditional Chinese user documentation.
- `README_en.md`: English user documentation.
- `.gitignore`: ignores build outputs, Visual Studio/Rider local files, packages, zip releases, and local Claude metadata.
- `XIVTCLauncher-v1.14.18.zip`: local release archive; ignored by `.gitignore`.

## Directory Map

### `Views/`

WPF view layer. Each window has a `.xaml` layout file and a `.xaml.cs` code-behind file.

- `MainWindow`: main launcher UI.
- `SettingsWindow`: game path, Dalamud, OTP, and other settings.
- `WebLoginWindow`: embedded WebView2 login flow.
- `OtpDialog`: OTP prompt/input window.
- `AccountManagementWindow`: account management UI.

### `ViewModels/`

View models for the UI.

- `MainViewModel.cs`: central launcher workflow. It loads settings, checks updates, manages selected accounts, opens settings/account windows, performs login, starts game updates, and coordinates Dalamud injection.
- `AccountManagementViewModel.cs`: account list and account editing workflow.

### `Services/`

Application service layer.

- `SettingsService.cs`: loads and saves launcher settings.
- `AccountService.cs`: manages configured accounts and selected account state.
- `CredentialService.cs`: stores and retrieves secrets via Windows Credential Manager.
- `OtpService.cs`: TOTP generation and OTP secret management.
- `LoginService.cs`: talks to Taiwan FFXIV launcher login/session APIs and starts `ffxiv_dx11.exe` with Taiwan launch arguments.
- `GamePathDetector.cs`: attempts to discover the installed Taiwan FFXIV path.
- `GameUpdateService.cs`: checks official patch server state, downloads required patches, installs them, and updates local version files.
- `PatchListParser.cs`: reads local game and expansion version metadata.
- `PatchInstaller.cs`: applies downloaded patch files to the game directory.
- `LauncherUpdateService.cs`: checks GitHub releases and downloads launcher updates.
- `DalamudService.cs`: validates/downloads Dalamud, prepares runtime/assets, launches the game, and injects Dalamud.
- `DalamudDownloader.cs`: downloads Dalamud releases.
- `DotNetRuntimeManager.cs`: downloads and validates the .NET runtime required by Dalamud.

### `Models/`

Shared data models.

- `LauncherSettings.cs`: persisted global settings, Dalamud mode, game path, selected account ID, and legacy migration fields.
- `Account.cs`: account profile data, OTP flags, remember-password options, and metadata.
- `PatchInfo.cs`: patch metadata such as size, version, repository, URL, and local path helpers.

### `Converters/`

Small WPF binding converters.

- `BoolToStringConverter.cs`
- `InvertBoolConverter.cs`

### `Dalamud/`

Models used when reading Dalamud and asset metadata.

- `AssetInfo.cs`
- `DalamudVersionInfo.cs`

### `Patching/`

Patch installation implementation. This directory contains general utility classes and a ZiPatch parser/handler stack.

Important areas:

- `Patching/Util/`: binary readers, CRC32, deflate stream support, multipart response handling, and reusable buffers.
- `Patching/ZiPatch/`: core ZiPatch file/config/exception types.
- `Patching/ZiPatch/Chunk/`: patch chunk implementations.
- `Patching/ZiPatch/Chunk/SqpkCommand/`: SQPK command chunks for add/delete/expand/header/index/file operations.
- `Patching/ZiPatch/Util/`: SQPK dat/index/file stream helpers.

### `Resources/`

Application resources.

- `app.ico`: Windows application icon.

### `cdn/`

Local CDN-like metadata used by the launcher.

- `cdn/dalamud/latest-release.json`: Dalamud release metadata cache or fixture.

### `Mods/`

Untracked local plugin payloads. At the time this spec was written, this directory contained Penumbra and ReMakePlacePlugin binaries and metadata. Because it contains third-party binary artifacts and is currently untracked, it should be reviewed before adding to Git.

### Generated Or Removed Build Output

The repository previously had tracked `publish/` and `release/` output directories. These were removed from Git because build and release artifacts are generated outputs. The `.gitignore` already ignores `bin/`, `obj/`, `out/`, `publish/`, and `*.zip`.

## Main Runtime Flow

1. `MainViewModel` loads settings through `SettingsService`.
2. Account data is loaded and the selected account is restored.
3. Startup update checks run:
   - launcher update check via `LauncherUpdateService`
   - game update check via `GameUpdateService`
4. If the game path is missing, the launcher attempts detection with `GamePathDetector` and opens `SettingsWindow`.
5. Login opens `WebLoginWindow`, using saved credentials and optional account-specific OTP.
6. `LoginService` obtains a launcher session and starts the game.
7. If Dalamud is enabled, `DalamudService` prepares Dalamud, runtime, and assets before launching/injecting.
8. If game patches are required, login is blocked until `GameUpdateService` downloads and installs them.

## External Services And Endpoints

The launcher interacts with several external services:

- Taiwan FFXIV login API:
  - `https://user.ffxiv.com.tw/api/login/launcherLogin`
  - `https://user.ffxiv.com.tw/api/login/launcherSession`
- Taiwan FFXIV patch version API:
  - `http://patch-gamever.ffxiv.com.tw/http/win32/ffxivtc_release_tc_game/{version}/`
- Taiwan FFXIV patch downloads:
  - `http://patch-dl.ffxiv.com.tw/...`
- Dalamud assets:
  - `https://raw.githubusercontent.com/yanmucorp/DalamudAssets/master/assetCN.json`
- GitHub releases for launcher and Dalamud update checks/downloads.

## Local User Data

Runtime data is stored outside the repository:

- Launcher settings: `%APPDATA%/FFXIVSimpleLauncher/settings.json`
- Patch downloads: `%APPDATA%/FFXIVSimpleLauncher/Patches`
- Dalamud data: `%APPDATA%/FFXIVSimpleLauncher/Dalamud`
- Passwords and OTP secrets: Windows Credential Manager

## Git State Notes

At the time this spec was created:

- `main` was aligned with `origin/main`.
- `Mods/` existed as an untracked local directory.
- `publish/` and `release/` generated outputs had been removed from version control.
