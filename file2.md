### Define source and destination folders
```
$sFolder = "D:\temp\VMware-tools-12.4.0-23259341-x86_64"
$dFolder = "\\hq-dc19-01\D$\MCS"
```
</br></br></br>
# Copy the folder and its contents to the remote computer
Copy-Item -Path $sFolder -Destination $dFolder -Recurse

# Get a list of domain controllers
$dcs = Get-ADDomainController -Filter * | Select-Object -ExpandProperty Name | Sort-Object -Descending

# Install or check VMware Tools on each domain controller
foreach ($dc in $dcs) {
    try {
        # Define the path to the VMware Tools installer
        $installerPath = Join-Path $dFolder "VMware-tools-12.4.0-23259341-x86_64\VMware-tools-12.4.0-23259341-x86_64.exe"

        # Check if VMware Tools is already installed
        $installed = Get-WmiObject Win32_Product -ComputerName $dc | Where-Object { $_.Name -eq "VMware Tools" }

        if ($installed) {
            Write-Host "VMware Tools is already installed on $dc"
        } else {
            # Install VMware Tools silently
            Write-Host "Installing VMware Tools on $dc"
            Invoke-Command -ComputerName $dc -ScriptBlock {
                Start-Process -Wait -FilePath $using:installerPath -ArgumentList '/s'
            }
        }

        # Check the installed version
        $result = Invoke-Command -ComputerName $dc -ScriptBlock {
            Get-WmiObject Win32_Product | Where-Object { $_.Name -eq "VMware Tools" } | Select-Object -Property Version
        }
        $installed_version = $result.Version

        # Check if the installed version matches 12.4.0.23
        if ($installed_version -eq "12.4.0.23") {
            Write-Host "VMware Tools version 12.4.0.23 is installed on $dc"
        } else {
            Write-Host "VMware Tools is installed on $dc, but the version is $installed_version"
        }
    } catch {
        Write-Host "Error checking or installing VMware Tools on $dc"
    }
}

# Print a success message
Write-Host "VMware Tools check completed for all domain controllers."
