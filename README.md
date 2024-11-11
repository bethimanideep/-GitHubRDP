# Remote Desktop Access via GitHub Actions with Ngrok

This GitHub Action workflow sets up a Windows environment with Remote Desktop Protocol (RDP) access through an Ngrok tunnel. Follow the steps below to configure and use the workflow.

## Steps to Set Up

1. **Add Ngrok Auth Token to Secrets**
   - Go to your GitHub repository settings.
   - Navigate to **Settings > Secrets and variables > Actions**.
   - Click **New repository secret**.
   - Set the name to `NGROK_AUTH_TOKEN` and paste your Ngrok auth token as the value.

2. **Run the Workflow**
   - Trigger the workflow by pushing a commit or manually running it via **Actions > Workflow Dispatch**.
   - Once the workflow completes, go to the workflow logs to retrieve the generated **username** and **password** for the remote desktop session.

3. **Access Remote Desktop**
   - Open [Ngrok Endpoints](https://dashboard.ngrok.com/endpoints).
   - Copy the generated TCP URL for the RDP connection.
   - Use the URL in your Remote Desktop client to access the environment.

---

## Workflow File

```yaml
name: Remote Desktop Access with Ngrok

on: [push, workflow_dispatch]

jobs:
  build:
    runs-on: windows-latest

    steps:
      - name: Download Ngrok
        run: Invoke-WebRequest https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-windows-amd64.zip -OutFile ngrok.zip

      - name: Extract Ngrok
        run: Expand-Archive ngrok.zip

      - name: Authenticate Ngrok
        run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
        env:
          NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}

      - name: Enable Remote Desktop
        run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0

      - name: Configure Firewall for RDP
        run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"

      - name: Configure RDP Authentication
        run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name "UserAuthentication" -Value 1

      - name: Set RDP User Password
        run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "P@ssw0rd!" -Force)

      - name: Create Ngrok Tunnel for RDP
        run: .\ngrok\ngrok.exe tcp 3389
