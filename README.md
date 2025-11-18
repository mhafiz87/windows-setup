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
    - [Scoop](#scoop)
    - [7-Zip](#7-zip)
    - [Powershell 7](#powershell-7)
    - [Git](#git)
    - [Git - Delta](#git---delta)
    - [NodeJS](#nodejs)
    - [VS Code](#vs-code)
    - [Neovim](#neovim)
    - [PowerToys](#powertoys)
    - [Docker Desktop](#docker-desktop)
    - [VS Build Tools](#vs-build-tools)
    - [cygwin](#cygwin)
    - [clink (cmd)](#clink-cmd)
    - [riprep](#riprep)
    - [jq](#jq)
    - [fzf](#fzf)
    - [bat](#bat)
    - [less](#less)
    - [yt-dlp](#yt-dlp)
    - [Pyenv (Python)](#pyenv-python)
    - [UV (Python)](#uv-python)
  - [Installing Language Server Protocol (LSP)](#installing-language-server-protocol-lsp)
    - [LuaLS](#luals)
    - [Ruff](#ruff)
    - [Marksman](#marksman)
    - [Stylua](#stylua)
    - [Taplo](#taplo)
    - [Powershell Service Editor](#powershell-service-editor)
    - [Markdownlint (requires npm)](#markdownlint-requires-npm)
    - [Markdownlint-cli2 (requires npm)](#markdownlint-cli2-requires-npm)
    - [Prettier (requires npm)](#prettier-requires-npm)
    - [YAML](#yaml)
    - [bash-language-server](#bash-language-server)
  - [WSL](#wsl)
    - [In Windows](#in-windows)
    - [In Linux](#in-linux)
  - [Git](#git)
  - [Gitlab](#gitlab)
    - [Gitlab Via Docker](#gitlab-via-docker)
    - [Gitlab Runner Via Docker](#gitlab-runner-via-docker)

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
  - [] Scoop
  - [x] 7-Zip
  - [x] Powershell 7
  - [x] Git
  - [x] Git - Delta
  - [x] NodeJS
  - [x] VSCode
  - [x] Neovim
  - [x] PowerToys
  - [x] Docker Desktop
  - [ ] Fastfetch
  - [ ] Wezterm
  - [ ] AutoHotKey
  - [ ] VS Build Tools
  - [ ] VLC
  - [ ] Notepad++
  - [ ] bitwarden
  - [ ] cygwin
  - [ ] msys2
  - [ ] clink (cmd)
  - [ ] ripgrep
  - [ ] jq
  - [ ] fzf
  - [x] bat
  - [x] less
  - [x] pyenv (python)
  - [x] uv (python)

### Scoop

```powershell
# https://github.com/ScoopInstaller/Scoop


```

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

### NodeJS (as admin)

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\node_js.msi"
$url = (invoke-webrequest -usebasicparsing -uri "https://nodejs.org/en/download" `
  | select-object -expandproperty links `
  | where-object {($_.href -match ".tar.gz")} `
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
if ((Test-Path $env:localappdata\neovim) -eq $true){
  echo "neovim already exist in localappdata. Deleting..."
  Remove-Item -Path $env:localappdata\neovim -Recurse -Force
}
expand-archive -path "$app" -destinationpath "$env:localappdata"
$temp = get-childitem -path  $env:localappdata -directory -filter "*nvim-win64*" | select-object -expandproperty name
rename-item "$env:localappdata\$temp" "$env:localappdata\neovim"
$exist_env = [System.Environment]::GetEnvironmentVariable('path', "User") -Like "*neovim\bin*"
if ($exist_env -eq $true) {
  echo "neovim already exist in path"
}else{
  [System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\neovim\bin;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
}
Remove-Item $app
```

### PowerToys

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\powertoys.exe"
$repo = "microsoft/PowerToys"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/PowerToysUserSetup-$version-x64.exe" -outfile (new-item -path "$app" -force)
#start-process -filepath "$app" -args "/quiet /passive" -wait
start-process -filepath "$app" -wait
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

### VS Build Tools

- Component To Install:
  - MSVC v143
  - C++ CMake tools for Windows
  - Windows 10 SDK

- Download build tools from [here](https://visualstudio.microsoft.com/visual-cpp-build-tools/) into Downloads.
- Run below script:

```powershell
cd $env:userprofile\downloads
new-item -itemtype directory -path vsinstaller
.\vs_buildtools.exe --layout e:\vsinstaller --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.VC.CMake.Project --add Microsoft.VisualStudio.Component.Windows11SDK.22621
cd vsinstaller
.\vs_buildtools.exe --noWeb --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.VC.CMake.Project --add Microsoft.VisualStudio.Component.Windows11SDK.22621
```

- Add CMAKE path to Windows Environment User Variable by searching CMAKE in Microsoft Visual Studio install path.
- Delete vsinstaller folder in downloads directory

```powershell
Remove-Item -Path $env:userprofile\downloads\vs_installer -Recurse -Force
```

- References:
  - [List of VS components](https://learn.microsoft.com/en-us/visualstudio/install/workload-and-component-ids)
  - [Create an offline installation](https://learn.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio)

### clink (cmd)

### riprep

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\rg.zip"
$repo = "BurntSushi/ripgrep"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/ripgrep-$version-x86_64-pc-windows-msvc.zip" -outfile (new-item -path "$app" -force)
expand-archive -path "$app" -destinationpath "$env:localappdata"
$temp = get-childitem -path  $env:localappdata -directory -filter "*ripgrep*" | select-object -expandproperty name
rename-item "$env:localappdata\$temp" "$env:localappdata\ripgrep"
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\ripgrep;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### jq

```powershell
$repo = "jqlang/jq"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/jq-win64.exe" -outfile (new-item -path "$env:localappdata\jq\jq.exe" -force)
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\jq;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
```

### fzf

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\fzf.zip"
$repo = "junegunn/fzf"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/fzf-$version-windows_amd64.zip" -outfile (new-item -path "$app" -force)
expand-archive -path "$app" -destinationpath "$env:localappdata\fzf"
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\fzf;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### bat

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\bat.zip"
$repo = "sharkdp/bat"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/bat-v$version-x86_64-pc-windows-msvc.zip" -outfile (new-item -path "$app" -force)
expand-archive -path "$app" -destinationpath "$env:localappdata"
$temp = get-childitem -path  $env:localappdata -directory -filter "*bat*" | select-object -expandproperty name
rename-item "$env:localappdata\$temp" "$env:localappdata\bat"
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\bat;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### less

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\less.zip"
$repo = "jftuga/less-Windows"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/less-x64.zip" -outfile (new-item -path "$app" -force)
expand-archive -path "$app" -destinationpath "$env:localappdata\less"
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\less;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
Remove-Item $app
```

### yt-dlp

```powershell
$root_download = "$env:localappdata\programs"
$app = $root_download + "\yt-dlp.exe"
$repo = "yt-dlp/yt-dlp"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/yt-dlp.exe" -outfile (new-item -path "$app" -force)
[System.Environment]::SetEnvironmentVariable('path', $env:localappdata + "\Programs;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
```


### Pyenv (Python)

```powershell
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

 ### UV (Python)

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Go

```powershell
$root_download = "$env:userprofile\setup"
$app = $root_download + "\software\go-windows-amd64.msi"
$url = "https://go.dev" + (invoke-webrequest -usebasicparsing -uri "https://go.dev/dl/" `
    | select-object -expandproperty links `
    | where-object {($_.outerhtml -match "windows-amd64.msi")} `
    | select-object -first 1 `
    | select-object -expandproperty href)
invoke-webrequest "$url" -outfile (new-item -path "$app" -force)
$install_path = "$env:localappdata\go\"
start-process -filepath "msiexec" -args "/i $app INSTALLDIR=$install_path /qn /L*V $root_download\software\go_install.log" -wait
remote-item $app
```

</details>


## Installing Language Server Protocol (LSP)

<details>
<summary>Installing LSP(s)</summary>

### LuaLS

```powershell
if(test-path -path $env:userprofile\lsp\luals){remove-item -path "$env:userprofile\lsp\luals" -recurse -force}
$root_download = "$env:userprofile\lsp"
$lsp = $root_download + "\luals.zip"
$repo = "LuaLS/lua-language-server"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/lua-language-server-$version-win32-x64.zip" -outfile (new-item -path "$lsp" -force)
expand-archive -path "$lsp" -destinationpath "$root_download\luals"
Remove-Item $lsp
[System.Environment]::SetEnvironmentVariable('path', "$root_download\luals\bin;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
```

### Ruff

- Via UV

  ```powershell
  uv tool install ruff@latest
  ```

- Via PowerShell

  ```powershell
  if(test-path -path $env:userprofile\lsp\ruff){remove-item -path "$env:userprofile\lsp\ruff" -recurse -force}
  $root_download = "$env:userprofile\lsp"
  $lsp = $root_download + "\ruff.zip"
  $repo = "astral-sh/ruff"
  $version = get-github-repo-latest-release "$repo"
  invoke-webrequest "https://github.com/$repo/releases/download/$version/ruff-x86_64-pc-windows-msvc.zip" -outfile (new-item -path "$lsp" -force)
  expand-archive -path "$lsp" -destinationpath "$root_download\ruff"
  Remove-Item $lsp
  [System.Environment]::SetEnvironmentVariable('path', "$root_download\ruff;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
  ```

### Marksman

```powershell
if(test-path -path $env:userprofile\lsp\marksman){remove-item -path "$env:userprofile\lsp\marksman" -recurse -force}
$root_download = "$env:userprofile\lsp"
$lsp = $root_download + "\marksman\marksman.exe"
$repo = "artempyanykh/marksman"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/marksman.exe" -outfile (new-item -path "$lsp" -force)
[System.Environment]::SetEnvironmentVariable('path', "$root_download\marksman;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
```

### Stylua

```powershell
if(test-path -path $env:userprofile\lsp\stylua){remove-item -path "$env:userprofile\lsp\stylua" -recurse -force}
$root_download = "$env:userprofile\lsp"
$lsp = $root_download + "\stylua.zip"
$repo = "JohnnyMorganz/StyLua"
# $version = get-github-repo-latest-release "$repo"
$apiUrl = "https://api.github.com/repos/$repo/releases/latest"
$response = Invoke-RestMethod -Uri $apiUrl
$version = $response.tag_name
invoke-webrequest "https://github.com/$repo/releases/download/$version/stylua-windows-x86_64.zip" -outfile (new-item -path "$lsp" -force)
expand-archive -path "$lsp" -destinationpath "$root_download\stylua"
Remove-Item $lsp
if(Get-Command -Name stylua -ErrorAction SilentlyContinue){
  Write-Host "stylua exist in path"
}else{
  Write-Host "adding stylua to path"
  [System.Environment]::SetEnvironmentVariable('path', "$root_download\stylua;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
}
```

### Taplo

```powershell
if(test-path -path $env:userprofile\lsp\taplo){remove-item -path "$env:userprofile\lsp\taplo" -recurse -force}
$root_download = "$env:userprofile\lsp"
$lsp = $root_download + "\taplo.zip"
$repo = "tamasfe/taplo"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/$version/taplo-windows-x86_64.zip" -outfile (new-item -path "$lsp" -force)
expand-archive -path "$lsp" -destinationpath "$root_download\taplo"
Remove-Item $lsp
[System.Environment]::SetEnvironmentVariable('path', "$root_download\taplo;" + [System.Environment]::GetEnvironmentVariable('path', "User"),"User")
```

### Powershell Service Editor

```powershell
if(test-path -path $env:userprofile\lsp\PowerShellEditorServices){remove-item -path "$env:userprofile\lsp\PowerShellEditorServices" -recurse -force}
$root_download = "$env:userprofile\lsp"
$lsp = $root_download + "\PowerShellEditorServices.zip"
$repo = "PowerShell/PowerShellEditorServices"
$version = get-github-repo-latest-release "$repo"
invoke-webrequest "https://github.com/$repo/releases/download/v$version/PowerShellEditorServices.zip" -outfile (new-item -path "$lsp" -force)
expand-archive -path "$lsp" -destinationpath "$root_download\PowerShellEditorServices"
Remove-Item $lsp
```

### Markdownlint (requires npm)

```powershell
npm install --iwr --global markdownlint
```

### Markdownlint-cli2 (requires npm)

```powershell
npm install --iwr --global markdownlint-cli2
```

### Prettier (requires npm)

```powershell
npm install --iwr --global prettier
```

### YAML

```powershell
npm install --iwr --global yaml-language-server
```

### bash-language-server

```powershell
npm install --iwr --global bash-language-server
```

### shfmt (requires go)

```powershell
go install mvdan.cc/sh/v3/cmd/shfmt@latest
```

</details>

## WSL

<details>
<summary>WSL Setup</summary>

### In Windows

```powershell
$text = "# Enable experimental features
[experimental]
sparseVhd=true

[wsl2]
networkingMode=mirrored
dnsTunneling=true`r`n"
if (!(Test-Path $env:userprofile\.wslconfig))
{
   New-Item -path $env:userprofile -name .wslconfig -type "file" -value $text
   Write-Host "Created new file and text content added"
} else {
  Add-Content -path $env:userprofile\.wslconfig -value $text
  Write-Host "File already exists and new text content added"
}
wsl --install -d Debian
```

### In Linux

- Disable windows path

  ```bash
  sudo echo -e "\n[interop]\nappendWindowsPath=false\n" >> /etc/wsl.conf
  ```

- Restart WSL in windows

  ```powershell
  wsl --shutdown -d Debian  # if using Debian
  ```

- Copy `$env:userprofile/.config/ohmyposh/zen.toml` to `~/.config/ohmyposh/zen.toml`

- Setup

  ```bash
  sudo apt update && sudo apt upgrade -y && sudo apt install -y build-essential \
  libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl git \
  libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev \
  unzip wget bat fzf jq
  # Install OhMyPosh
  curl -s https://ohmyposh.dev/install.sh | bash -s
  echo 'eval "$(oh-my-posh init bash --config ~/.config/ohmyposh/zen.toml)"' >> ~/.bashrc
  # Install PyEnv
  curl -fsSL https://pyenv.run | bash
  echo 'export PYENV_ROOT="$HOME/.pyenv"'  >> ~/.bashrc
  echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
  echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc
  # Install UV
  curl -LsSf https://astral.sh/uv/install.sh | sh
  source ~/.bashrc
  ```

- References
  - [No internet connection Ubuntu-WSL while VPN](https://superuser.com/a/1818812)
  - [How to remove the Win10's PATH from WSL](https://stackoverflow.com/a/51345880)
  - [UV](https://docs.astral.sh/uv/)
  - [PyEnv](https://github.com/pyenv/pyenv)
  - [PyEnv Build Environment](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)

</details>

## Git

<details>
<summary>Git Config</summary>

```powershell
git config --global user.name <username>
git config --global user.email <email>
git config --global color.ui "auto"
git config --global alias.hist "log --oneline --graph --decorate --all --abbrev-commit --pretty=format:'%C(red)%h%C(reset) -%C(yellow)%d%C(reset) %s %C(green)(%cr) %C(bold blue)%an%C(reset)'"
git config --global alias.lg "log --no-merges --first-parent --decorate --abbrev-commit --pretty=tformat:'%C(bold red)%h%C(reset)%C(bold yellow)%d%C(reset) | %C(bold cyan)%an%C(reset) | %C(bold green)%cr%C(reset)%n%s%n%b'"
git config --global commit.template "~/.config/git/.gitmessage"
git config --global status.short true
git config --global status.branch true
git config --global status.showStash true
git config --global core.hooksPath "~/.config/git/hooks"
# git config --global core.editor "notepad++ -multiInst -notabbar -nosession -noPlugin"
git config --global core.editor "nvim -f --clean -c 'set mouse=a nu rnu splitbelow splitright smartindent expandtab shiftwidth=4 softtabstop=4 timeoutlen=500 nowrap clipboard=unnamedplus completeopt=fuzzy,menu,menuone,noinsert,noselect,popup ignorecase scrolloff=8 sidescrolloff=8 guicursor=n-v-c:block,i-ci-ve:ver25,r-cr:hor20,o:hor50,a:blinkwait700-blinkoff400-blinkon250-Cursor/lCursor,sm:block-blinkwait175-blinkoff150-blinkon175 wildmode=longest:full,full wildoptions=fuzzy,pum,tagfile winborder=rounded' -c 'imap jk <Esc>'"
git config --global core.longpaths true
git config --global core.pager "delta"
git config --global url."https://github.com/mhafiz87/".insteadOf "mhafiz87:"
# git config --global diff.tool "vscode"
# git config --global difftool.vscode.cmd "code --wait --diff $local $remote"
# git config --global merge.tool "vscode"
# git config --global mergetool.vscode.cmd "code --wait $merged"
# git config --global merge.tool "diffview"
# git config --global mergetool.prompt "false"
# git config --global mergetool.keepBackup "false"
# git config --global mergetool.diffview.cmd 'nvim -n -c "DiffviewOpen" "$MERGE"'
# git config --global merge.tool "nvimdiff"
# git config --global mergetool.prompt "false"
# git config --global mergetool.keepBackup "false"
# git config --global mergetool.nvimdiff.cmd "LOCAL,BASE,REMOTE / MERGED"
git config --global interactive.diffFilter "delta --color-only"
git config --global delta.true-color always
git config --global delta.navigate true
git config --global delta.line-numbers true
git config --global delta.hyperlinks true
git config --global delta.side-by-side true
```

</details>

## FreeCAD

<details>
  <summary>Freecad</summary>

### Addon Manager

- OpenTheme
- Maker Workbench

### Macro

- [Honeycomb](https://github.com/mhafiz87/honeycomb/blob/main/Honeycomb.py)

### Preferences

- General
  - Application
    - Theme: OpenDark
    - Size of toolbar icons: Small(16px)
  - Preference Pack Name: OpenPreferences
  
</details>

## Gitlab

<details>
<summary>Gitlab</summary>

### Gitlab Via Docker

- Create GITLAB_HOME environment variable

  ```powershell
  [Environment]::SetEnvironmentVariable('GITLAB_HOME', 'D:\.gitlab', 'user')
  ```

- Create docker container

  ```powershell
  $external_url_port = Read-Host "Enter external url port: "
  $ssh_port = Read-Host "Enter SSH port: "
  docker run --detach --hostname 127.0.0.1 `
  --publish ${external_url_port}:${external_url_port} --publish ${ssh_port}:22 --name gitlab `
  --restart always --volume $env:GITLAB_HOME\config:/etc/gitlab `
  --volume $env:GITLAB_HOME\logs:/var/log/gitlab `
  --volume $env:GITLAB_HOME\data:/var/opt/gitlab gitlab/gitlab-ce:latest
  ```

- Check gitlab container ip address.

  - Check IP Address
    - Find the container ID

      ```powershell
      docker ps -a
      ```

    - Find the IP address; __remove `<` `>`__.

      ``` powershell
      docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container id>
      ```

- Install vim in docker container. Edit /etc/gitlab/gitlab.rb, modify `external_url "http://gitlab-container-ip-address:external_url_port"`, `gitlab_rails['gitlab_shell_ssh_port'] = gitlab_shell_ssh_port`

  ```bash
  docker exec -it gitlab bash
  apt install -y vim
  vim /etc/gitlab/gitlab.rb
  gitlab-ctl reconfigure
  ```

- Restart docker container:

  ```powershell
  docker restart docker
  ```

- Store your smtp user name and password via docker terminal:

  ```bash
  gitlab-rake gitlab:smtp:secret:edit EDITOR=vim
  ```

- Modify /etc/gitlab/gitlab.rb for gmail smtp:

  ```bash
  gitlab_rails['smtp_enable'] = true
  gitlab_rails['smtp_address'] = "smtp.gmail.com"
  gitlab_rails['smtp_port'] = 587
  gitlab_rails['smtp_domain'] = "smtp.gmail.com"
  gitlab_rails['smtp_authentication'] = "login"
  gitlab_rails['smtp_enable_starttls_auto'] = true
  gitlab_rails['smtp_tls'] = false

  gitlab_rails['gitlab_email_enabled'] = true

  gitlab_rails['gitlab_email_from'] = '<gmail email>'
  gitlab_rails['gitlab_email_display_name'] = '<your name>'
  gitlab_rails['gitlab_email_reply_to'] = '<gmail email>'
```

7. Set gitlab root password

  ```bash
  gitlab-rake "gitlab:password:reset[root]"
  ```

- To test sending emails, run the following command:

  ```bash
  gitlab-rails console
  Notify.test_email("<email-address>", "Test email from Gitlab", "This is a test email from Gitlab!").deliver_now
  ```

- References:
  - [Install gitlab on Windows with Docker](https://stackoverflow.com/a/66357935)
  - [YouTube: Gmail SMTP Server Settings: Host Username and Password for Projects](https://www.youtube.com/watch?v=I9x0w8cjR_o)
  - [Gmail SMTP Settings: Easy Step-by-Step Setup Guide](https://www.gmass.co/blog/gmail-smtp/)
  - [Gitlab Omnibus SMTP SMTP](https://docs.gitlab.com/omnibus/settings/smtp.html)

### Gitlab Runner Via Docker

- Create docker volume for gitlab-runner-config

  ```powershell
  docker volume create gitlab-runner-config
  ```

- Install gitlab-runner via docker

  ```powershell
  docker run -d --name gitlab-runner --restart always `
  -v /var/run/docker.sock:/var/run/docker.sock `
  -v gitlab-runner-config:/etc/gitlab-runner `
  gitlab/gitlab-runner:alpine
  ```

- Check IP Address
  - Find the container ID

    ```powershell
    docker ps -a
    ```

  - Find the IP address; __remove `<` `>`__.

    ``` powershell
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container id>
    ```

- Create runner in gitlab, copy the token, then register `runner`. Select `shell` as the executor.

  ```powershell
  docker run --rm -it -v gitlab-runner-config:/etc/gitlab-runner gitlab/gitlab-runner:alpine register
  ```

- Modify /etc/gitlab-runner/config.toml in `gitlab-runner` container by adding `clone_url` key under `url` with same value as `url`.

  ```powershell
  docker exec -it gitlab-runner bash
  ```

  ```bash
  vi /etc/gitlab-runner/config.toml
  ```

- Exit gitlab-runner bash

- References:
  - [YouTube - How to register & run GitLab Runner inside a Docker container](https://www.youtube.com/watch?v=JLdPiq0owUM)
  - [Run GitLab Runner in a container](https://docs.gitlab.com/runner/install/docker.html)
  - [https://docs.gitlab.com/runner/register/?tab=Docker](https://docs.gitlab.com/runner/register/?tab=Docker)
  - [Jobs unable to access my gitlab instances](https://forum.gitlab.com/t/jobs-are-unable-to-access-my-gitlab-instance/62060/2)

  </details>
