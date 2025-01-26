# WINDOWS SETUP

<details>
<summary>Table of Contents</summary>

- [WINDOWS SETUP](#windows-setup)
  - [Base Configuration](#base-configuration)
    - [Enable powershell script execution](#enable-powershell-script-execution)
    - [File Explorer](#file-explorer)
    - [Settings](#settings)
    - [Add Webpage To Host](#add-webpage-to-host)
    - [Fonts](#fonts)
  - [Installing Software(s)](#installing-softwares)
    - [7-Zip](#7-zip)
    - [Powershell 7](#powershell-7)
    - [Git](#git)
    - [Git - Delta](#git---delta)
    - [NodeJS](#nodejs)
    - [VS Code](#vs-code)
    - [Neovim](#neovim)
    - [PowerToys](#powertoys)
    - [Docker Desktop](#docker-desktop)
  - [Gitlab Via Docker](#gitlab-via-docker)

</details>

## Base Configuration

<details>
<summary>Configuration</summary>

### Enable powershell script execution

```powershell
  # Only For Current Session
Set-ExecutionPolicy -ExecutionPolicy AllSigned -Scope Process

# Always Enable For Current User
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force
```

- References:
  - [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy)
  - [makeuseof](https://www.makeuseof.com/enable-script-execution-policy-windows-powershell/)

### File Explorer

- Option
  - General
    - Open File Explorer to `This PC`.
    - Disable
      - Show recently used files
      - Show frequently used folders
      - Show recommended sections
      - Include account based insights
    - View
      - Enable `Decrease space between items`
      - Enable `Display the full path in the title bar`
      - Select `Show hidden files, folders, and drives`
      - Disable `Hide extensions for known file types`

### Settings

- Open settings
  - Personalization
    - Colors
      - Dark
    - Start
      - Layout: more pins
      - Disable all options
    - Taskbar
      - Disable Copilot, Task View, Widgets
      - Modify 'other system tray icons'
      - Modify 'taskbar behaviors'
  - Apps > Advanced app settings > App execution aliases
    - Disable `python` and `python3`

### Add Webpage To Host

- Open powershell as admin

  ```powershell
  Add-Content -Path $env:windir\System32\drivers\etc\hosts -Value "`n127.0.0.1`tlocalhost" -Force
  ```

### Fonts

- Install fonts for overall use.

```powershell
function get-github-repo-latest-release {
    param(
        [Parameter(Mandatory = $true)][string]$repo
    )
    $url = "https://github.com/" + $repo + "/releases/latest"
    $request = [System.Net.WebRequest]::Create($url)
    $response = $request.GetResponse()
    $realTagUrl = $response.ResponseUri.OriginalString
    $version = $realTagUrl.split('/')[-1].Trim('v')
    return $version
}

$sourcedir   = "$env:userprofile/setup/fonts"

# Font - Fira Code, JetBrainsMono, Caskaydia Cove
$repo = "ryanoasis/nerd-fonts"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/JetBrainsMono.zip" `
 -outfile (new-item -path "$sourcedir\JetBrainsMono.zip" -force)

get-childitem -path $sourcedir | foreach {
    expand-archive -path $_.fullname -destinationpath "$sourcedir" -force
}

# Only copy below lines if filter is correct, install manually if unsure.
$destination = (new-object -comobject shell.application).namespace(0x14)
# filter filename that contains `font-`, and does not include NL
get-childitem -path $sourcedir -filter "*font-*" | where-object {$_.name -match "^((?!NL).)*$"} | foreach {
    # install font
    $destination.copyhere($_.fullname,0x10)
}
# filter filename that contains `fontmono-`, and does not include NL
get-childitem -path $sourcedir -filter "*fontmono-*" | where-object {$_.name -match "^((?!NL).)*$"} | foreach {
    # install font
    $destination.copyhere($_.fullname,0x10)
}
remove-item -path "$sourcedir/*" -recurse -force

```

- Install fonts for console. Open powershell as admin.

```powershell
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Console\TrueTypeFont' -Name '0000' -Value 'JetBrainsMono Nerd Font Mono'

```

- Open each console (cmd, powershell, etc) and update the font.

- References:
  - https://superuser.com/questions/1347724/how-can-i-add-additional-fonts-to-the-windows-console
  - https://gist.github.com/anthonyeden/0088b07de8951403a643a8485af2709b
  - https://richardspowershellblog.wordpress.com/2008/03/20/special-folders/


</details>


## Installing Software(s)

<details>
<summary>Installing Software(s)</summary>

- Ensure powershell already has this fucntion in powershell profile. If not, copy the function to your current terminal.

  ```powershell
  function get-github-repo-latest-release {
      param(
          [Parameter(Mandatory = $true)][string]$repo
      )
      $url = "https://github.com/" + $repo + "/releases/latest"
      $request = [System.Net.WebRequest]::Create($url)
      $response = $request.GetResponse()
      $realTagUrl = $response.ResponseUri.OriginalString
      $version = $realTagUrl.split('/')[-1].Trim('v')
      return $version
  }
  ```

- List of apps:
  - [x] 7-Zip
  - [x] Powershell 7
  - [x] Git
  - [x] Git - Delta
  - [x] NodeJS
  - [x] VSCode
  - [x] Neovim
  - [ ] Docker Desktop
  - [ ] Fastfetch
  - [ ] Wezterm
  - [ ] AutoHotKey
  - [ ] VS Build Tools
  - [ ] VLC
  - [ ] Notepad++
  - [ ] bitwarden
  - [ ] cygwin
  - [ ] clink (cmd)
  - [ ] powertoys
  - [ ] ripgrep
   - [ ] jq
   - [ ] fzf
   - [ ] bat
   - [ ] pyenv (python)
   - [ ] uv (python)

### 7-Zip

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\7zip.exe"
$url = 'https://7-zip.org/' + (invoke-webrequest -usebasicparsing -uri 'https://7-zip.org/' `
  | select-object -expandproperty links `
  | where-object {($_.outerhtml -match 'download') -and ($_.href -like "a/*") -and ($_.href -like "*-x64.exe")} `
  | select-object -first 1 | select-object -expandproperty href)
invoke-webrequest $url -outfile (new-item -path "$app" -force)
Start-Process -FilePath $app -Args "/S" -Verb RunAs -Wait
Remove-Item $app
```

### Powershell 7

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\powershell.msi"
$repo = "powershell/powershell"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/powershell-$version-win-x64.msi" -outfile (new-item -path "$app" -force)
# iex "& { $(irm https://aka.ms/install-powershell.ps1) } -UseMSI -Quiet"
start-process -filepath "$app" -Args "/quiet /passive ADD_EXPLORER_CONTEXT_MENU_OPENPOWERSHELL=1 ADD_PATH=1" -Wait
Remove-Item $app
```

### Git

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\git.exe"
$repo = "git-for-windows/git"
$version = get-github-repo-latest-release "$repo"
$version = $version -split "\.\D+.+"
$version = $version.split(" ")[0]
$url = "https://github.com/$repo/releases/download/v$version.windows.1/Git-$version-64-bit.exe"
invoke-webrequest "https://github.com/$repo/releases/download/v$version.windows.1/Git-$version-64-bit.exe" -outfile (new-item -path "$app" -force)
start-process -filepath "$app" -args "/VERYSILENT /NORESTART" -wait
[System.Environment]::SetEnvironmentVariable('path', "C:\Program Files\Git\bin;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### Git - Delta

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\delta.zip"
$repo = "dandavison/delta"
$version = get-github-repo-latest-release "$repo"
$url = "https://github.com/$repo/releases/download/$version/delta-$version-x86_64-pc-windows-msvc.zip"
invoke-webrequest $url -outfile (new-item -path "$app" -force)
expand-archive -path "$app" -destinationpath "$env:localappdata"
$temp = get-childitem -path  $env:localappdata -directory -filter "*delta*" | select-object -expandproperty name
rename-item "$env:localappdata\$temp" "$env:localappdata\delta"
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\delta;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### NodeJS

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\node_js.msi"
$url = (invoke-webrequest -usebasicparsing -uri "https://nodejs.org/en" `
  | select-object -expandproperty links `
  | where-object {($_.outerhtml -match "LTS")} `
  | select-object -first 1 `
  | select-object -expandproperty href).replace(".tar.gz", "-x64.msi")
invoke-webrequest "$url" -outfile (new-item -path "$app" -force)
start-process -filepath "msiexec.exe" -args "/i $app /qn /l* $root_download\software\node-log.txt" -wait
Remove-Item $app
```

### VS Code

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\vscode.exe"
invoke-webrequest "https://code.visualstudio.com/sha/download?build=stable&os=win32-x64-user" -outfile (new-item -path "$app" -force)
start-process -filepath "$app" -args "/verysilent /norestart /mergetasks=addcontextmenufiles,addcontextmenufolders,!runcode,!desktopicon" -wait
Remove-Item $app
```

### Neovim

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\neovim.zip"
$repo = "neovim/neovim"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/nvim-win64.zip" -outfile (new-item -path "$app" -force)
expand-archive -path "$app" -destinationpath "$env:localappdata"
$temp = get-childitem -path  $env:localappdata -directory -filter "*nvim-win64*" | select-object -expandproperty name
rename-item "$env:localappdata\$temp" "$env:localappdata\neovim"
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\neovim\bin;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### PowerToys

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\powertoys.exe"
$repo = "microsoft/PowerToys"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/PowerToysUserSetup-$version-x64.exe" -outfile (new-item -path "$app" -force)
start-process -filepath "$app" -args "/quiet /passive" -wait
Remove-Item $app
```

### Docker Desktop

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\docker.exe"
$url = (invoke-webrequest -usebasicparsing -uri "https://docs.docker.com/desktop/setup/install/windows-install/" | select-object -expandproperty links | where-object {($_.outerhtml -match "amd64")} | select-object -expandproperty href)
invoke-webrequest "$url" -outfile (new-item -path "$app" -force)
start-process -filepath $app -wait install
Remove-Item $app
```

</details>

## Gitlab Via Docker

1. Create GITLAB_HOME environment variable

  ```powershell
  [Environment]::SetEnvironmentVariable('GITLAB_HOME', 'D:\.gitlab', 'user')
  ```

2. Create docker container

  ```powershell
  docker run --detach --hostname 127.0.0.1 `
  --publish <external_url_port>:<external_url_port> --publish <gitlab_shell_ssh_port>:22 --name gitlab `
  --restart always --volume $env:GITLAB_HOME\config:/etc/gitlab `
  --volume $env:GITLAB_HOME\logs:/var/log/gitlab `
  --volume $env:GITLAB_HOME\data:/var/opt/gitlab gitlab/gitlab-ce:latest
  ```

3. Install vim in docker container. Edit /etc/gitlab/gitlab.rb, modify `external_url "http://your-ip-address:external_url_port"`, `gitlab_rails['gitlab_shell_ssh_port'] = gitlab_shell_ssh_port`

  ```bash
  docker exec -it gitlab bash
  apt install -y vim
  vim /etc/gitlab/gitlab.rb
  gitlab-ctl reconfigure
  ```

4. Set gitlab root password

  ```bash
  gitlab-rake "gitlab:password:reset[root]"
  ```

- References:
  - [Install gitlab on Windows with Docker](https://stackoverflow.com/a/66357935)
