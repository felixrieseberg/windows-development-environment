## Setup Windows 11 for Modern/Hipster Development

A fresh Windows isn't entirely ready for modern development, but all the tools you need are available. A good terminal, popular bash tools, Git, a decent package manager - when properly setup, modern development on Windows can be a lot of fun. In particular, this document outlines how to configure your Windows in such a way that it can easily handle most development tasks usually run on a Mac OS X or a Linux distro.

## A Word about Ubuntu Linux on Windows

:point_up: While [Bash on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about) isn't perfect yet, it's an amazing tool that can make development a lot easier - especially when you're dealing with Bash scripts, Ruby, or Ubuntu binaries. It's an amazing tool, but keep in mind that Git, Node, or Go are already pretty performant on Windows itself. However, they run just fine in Bash, so if you feel like moving most of your development over, go for it. Here's the how-to:

-   Ensure that you're running Windows 10 Anniversary Update (build 14311 and up)
-   Enable Developer Mode (Settings - Update & security > For developers)
-   Search for “Windows Features” and choose “Turn Windows features on or off” and enable Windows Subsystem for Linux.
-   To get Bash installed, open Command Prompt and type “bash”

## Automate it!

Below, you can see the all the things I need to actually go and work on stuff. An opinionated version of the list below can be installed automatically thanks to the magic of [Boxstarter](http://boxstarter.org/). [Check out the script to see what it runs](https://github.com/felixrieseberg/windows-development-environment/blob/master/boxstarter). It's a trimmed down version of all the tools below, leaving out duplicate tools. As an example, it installs VS Code (and not Atom or Sublime) and Git (but not Subversion or Mercurial). Simply start PowerShell as Administrator and run:

```powershell
START http://boxstarter.org/package/nr/url?https://raw.githubusercontent.com/felixrieseberg/windows-development-environment/master/boxstarter
```

## The Goods

-   [Package Management](#package-management-chocolatey)
-   [Terminal](#terminal-cmder-with-powershell-support)
-   [PowerShell Profile](#powershell-profile)
-   [Node.js](#node-and-npm)
-   [Git](#version-control-git)
-   [Atom, Sublime, VS Code](#code-editors-atom-sublime-vs-code)
-   [Ruby](#ruby)
-   [Go](#go)
-   [Python](#python)
-   [DevOps: VirtualBox, Vagrant, Docker](#devops-virtualbox-vagrant-docker)
    -   [SSH](#ssh)
    -   [Azure Cli](#azure-cli)
    -   [AWS Cli](#aws-cli)
-   [Basic Tools](#basic-tools)

#### Package Management: Chocolatey

Chocolatey is a powerful package manager for Windows, working sort of like apt-get or homebrew. Let's get that first. Fire up CMD.exe as Administrator and run:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Once done, you can install packages by running `cinst` (short for `choco install`). Most packages below will be installed with Chocolatey.

#### Terminal: Windows Terminal

These days, I just use [Windows Terminal](https://github.com/microsoft/terminal) with [oh-my-posh](https://ohmyposh.dev/).

I used to use [CMDer](https://github.com/bliker/cmder/) and [Hyper](https://hyper.is/), both powerful tools to do more command-line things with. They were great. I've personally moved on but you might enjoy the old-timers:

```powershell
cinst microsoft-windows-terminal
cinst oh-my-posh
```

Even if you don't want to use either, you should enable your PowerShell to execute scripts. You're a developer - the terminal is your friend.

```powershell
Set-ExecutionPolicy Unrestricted -Scope CurrentUser
```

If you want to go even further, check out the attached PowerShell Profile in this repository. It's my personal one and might not be perfect for you, but it makes my personal life a lot easier. You can edit your PowerShell profile with your favorite editor by calling `$PROFILE`, so if you're using Visual Studio Code, call `code $PROFILE` (or `vim $PROFILE` - you get the idea).

#### PowerShell Profile

Now that you have a good terminal, you might wonder how you can beef it up. This isn't part of the automated setup, but I'm quite happy with my profile. I've changed my PowerShell in the following ways:

-   Add helper functions (`uptime`, `reload-profile`, `find-file`, `unzip`, and `print-path`)
-   Add equivalents for my favorite Unix commands (`df`, `sed`, `sed-recursive`, `grep`, `grepv`, `which`, `export`, `pkill`, `pgrep`, `touch`, `sudo`, `pstree`)
-   Add Git aliases (`gc` for `git checkout`, `gp` for `git pull`)
-   Set the default output to `utf8`
-   Increase the linme history to 10000

To edit your PowerShell profile, run `notepad.exe $PROFILE` (or use an editor of your choice). Then, replce it with [this profile](https://github.com/felixrieseberg/windows-development-environment/blob/master/Microsoft.PowerShell-profile.ps1).

#### Node and npm

A bunch of tools are powered by Node and installed via npm. This applies to you even if you don't care about Node development. If you want to install tools for React, Azure, TypeScript, or Cordova, you'll need this.

```powershell
cinst nodejs.install
```

#### Version Control: Git

Obviously. If you want Git to be able to save credentials (so you don't have to enter SSH keys / passwords every single time you do anything), use the `git.install` package, which also installs the Git Credential Manager for Windows. I add parameters I care about.

```powershell
cinst git.install /NoShellIntegration /WindowsTerminalProfile /Symlinks /DefaultBranchName:main /Editor:VisualStudioCodeInsiders
```

<details>
<summary>📝 Bonus: Mercurial, Subversion</summary>
   
If you're using Mercurial or Subversion (you poor, poor thing), install with:
```
cinst subversion
cinst mercurial
```

</details>

#### Code Editors: Atom, Sublime, VS Code

I won't join the war debating which editor is the best, but if you're looking for an editor and not a full IDE, chances are that you'll end up using one of those three.

```powershell
cinst visualstudiocode
```

```powershell
cinst SublimeText3
cinst sublimetext3-contextmenu
cinst SublimeText3.PackageControl
cinst SublimeText3.PowershellAlias
```

##### Visual Studio 2017

If you already own Visual Studio, you should obviously install the version you bought with your precious money. If you don't, do know that Visual Studio Community Edition is free (for [most people](https://visualstudio.microsoft.com/vs/community/)).

If you have development needs that require an older version, Visual Studio 2015 Community Edition can be installed with this command:

```powershell
cinst visualstudio2022community
```

#### Ruby

Even if you don't care about Ruby at all, bear in mind that it's preinstalled on OS X (and easy to install on Unix), so many dev tools might be trying to leverage it. For example, GitHub pages are compiled using Jekyll - if you want to get in on that, install Ruby.

```powershell
cinst ruby
cinst ruby.devkit
```

#### Rust

Rust development on Windows is fantastic. I applaud the whole community for their cross-platform efforts. To get started, install `rustup`, the official installer for Rust. The easiest way to aquire it is to download it directly [from the homepage](https://rustup.rs).

`rustup` allows you to install so-called "Rust toolchains". I personally use the nightly one (because that's how I roll; but also, because many libraries require it):

```powershell
rustup install nightly-msvc
rustup default nightly-msvc
```

From this point on forward, you can install additional Rust tools to make development easier with `cargo`.

#### Go

Sure, why not.

```powershell
cinst golang
```

#### Python

Python is a complex world with a bunch of flavors, but Python 3 and pip should have you prepared for most things.

```powershell
cinst python
cinst pip
```

Some older Node.js packages still require Python 2.7. Don't use this version unless you know you need it:

```powershell
cinst python2
```

### DevOps

This stuff is really only relevant if you're interested in DevOps - but if you are, you should probably install the stuff below. It's not likely that you need any of the things below unless you're directly working with it, because none of those things are expected to be installed on a Unix machine.

#### Docker, VirtualBox, Vagrant

If you want to run Docker machines and images, you might not need VirtualBox. In fact, installing VirtualBox on your system isn't always the best idea, given that it messes with a few system components.

Docker released a version for OS X and Windows that no longer requires VirtualBox to be installed - and instead uses the default Hypervisor that comes with the operating system (on Windows, that's Hyper-V). You can [read more about it over in Docker's documentation](https://docs.docker.com/docker-for-windows/).

```powershell
cinst docker-desktop
```

#### SSH

If you installed CMDer or Gow as indicated above, you're already set - simply run `ssh` from CMD, CMDer, or PowerShell.  If you haven't and you're running at least the Windows 10 Fall Creators Update, know that OpenSSH is now an official Windows feature. Install it from PowerShell with the following commands:

```powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'

# The above command should return something that looks like this:

# Name  : OpenSSH.Client~~~~0.0.1.0
# State : NotPresent
# 
# Name  : OpenSSH.Server~~~~0.0.1.0
# State : NotPresent

# Then, knowing the version numbers:

# Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

# Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

Alternatively (and for older versions of Windows), install the official Microsoft port of OpenSSH directly:

```powershell
cinst win32-openssh
```

#### Azure Cli

Super duper useful if you're working with Azure at all.

```powershell
cinst azure-cli
```

#### AWS Cli

Super duper useful if you're working with Amazon Web Services at all.

```powershell
cinst awscli
cinst awstools.powershell
```

### Basic Tools

You'll recognize many of these names. Nothing here is crazy unique, it's just stuff you probably want installed to have a well-running machine.

```powershell
cinst vlc
cinst GoogleChrome
cinst 7zip.install
cinst sysinternals
cinst paint.net
cinst spotify
```

If you're not running Windows 10, also install:

```powershell
cinst DotNet3.5 -- not needed on windows 8
cinst DotNet4.0 -- not needed on windows 8
cinst DotNet4.5 -- not needed on windows 10
cinst PowerShell -- not needed on windows 10
```

### Remove unwanted packages

Windows comes with a bunch of Store apps you might not want.

```
Get-AppxPackage Facebook.Facebook | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage TuneIn.TuneInRadio | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage Microsoft.MinecraftUWP | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage Microsoft.MicrosoftSolitaireCollection | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage KeeperSecurityInc.Keeper | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage 2FE3CB00.PicsArt-PhotoStudio | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage 9E2F88E3.Twitter | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name *Twitter | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name *MarchofEmpires | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name king.com.* | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.3DBuilder | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name *Bing* | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.Office.Word | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.Office.PowerPoint | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.Office.Excel | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.MicrosoftOfficeHub | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name DellInc.PartnerPromo | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.Office.OneNote | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.MicrosoftSolitaireCollection | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.SkypeApp | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.YourPhone | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name *XBox* | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.MixedReality.Portal | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.Microsoft3DViewer | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name SpotifyAB.SpotifyMusic | Remove-AppxPackage -ErrorAction SilentlyContinue
Get-AppxPackage -AllUser -Name Microsoft.MSPaint | Remove-AppxPackage -ErrorAction SilentlyContinue # Paint3D
```