# IDM EXECUTABLE FILE IS DAMAGED FIXEDDD!!!!!!!
The main idm executive file is Damage!! FIXED

The main idm executive file is Damage!! FIXED



<#
.SYNOPSIS
    Automatically enables and clicks the "Continue downloading" button on the
    Internet Download Manager (IDM) "main IDM executive file is damaged" dialog.

.DESCRIPTION
    Runs a lightweight background loop using the Windows API (user32.dll) to detect
    the IDM warning dialog, force-enable the disabled countdown button, and click it.
    Does not modify IDM binaries, system files, or the Windows Registry.

.PARAMETER Install
    Copies the script to %LOCALAPPDATA%\IDM-AutoDismiss and registers a hidden
    Scheduled Task to run automatically at user logon.

.PARAMETER Uninstall
    Stops the background process, removes the Scheduled Task, and deletes the
    installed script.

.EXAMPLE
    .\IDM-AutoDismiss.ps1 -Install
    .\IDM-AutoDismiss.ps1 -Uninstall
    .\IDM-AutoDismiss.ps1
#>

[CmdletBinding()]
param (
    [switch]$Install,
    [switch]$Uninstall
)

$TaskName   = "IDM_AutoDismiss"
$InstallDir = Join-Path$env:LOCALAPPDATA "IDM-AutoDismiss"
$ScriptDest = Join-Path$InstallDir "IDM-AutoDismiss.ps1"

function Stop-WatcherProcess {
    Get-CimInstance Win32_Process -Filter "CommandLine LIKE '%IDM-AutoDismiss.ps1%'" -ErrorAction SilentlyContinue |
        Where-Object { $_.ProcessId -ne$PID } |
        ForEach-Object { Stop-Process -Id $_.ProcessId -Force -ErrorAction SilentlyContinue }
}

if ($Uninstall) {
    Stop-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue
    Unregister-ScheduledTask -TaskName $TaskName -Confirm:$false -ErrorAction SilentlyContinue
    Stop-WatcherProcess
    if (Test-Path $InstallDir) {
        Remove-Item -Path $InstallDir -Recurse -Force -ErrorAction SilentlyContinue
    }
    Write-Host "IDM-AutoDismiss has been completely removed." -ForegroundColor Green
    exit 0
}

if ($Install) {
    Stop-ScheduledTask -TaskName $TaskName -ErrorAction SilentlyContinue
    Stop-WatcherProcess

    if (-not (Test-Path $InstallDir)) {
        New-Item -Path $InstallDir -ItemType Directory -Force | Out-Null
    }
    Copy-Item -Path $PSCommandPath -Destination$ScriptDest -Force

    $action  = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File `"$ScriptDest`""
    $trigger = New-ScheduledTaskTrigger -AtLogOn -User$env:USERNAME
    Register-ScheduledTask -TaskName $TaskName -Action $action -Trigger$trigger -Description "Auto-dismisses the IDM damaged executive file dialog." -Force | Out-Null
    Start-ScheduledTask -TaskName $TaskName

    Write-Host "IDM-AutoDismiss installed and running in the background." -ForegroundColor Green
    exit 0
}

# --- Main Watcher Loop ---

Add-Type @"
using System;
using System.Text;
using System.Runtime.InteropServices;

public class Win32IDM {
    public delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);

    [DllImport("user32.dll")]
    public static extern bool EnumWindows(EnumWindowsProc enumProc, IntPtr lParam);

    [DllImport("user32.dll")]
    public static extern bool EnumChildWindows(IntPtr hWndParent, EnumWindowsProc enumProc, IntPtr lParam);

    [DllImport("user32.dll", CharSet = CharSet.Auto)]
    public static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);

    [DllImport("user32.dll", CharSet = CharSet.Auto)]
    public static extern int GetClassName(IntPtr hWnd, StringBuilder lpClassName, int nMaxCount);

    [DllImport("user32.dll")]
    public static extern bool EnableWindow(IntPtr hWnd, bool bEnable);

    [DllImport("user32.dll")]
    public static extern IntPtr SendMessage(IntPtr hWnd, uint Msg, IntPtr wParam, IntPtr lParam);

    public const uint BM_CLICK = 0x00F5;
}
"@

function Get-WindowTextSafe([IntPtr]$hwnd) {
    $sb = New-Object System.Text.StringBuilder 512 [void][Win32IDM]::GetWindowText($hwnd, $sb,$sb.Capacity)
    return $sb.ToString()
}

function Get-ClassNameSafe([IntPtr]$hwnd) {
    $sb = New-Object System.Text.StringBuilder 256 [void][Win32IDM]::GetClassName($hwnd, $sb,$sb.Capacity)
    return $sb.ToString()
}
