# IDM EXECUTABLE FILE IS DAMAGED FIXEDDD!!!!!!!
The main idm executive file is Damage!! FIXED

The main idm executive file is Damage!! FIXED


# IDM Auto-Dismiss

Automatically unlocks and clicks **"Continue downloading"** on Internet Download Manager's *"The main IDM executive file is damaged"* countdown dialog.

No file patching. No registry edits. 100% reversible.

---

### Install
Open **PowerShell**, paste this, and press **Enter**:

```powershell
$dir = "$env:LOCALAPPDATA\IDM-AutoDismiss"
$file = "$dir\IDM-AutoDismiss.ps1"
New-Item -Path $dir -ItemType Directory -Force | Out-Null

@'
Add-Type @"
using System;
using System.Text;
using System.Runtime.InteropServices;

public class Win32IDM {
    public delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);
    [DllImport("user32.dll")] public static extern bool EnumWindows(EnumWindowsProc enumProc, IntPtr lParam);
    [DllImport("user32.dll")] public static extern bool EnumChildWindows(IntPtr hWndParent, EnumWindowsProc enumProc, IntPtr lParam);
    [DllImport("user32.dll", CharSet = CharSet.Auto)] public static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
    [DllImport("user32.dll", CharSet = CharSet.Auto)] public static extern int GetClassName(IntPtr hWnd, StringBuilder lpClassName, int nMaxCount);
    [DllImport("user32.dll")] public static extern bool EnableWindow(IntPtr hWnd, bool bEnable);
    [DllImport("user32.dll")] public static extern IntPtr SendMessage(IntPtr hWnd, uint Msg, IntPtr wParam, IntPtr lParam);
    public const uint BM_CLICK = 0x00F5;
}
"@

function Get-Text($h) {$s = New-Object System.Text.StringBuilder 512; [void][Win32IDM]::GetWindowText($h,$s, $s.Capacity);$s.ToString() }
function Get-Class($h) {$s = New-Object System.Text.StringBuilder 256; [void][Win32IDM]::GetClassName($h,$s, $s.Capacity);$s.ToString() }

while ($true) {$cb = [Win32IDM+EnumWindowsProc]{
        param($hwnd,$lParam)
        if ((Get-Text $hwnd) -eq "Internet Download Manager") {
            $script:hit =$false; $script:btn = [IntPtr]::Zero$childCb = [Win32IDM+EnumWindowsProc]{
                param($ch,$cl)
                $t = Get-Text$ch; $c = Get-Class$ch
                if ($t -match "damaged|executive file") { $script:hit =$true }
                if ($c -eq "Button" -and $t -match "Continue downloading") { $script:btn =$ch }
                return $true
            }
            [void][Win32IDM]::EnumChildWindows($hwnd,$childCb, [IntPtr]::Zero)
            if ($script:hit -and$script:btn -ne [IntPtr]::Zero) {
                [void][Win32IDM]::EnableWindow($script:btn,$true)
                [void][Win32IDM]::SendMessage($script:btn, [Win32IDM]::BM_CLICK, [IntPtr]::Zero, [IntPtr]::Zero)
            }
        }
        return $true
    }
    [void][Win32IDM]::EnumWindows($cb, [IntPtr]::Zero)
    Start-Sleep -Milliseconds 300
}
'@ | Set-Content -Path $file -Encoding UTF8

$action  = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -WindowStyle Hidden -ExecutionPolicy Bypass -File `"$file`""
$trigger = New-ScheduledTaskTrigger -AtLogOn -User$env:USERNAME
Register-ScheduledTask -TaskName "IDM_AutoDismiss" -Action $action -Trigger$trigger -Force | Out-Null
Start-ScheduledTask -TaskName "IDM_AutoDismiss"
Write-Host "Installed and running in the background." -ForegroundColor Green
```

---

### Uninstall
Open **PowerShell**, paste this, and press **Enter**:

```powershell
Stop-ScheduledTask -TaskName "IDM_AutoDismiss" -ErrorAction SilentlyContinue
Unregister-ScheduledTask -TaskName "IDM_AutoDismiss" -Confirm:$false -ErrorAction SilentlyContinue
Get-CimInstance Win32_Process -Filter "CommandLine LIKE '%IDM-AutoDismiss.ps1%'" -ErrorAction SilentlyContinue | ForEach-Object { Stop-Process -Id $_.ProcessId -Force -ErrorAction SilentlyContinue }
Remove-Item -Path "$env:LOCALAPPDATA\IDM-AutoDismiss" -Recurse -Force -ErrorAction SilentlyContinue
Write-Host "Completely removed." -ForegroundColor Green
```
