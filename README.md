name: ZERORDP

# ZERO RDP - Windows Environment Monitor
# GitHub Actions + Windows + Tailscale
# Untuk lingkungan yang kamu berwenang kelola

on:
  workflow_dispatch:

jobs:
  zero-rdp:
    name: ZERO RDP System Monitor
    runs-on: windows-latest
    timeout-minutes: 10

    steps:
      - name: Initialize ZERO RDP
        shell: pwsh
        run: |
          Write-Host "======================================"
          Write-Host "             ZERO RDP"
          Write-Host "        SYSTEM INITIALIZATION"
          Write-Host "======================================"

          Write-Host "Computer: $env:COMPUTERNAME"
          Write-Host "Workflow ID: $env:GITHUB_RUN_ID"
          Write-Host "Operating System: Windows"
          Write-Host "Start Time: $(Get-Date)"

      - name: Check Windows Information
        shell: pwsh
        run: |
          Write-Host "Checking Windows information..."

          Get-CimInstance Win32_OperatingSystem |
            Select-Object Caption,
                          Version,
                          OSArchitecture,
                          LastBootUpTime |
            Format-List

          Write-Host "Windows information check completed."

      - name: Check Remote Desktop Service
        shell: pwsh
        run: |
          Write-Host "Checking Remote Desktop service..."

          $service = Get-Service `
            -Name TermService `
            -ErrorAction SilentlyContinue

          if ($null -ne $service) {
            Write-Host "Service Name: $($service.Name)"
            Write-Host "Service Status: $($service.Status)"
            Write-Host "Startup Type: $($service.StartType)"
          }
          else {
            Write-Host "Remote Desktop service is unavailable."
          }

      - name: Check RDP Configuration
        shell: pwsh
        run: |
          Write-Host "Checking Remote Desktop configuration..."

          $registryPath = `
            "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server"

          if (Test-Path $registryPath) {
            $settings = Get-ItemProperty `
              -Path $registryPath `
              -Name fDenyTSConnections `
              -ErrorAction SilentlyContinue

            if ($null -ne $settings) {
              Write-Host "RDP configuration found."
              Write-Host "fDenyTSConnections: $($settings.fDenyTSConnections)"
            }
          }

          Write-Host "Configuration check completed."

      - name: Check Network Configuration
        shell: pwsh
        run: |
          Write-Host "Checking network interfaces..."

          Get-NetIPConfiguration |
            Select-Object InterfaceAlias,
                          IPv4Address,
                          IPv4DefaultGateway,
                          DNSServer |
            Format-List

          Write-Host "Network check completed."

      - name: Check Tailscale Installation
        shell: pwsh
        run: |
          Write-Host "Checking Tailscale installation..."

          $tailscalePath = Join-Path `
            $env:ProgramFiles `
            "Tailscale\tailscale.exe"

          if (Test-Path $tailscalePath) {
            Write-Host "Tailscale is installed."

            & $tailscalePath version
          }
          else {
            Write-Host "Tailscale is not installed."
            Write-Host "Install it on a device you control."
          }

      - name: Check Listening TCP Ports
        shell: pwsh
        run: |
          Write-Host "Checking listening TCP ports..."

          Get-NetTCPConnection `
            -State Listen `
            -ErrorAction SilentlyContinue |
            Select-Object -First 20 `
              LocalAddress,
              LocalPort,
              State |
            Format-Table -AutoSize

          Write-Host "Port inspection completed."

      - name: Check Available System Resources
        shell: pwsh
        run: |
          Write-Host "Checking system resources..."

          Get-CimInstance Win32_ComputerSystem |
            Select-Object Name,
                          NumberOfProcessors,
                          TotalPhysicalMemory |
            Format-List

          Get-CimInstance Win32_OperatingSystem |
            Select-Object FreePhysicalMemory,
                          TotalVisibleMemorySize |
            Format-List

          Write-Host "Resource check completed."

      - name: Generate ZERO RDP Status
        shell: pwsh
        run: |
          Write-Host ""
          Write-Host "======================================"
          Write-Host "          ZERO RDP STATUS"
          Write-Host "======================================"

          Write-Host "Workflow: $env:GITHUB_WORKFLOW"
          Write-Host "Run ID: $env:GITHUB_RUN_ID"
          Write-Host "Repository: $env:GITHUB_REPOSITORY"
          Write-Host "Computer: $env:COMPUTERNAME"
          Write-Host "Completed: $(Get-Date)"

          Write-Host "======================================"
          Write-Host "       SYSTEM CHECK COMPLETED"
          Write-Host "======================================"
