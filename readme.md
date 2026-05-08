<div style="margin: 20px; line-height: 1.8; font-family: sans-serif;">

## Steps to Convert Win 11 LTSC Eval Version to Full Version


1. Download Evaluation folder: https://archive.org/details/enterprise_SKU


2. Copy Evaluation Folder to: `C:\Windows\System32\spp\tokens\skus`


3. Run these commands in CMD as administrator:


```text
cd C:\Windows\System32\spp\tokens\skus  
cscript.exe %windir%\system32\slmgr.vbs /rilc
cscript.exe %windir%\system32\slmgr.vbs /upk >nul 2>&1
cscript.exe %windir%\system32\slmgr.vbs /ckms >nul 2>&1
cscript.exe %windir%\system32\slmgr.vbs /cpky >nul 2>&1
cscript.exe %windir%\system32\slmgr.vbs /ipk NPPR9-FWDCX-D2C8J-H872K-2YT43
sc config LicenseManager start= auto & net start LicenseManager
sc config wuauserv start= auto & net start wuauserv
```


## Activate Windows/Office (Powershell)


```text
irm https://get.activated.win | iex
```


## Disable Search Highlights (Powershell)


```text
if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]"Administrator")) {
    Write-Error "ERROR: Must be run as Administrator."; exit 1
}
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\SearchSettings" /v IsDynamicSearchBoxEnabled /t REG_DWORD /d 0 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v DisableWebSearch /t REG_DWORD /d 1 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v ConnectedSearchUseWeb /t REG_DWORD /d 0 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v ConnectedSearchUseWebOverMeteredConnections /t REG_DWORD /d 0 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v AllowCortana /t REG_DWORD /d 0 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v AllowCortanaAboveLock /t REG_DWORD /d 0 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v AllowSearchToUseLocation /t REG_DWORD /d 0 /f | Out-Null
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v DoNotUseWebResults /t REG_DWORD /d 1 /f | Out-Null
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Search" /v BingSearchEnabled /t REG_DWORD /d 0 /f | Out-Null
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Search" /v CortanaConsent /t REG_DWORD /d 0 /f | Out-Null
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Search" /v AllowSearchToUseLocation /t REG_DWORD /d 0 /f | Out-Null
reg add "HKCU\Software\Policies\Microsoft\Windows\Explorer" /v DisableSearchBoxSuggestions /t REG_DWORD /d 1 /f | Out-Null
taskkill /f /im explorer.exe 2>$null
Start-Sleep -Seconds 2
Start-Process explorer.exe
Write-Host "Done. Web search in Windows Search has been disabled." -ForegroundColor Green
```


## Disable Updates Permanently


Run the following script as Administrator:


```text
if (-not ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]"Administrator")) {
    Write-Error "ERROR: Must be run as Administrator."; exit 1
}
$services = @("wuauserv", "bits", "usosvc", "WaaSMedicSvc", "dosvc", "UsoSvc", "wuauclt", "UpdateOrchestrator")
foreach ($svc in $services) { Stop-Service -Name $svc -Force -ErrorAction SilentlyContinue }
$serviceKeys = @(
    "HKLM:\SYSTEM\CurrentControlSet\Services\wuauserv",
    "HKLM:\SYSTEM\CurrentControlSet\Services\bits",
    "HKLM:\SYSTEM\CurrentControlSet\Services\usosvc",
    "HKLM:\SYSTEM\CurrentControlSet\Services\dosvc",
    "HKLM:\SYSTEM\CurrentControlSet\Services\UsoSvc"
)
foreach ($key in $serviceKeys) { if (Test-Path $key) { Set-ItemProperty -Path $key -Name "Start" -Value 4 -ErrorAction SilentlyContinue } }
Add-Type @"
using System;
using System.Runtime.InteropServices;
using Microsoft.Win32;
public class RegOwnership {
    [DllImport("advapi32.dll", SetLastError=true)]
    public static extern int RegOpenKeyEx(UIntPtr hKey, string subKey, int options, int samDesired, out UIntPtr phkResult);
    [DllImport("advapi32.dll", SetLastError=true)]
    public static extern int RegSetKeySecurity(UIntPtr hKey, int secInfo, byte[] pSecDesc);
    [DllImport("advapi32.dll", SetLastError=true)]
    public static extern int RegCloseKey(UIntPtr hKey);
}
"@ -ErrorAction SilentlyContinue
$medicKeyPS = "HKLM:\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc"
try {
    $sid = New-Object System.Security.Principal.SecurityIdentifier("S-1-5-32-544")
    $account = $sid.Translate([System.Security.Principal.NTAccount])
    $key = [Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("SYSTEM\CurrentControlSet\Services\WaaSMedicSvc",[Microsoft.Win32.RegistryKeyPermissionCheck]::ReadWriteSubTree,[System.Security.AccessControl.RegistryRights]::TakeOwnership)
    if ($key) {
        $acl = $key.GetAccessControl()
        $acl.SetOwner($account)
        $key.SetAccessControl($acl)
        $key2 = [Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("SYSTEM\CurrentControlSet\Services\WaaSMedicSvc",[Microsoft.Win32.RegistryKeyPermissionCheck]::ReadWriteSubTree,[System.Security.AccessControl.RegistryRights]::ChangePermissions)
        $acl2 = $key2.GetAccessControl()
        $rule = New-Object System.Security.AccessControl.RegistryAccessRule($account,"FullControl","ContainerInherit,ObjectInherit","None","Allow")
        $acl2.SetAccessRule($rule)
        $key2.SetAccessControl($acl2)
        $key2.Close()
        $key.Close()
        Set-ItemProperty -Path $medicKeyPS -Name "Start" -Value 4 -Force
    }
} catch {
    reg add "HKLM\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc" /v Start /t REG_DWORD /d 4 /f | Out-Null
}
$wuPolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate"
$auPolicyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
New-Item -Path $wuPolicyPath -Force | Out-Null
New-Item -Path $auPolicyPath -Force | Out-Null
Set-ItemProperty -Path $auPolicyPath -Name "NoAutoUpdate" -Type DWord -Value 1
Set-ItemProperty -Path $auPolicyPath -Name "AUOptions" -Type DWord -Value 1
Set-ItemProperty -Path $auPolicyPath -Name "NoAutoRebootWithLoggedOnUsers" -Type DWord -Value 1
Set-ItemProperty -Path $auPolicyPath -Name "ScheduledInstallDay" -Type DWord -Value 0
Set-ItemProperty -Path $auPolicyPath -Name "ScheduledInstallTime" -Type DWord -Value 3
Set-ItemProperty -Path $wuPolicyPath -Name "DisableWindowsUpdateAccess" -Type DWord -Value 1
Set-ItemProperty -Path $wuPolicyPath -Name "DoNotConnectToWindowsUpdateInternetLocations" -Type DWord -Value 1
Set-ItemProperty -Path $wuPolicyPath -Name "DisableDualScan" -Type DWord -Value 1
Set-ItemProperty -Path $wuPolicyPath -Name "ExcludeWUDriversInQualityUpdate" -Type DWord -Value 1
Set-ItemProperty -Path $wuPolicyPath -Name "WUServer" -Type String -Value " "
Set-ItemProperty -Path $wuPolicyPath -Name "WUStatusServer" -Type String -Value " "
Set-ItemProperty -Path $wuPolicyPath -Name "UpdateServiceUrlAlternate" -Type String -Value " "
$hostsPath = "$env:SystemRoot\System32\drivers\etc\hosts"
$wuDomains = @("windowsupdate.microsoft.com","update.microsoft.com","download.windowsupdate.com","wustat.windows.com","ntservicepack.microsoft.com","geo-prod.do.dsp.mp.microsoft.com")
$hostsContent = Get-Content $hostsPath -ErrorAction SilentlyContinue
foreach ($domain in $wuDomains) { $entry = "0.0.0.0 $domain"; if ($hostsContent -notcontains $entry) { Add-Content -Path $hostsPath -Value $entry } }
$tasks = @(
    "\Microsoft\Windows\WindowsUpdate\Scheduled Start",
    "\Microsoft\Windows\WindowsUpdate\ntu",
    "\Microsoft\Windows\UpdateOrchestrator\Schedule Scan",
    "\Microsoft\Windows\UpdateOrchestrator\Schedule Maintenance Work",
    "\Microsoft\Windows\UpdateOrchestrator\Schedule Wake To Work",
    "\Microsoft\Windows\UpdateOrchestrator\USO_UxBroker",
    "\Microsoft\Windows\UpdateOrchestrator\Start Oobe Expedited",
    "\Microsoft\Windows\UpdateOrchestrator\StartInstallScheduled",
    "\Microsoft\Windows\UpdateOrchestrator\Report policies",
    "\Microsoft\Windows\UpdateOrchestrator\UpdateModelTask",
    "\Microsoft\Windows\UpdateOrchestrator\UUS Failover Task",
    "\Microsoft\Windows\WaaSMedic\PerformRemediation",
    "\Microsoft\Windows\WaaSMedic\WaaSMedicCapsule",
    "\Microsoft\Windows\InstallService\ScanForUpdates",
    "\Microsoft\Windows\InstallService\ScanForUpdatesAsUser",
    "\Microsoft\Windows\InstallService\SmartRetry"
)
foreach ($task in $tasks) { schtasks /Change /TN $task /Disable 2>$null }
$fwRuleNames = @("Block WU wuauclt","Block WU UsoClient")
$fwPrograms = @("$env:SystemRoot\System32\wuauclt.exe","$env:SystemRoot\System32\UsoClient.exe")
for ($i = 0; $i -lt $fwRuleNames.Count; $i++) {
    $existing = Get-NetFirewallRule -DisplayName $fwRuleNames[$i] -ErrorAction SilentlyContinue
    if (-not $existing) { New-NetFirewallRule -DisplayName $fwRuleNames[$i] -Direction Outbound -Action Block -Program $fwPrograms[$i] -Enabled True -Profile Any | Out-Null }
}
$orchPath = "$env:SystemRoot\System32\Tasks\Microsoft\Windows\UpdateOrchestrator"
if (Test-Path $orchPath) {
    try {
        $acl = Get-Acl $orchPath
        $denyRule = New-Object System.Security.AccessControl.FileSystemAccessRule("SYSTEM","Write,Modify","ContainerInherit,ObjectInherit","None","Deny")
        $acl.AddAccessRule($denyRule)
        Set-Acl -Path $orchPath -AclObject $acl
    } catch {}
}
$updateCachePaths = @(
    "$env:SystemRoot\SoftwareDistribution\Download",
    "$env:SystemRoot\SoftwareDistribution\DeliveryOptimization"
)
foreach ($path in $updateCachePaths) { if (Test-Path $path) { Remove-Item -Path "$path\*" -Recurse -Force -ErrorAction SilentlyContinue } }
$actionScriptPath = "C:\Windows\System32\WUKill.ps1"
$killScript = @'
$services = @("wuauserv","bits","usosvc","WaaSMedicSvc","dosvc","UsoSvc")
$updateCachePaths = @("$env:SystemRoot\SoftwareDistribution\Download","$env:SystemRoot\SoftwareDistribution\DeliveryOptimization")
foreach ($svc in $services) { Stop-Service -Name $svc -Force -ErrorAction SilentlyContinue }
foreach ($path in $updateCachePaths) { if (Test-Path $path) { Remove-Item -Path "$path\*" -Recurse -Force -ErrorAction SilentlyContinue } }
$medicKey = "HKLM:\SYSTEM\CurrentControlSet\Services\WaaSMedicSvc"
$startVal = (Get-ItemProperty -Path $medicKey -Name "Start" -ErrorAction SilentlyContinue).Start
if ($startVal -ne 4) { try { Set-ItemProperty -Path $medicKey -Name "Start" -Value 4 -Force -ErrorAction SilentlyContinue } catch {} }
'@
$killScript | Set-Content -Path $actionScriptPath -Force
$wuProcessNames = @("wuauclt","UsoClient","WaaSMedicAgent","TiWorker","wusa")
$scope = New-Object System.Management.ManagementScope("\\.\root\subscription")
$scope.Connect()
foreach ($procName in $wuProcessNames) {
    $filterName = "WUFilter_$procName"
    $consumerName = "WUConsumer_$procName"
    $filterQuery = "SELECT * FROM __InstanceCreationEvent WITHIN 2 WHERE TargetInstance ISA 'Win32_Process' AND TargetInstance.Name = '$procName.exe'"
    try {
        $existingFilter = Get-WMIObject -Namespace "root\subscription" -Class "__EventFilter" -Filter "Name='$filterName'" -ErrorAction SilentlyContinue
        if ($existingFilter) { $existingFilter.Delete() }
        $filterPath = New-Object System.Management.ManagementPath("__EventFilter")
        $filterClass = New-Object System.Management.ManagementClass($scope, $filterPath, $null)
        $filter = $filterClass.CreateInstance()
        $filter["Name"] = $filterName
        $filter["EventNameSpace"] = "root\cimv2"
        $filter["QueryLanguage"] = "WQL"
        $filter["Query"] = $filterQuery
        $filter.Put() | Out-Null
        $existingConsumer = Get-WMIObject -Namespace "root\subscription" -Class "CommandLineEventConsumer" -Filter "Name='$consumerName'" -ErrorAction SilentlyContinue
        if ($existingConsumer) { $existingConsumer.Delete() }
        $consumerPath = New-Object System.Management.ManagementPath("CommandLineEventConsumer")
        $consumerClass = New-Object System.Management.ManagementClass($scope, $consumerPath, $null)
        $consumer = $consumerClass.CreateInstance()
        $consumer["Name"] = $consumerName
        $consumer["CommandLineTemplate"] = "powershell.exe -NonInteractive -WindowStyle Hidden -ExecutionPolicy Bypass -File `"$actionScriptPath`""
        $consumer.Put() | Out-Null
        $existingBinding = Get-WMIObject -Namespace "root\subscription" -Class "__FilterToConsumerBinding" -ErrorAction SilentlyContinue | Where-Object { $_.Filter -like "*$filterName*" }
        if ($existingBinding) { $existingBinding.Delete() }
        $bindingPath = New-Object System.Management.ManagementPath("__FilterToConsumerBinding")
        $bindingClass = New-Object System.Management.ManagementClass($scope, $bindingPath, $null)
        $binding = $bindingClass.CreateInstance()
        $binding["Filter"] = $filter.Path.RelativePath
        $binding["Consumer"] = $consumer.Path.RelativePath
        $binding.Put() | Out-Null
    } catch { Write-Host "WMI subscription failed for $procName : $_" -ForegroundColor Yellow }
}
$scriptPath = "C:\Windows\System32\BlockWindowsUpdate.ps1"
$scriptContent = $MyInvocation.MyCommand.ScriptBlock.ToString()
if ($MyInvocation.MyCommand.Path) {
    Copy-Item -Path $MyInvocation.MyCommand.Path -Destination $scriptPath -Force -ErrorAction SilentlyContinue
} else {
    $scriptContent | Out-File -FilePath $scriptPath -Force -Encoding UTF8
}
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest
$action1 = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NonInteractive -WindowStyle Hidden -ExecutionPolicy Bypass -File `"$scriptPath`""
$trigger1 = New-ScheduledTaskTrigger -AtStartup
$trigger2 = New-ScheduledTaskTrigger -RepetitionInterval (New-TimeSpan -Hours 1) -Once -At (Get-Date)
$settings1 = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries -StartWhenAvailable -ExecutionTimeLimit (New-TimeSpan -Minutes 5)
Register-ScheduledTask -TaskName "BlockWindowsUpdatePersist" -Action $action1 -Trigger $trigger1,$trigger2 -Settings $settings1 -Principal $principal -Force | Out-Null
Write-Host "Done. Restart your PC for full effect." -ForegroundColor Green
```

</div>