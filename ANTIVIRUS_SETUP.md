# Antivirus Scanner Setup Guide

## Overview
The antivirus scanner is integrated into the Messages page (Antivirus tab). It:
- Reads a configuration file (`1_version.txt`) that lists threats
- Scans and deletes malicious files and folders
- Removes suspicious registry keys
- Removes malicious scheduled tasks
- Downloads the latest configuration from GitHub

## Quick Start

### 1. Configuration File Format

The `1_version.txt` file should be placed at: `C:\Users\Abeken_A\Desktop\Antivirus BIbon\1_version.txt`

File format (supports Kazakh/Russian/English):
```
осы жердегін тазарту керек
Компьютер\HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender\Exclusions\Paths
Компьютер\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run\WinSysCache

Өшіру керек файлдар
C:\ProgramData\Microsoft\Windows\malware.exe
C:\Windows\System32\suspicious.exe

Осы папкаларды өшіру керек
C:\Users\Abeken_A\AppData\Local\Microsoft\Windows\Caches\AAADE3DC
C:\ProgramData\7979061C

Планировщик заданий очистит есту керек taskschd.msc
ProxiesPeer
Windows System Health
WU_1882d487
```

### 2. Configuration Sections

- **Registry Keys**: Lines starting with `HKEY_` or `Компьютер\HKEY_`
- **Files to Delete**: Lines starting with `C:\` under "Өшіру керек файлдар" section
- **Folders to Delete**: Lines starting with `C:\` under "Осы папкаларды өшіру керек" section
- **Scheduled Tasks**: Lines under "Планировщик заданий" section

### 3. GitHub Setup

#### Step 1: Create GitHub Repository
1. Create a new repository on GitHub (e.g., `antivirus-bibon`)
2. Upload your `1_version.txt` file
3. Commit and push to `main` branch

#### Step 2: Update URL in Code
In `Pages\Messages.xaml.cs`, line 164, change:
```csharp
var githubUrl = "https://raw.githubusercontent.com/YourUsername/YourRepo/main/1_version.txt";
```

Replace:
- `YourUsername` - Your GitHub username
- `YourRepo` - Your repository name

Example:
```csharp
var githubUrl = "https://raw.githubusercontent.com/abeken47/antivirus-bibon/main/1_version.txt";
```

### 4. Using the Antivirus Scanner

1. **Open the Antivirus tab** in the application
2. **Check Updates** button:
   - Downloads the latest configuration from GitHub
   - Saves to `C:\Users\Abeken_A\Desktop\Antivirus BIbon\1_version.txt`
   - Updates the database version timestamp

3. **Scan** button:
   - Reads the configuration file
   - Scans for files and folders listed in config
   - Deletes detected threats
   - Shows progress with:
     - Animated arc around shield
     - Percentage counter
     - Current item being scanned
     - Final statistics: Files Scanned, Threats Found, Quarantined

### 5. Scan Modes (Currently All Use Same Logic)

- **Quick Scan**: Scans all configured paths (default)
- **Full Scan**: Scans all configured paths
- **Custom Scan**: Scans all configured paths

All modes use the same configuration file. Future versions can extend each mode to scan different path sets.

## API Reference

### AntivirusService

#### LoadConfigFromFile(string filePath)
Loads and parses the configuration file.

```csharp
var config = AntivirusService.LoadConfigFromFile(configPath);
```

Returns: `ScanConfig` object with:
- `RegistryKeys` - List of registry paths to remove
- `FilesToDelete` - List of file paths to delete
- `FoldersToDelete` - List of folder paths to delete
- `ScheduledTasksToRemove` - List of task names to remove

#### PerformScan(ScanConfig config, Action<double, string> progressCallback)
Executes the scan and returns results.

```csharp
var result = AntivirusService.PerformScan(config, (percent, item) =>
{
    Debug.WriteLine($"Progress: {percent}% - {item}");
});
```

Returns: `ScanResult` object with:
- `FilesScanned` - Total items checked
- `ThreatsFound` - Total threats detected
- `QuarantinedCount` - Successfully deleted items
- `ThreatsDetected` - List of threat paths

#### RemoveRegistryKeys(ScanConfig config)
Removes registry keys specified in config.

```csharp
AntivirusService.RemoveRegistryKeys(config);
```

#### RemoveScheduledTasks(ScanConfig config)
Removes Windows scheduled tasks.

```csharp
AntivirusService.RemoveScheduledTasks(config);
```

## Extending the Scanner

### Add Quick Scan Specific Paths
Modify `btnScan_Click` to use different configs based on scan mode:

```csharp
var scanMode = rbQuickScan.IsChecked.Value ? "quick" : 
               rbFullScan.IsChecked.Value ? "full" : "custom";

var configFile = scanMode == "quick" 
    ? Path.Combine(configDir, "quick_scan.txt")
    : Path.Combine(configDir, "1_version.txt");

var config = AntivirusService.LoadConfigFromFile(configFile);
```

### Add Custom Path Selection
Store selected paths and create dynamic config:

```csharp
var config = new AntivirusService.ScanConfig();
config.FilesToDelete.AddRange(userSelectedFiles);
config.FoldersToDelete.AddRange(userSelectedFolders);

var result = AntivirusService.PerformScan(config, progressCallback);
```

## Security Notes

1. **Admin Privileges**: Some operations (registry, scheduled tasks) may require admin rights
2. **Backup**: Always backup important files before running scans on critical systems
3. **Whitelisting**: Carefully review threat lists before applying to production
4. **Update Frequency**: Check GitHub for updates regularly

## Troubleshooting

### "Configuration file not found"
- Ensure `C:\Users\Abeken_A\Desktop\Antivirus BIbon\1_version.txt` exists
- Click "Check Updates" to download from GitHub

### "Failed to download configuration"
- Verify GitHub URL is correct in code
- Check internet connection
- Ensure repository is public or authenticated

### Files not deleted
- Check if running with admin privileges
- Verify files are not in use by other processes
- Check permissions on file/folder

### Registry keys not removed
- May require admin privileges
- Verify registry path format is correct
- Check if registry key exists before deletion

## Example: Full Workflow

1. **Setup GitHub repo**
   - Create `antivirus-bibon` repo
   - Add `1_version.txt` with threat definitions
   - Get raw file URL

2. **Update application code**
   - Edit `Messages.xaml.cs`
   - Set GitHub URL to your repository
   - Rebuild application

3. **Use the scanner**
   - Open Antivirus tab
   - Click "Check Updates" to download latest config
   - Click "Scan" to detect and remove threats
   - Review results

4. **Maintain threats list**
   - Keep `1_version.txt` updated on GitHub
   - Add new threats as they're discovered
   - Commit changes regularly
