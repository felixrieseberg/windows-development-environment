## Setup Windows 10 for Modern/Hipster Development
A fresh Windows isn't entirely ready for modern development, but all the tools you need are available. A good terminal, popular bash tools, Git, a decent package manager - when properly setup, modern development on Windows can be a lot of fun. In particular, this document outlines how to configure your Windows in such a way that it can easily handle most development tasks usually run on a Mac OS X or a Linux distro.

## A Word about Ubuntu Linux on Windows
:point_up: While [Bash on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about) isn't perfect yet, it's an amazing tool that can make development a lot easier - especially when you're dealing with Bash scripts, Ruby, or Ubuntu binaries. It's an amazing tool, but keep in mind that Git, Node, or Go are already pretty performant on Windows itself. However, they run just fine in Bash, so if you feel like moving most of your development over, go for it. Here's the how-to:

 * Ensure that you're running Windows 10 Anniversary Update (build 14311 and up)
 * Enable Developer Mode (Settings - Update & security > For developers)
 * Search for “Windows Features” and choose “Turn Windows features on or off” and enable Windows Subsystem for Linux.
 * To get Bash installed, open Command Prompt and type “bash”

## Automate it!
Below, you can see the all the things I need to actually go and work on stuff. An opiniated version of the list below can be installed automatically thanks to the magic of [Boxstarter](http://boxstarter.org/). [Check out the script to see what it runs](https://github.com/felixrieseberg/windows-development-environment/blob/master/boxstarter). It's a trimmed down version of all the tools below, leaving out duplicate tools. As an example, it installs VS Code (and not Atom or Sublime) and Git (but not Subversion or Mercurial). Simply start PowerShell as Administrator and run:

```powershell
START http://boxstarter.org/package/nr/url?https://raw.githubusercontent.com/felixrieseberg/windows-development-environment/master/boxstarter
```

## The Goods
 * [Package Management](#package-management-chocolatey)
 * [Terminal](#terminal-cmder-with-powershell-support)
 * [PowerShell Profile](#powershell-profile)
 * [Node.js](#node-and-npm)
 * [Git](#version-control-git)
 * [Atom, Sublime, VS Code](#code-editors-atom-sublime-vs-code)
 * [Ruby](#ruby)
 * [Go](#go)
 * [Python](#python)
 * [DevOps: VirtualBox, Vagrant, Docker](#devops-virtualbox-vagrant-docker)
    * [SSH](#ssh)
    * [Azure Cli](#azure-cli)
    * [AWS Cli](#aws-cli)
 * [Basic Tools](#basic-tools)

#### Package Management: Chocolatey
Chocolatey is a powerful package manager for Windows, working sort of like apt-get or homebrew. Let's get that first. Fire up CMD.exe as Administrator and run:

```powershell
@powershell -NoProfile -ExecutionPolicy Bypass -Command "iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
```

Once done, you can install packages by running `cinst` (short for `choco install`). Most packages below will be installed with Chocolatey.

###### Bonus: Use Windows 10 & OneGet
Windows 10 comes with [OneGet](https://github.com/OneGet/oneget), a universal package manager that can use Chocolatey to find and install packages. To install, run:

```powershell
Get-PackageProvider -name chocolatey
```

Once done, you can look for all Chrome packages by typing `Find-Package -Name Chrome` or install it by typing `Install-Package -Name GoogleChrome`.

#### Terminal: CMDer (with PowerShell Support)
The PowerShell in Windows 10 got a bunch of upgrades, but it's even better if used with [CMDer](https://github.com/bliker/cmder/) or [Hyper](https://hyper.is/), both powerful tools to do more command-line things with. CMDer is the old-school veteran, while Hyper hasn't been around for long. Try both and see what you like more! I _personally_ prefer Hyper, simply because it can be styled and extended with addons. Install with:

```powershell
cinst cmder -pre
cinst hyper
```

Even if you don't want to use either, you should enable your PowerShell to execute scripts. You're a developer - the terminal is your friend.

```
Set-ExecutionPolicy Unrestricted -Scope CurrentUser
```

If you want to go even further, check out the attached PowerShell Profile in this repository. It's my personal one and might not be perfect for you, but it makes my personal life a lot easier. You can edit your PowerShell profile with your favorite editor by calling `$PROFILE`, so if you're using Visual Studio Code, call `code $PROFILE` (or `vim $PROFILE` - you get the idea).

#### PowerShell Profile
Now that you have a good terminal, you might wonder how you can beef it up. This isn't part of the automated setup, but I'm quite happy with my profile. I've changed my PowerShell in the following ways:

 * Add helper functions (`uptime`, `reload-profile`, `find-file`, `unzip`, and `print-path`)
 * Add equivalents for my favorite Unix commands (`df`, `sed`, `sed-recursive`, `grep`, `grepv`, `which`, `export`, `pkill`, `pgrep`, `touch`, `sudo`, `pstree`)
 * Add Git aliases (`gc` for `git checkout`, `gp` for `git pull`)
 * Set the default output to `utf8`
 * Increase the linme history to 10000
 
To edit your PowerShell profile, run `notepad.exe $PROFILE` (or use an editor of your choice). Then, add the following:

<details>
<summary>My PowerShell Profile</summary>
   
```ps
# Increase history
$MaximumHistoryCount = 10000

# Produce UTF-8 by default
$PSDefaultParameterValues["Out-File:Encoding"]="utf8"

# Show selection menu for tab
Set-PSReadlineKeyHandler -Chord Tab -Function MenuComplete

# Helper Functions
#######################################################

function uptime {
	Get-WmiObject win32_operatingsystem | select csname, @{LABEL='LastBootUpTime';
	EXPRESSION={$_.ConverttoDateTime($_.lastbootuptime)}}
}

function reload-profile {
	& $profile
}

function find-file($name) {
	ls -recurse -filter "*${name}*" -ErrorAction SilentlyContinue | foreach {
		$place_path = $_.directory
		echo "${place_path}\${_}"
	}
}

function print-path {
	($Env:Path).Split(";")
}

function unzip ($file) {
	$dirname = (Get-Item $file).Basename
	echo("Extracting", $file, "to", $dirname)
	New-Item -Force -ItemType directory -Path $dirname
	expand-archive $file -OutputPath $dirname -ShowProgress
}


# Unixlike commands
#######################################################

function df {
	get-volume
}

function sed($file, $find, $replace){
	(Get-Content $file).replace("$find", $replace) | Set-Content $file
}

function sed-recursive($filePattern, $find, $replace) {
	$files = ls . "$filePattern" -rec
	foreach ($file in $files) {
		(Get-Content $file.PSPath) |
		Foreach-Object { $_ -replace "$find", "$replace" } |
		Set-Content $file.PSPath
	}
}

function grep($regex, $dir) {
	if ( $dir ) {
		ls $dir | select-string $regex
		return
	}
	$input | select-string $regex
}

function grepv($regex) {
	$input | ? { !$_.Contains($regex) }
}

function which($name) {
	Get-Command $name | Select-Object -ExpandProperty Definition
}

function export($name, $value) {
	set-item -force -path "env:$name" -value $value;
}

function pkill($name) {
	ps $name -ErrorAction SilentlyContinue | kill
}

function pgrep($name) {
	ps $name
}

function touch($file) {
	"" | Out-File $file -Encoding ASCII
}

function sudo {
	$file, [string]$arguments = $args;
	$psi = new-object System.Diagnostics.ProcessStartInfo $file;
	$psi.Arguments = $arguments;
	$psi.Verb = "runas";
	$psi.WorkingDirectory = get-location;
	[System.Diagnostics.Process]::Start($psi) >> $null
}

# https://gist.github.com/aroben/5542538
function pstree {
	$ProcessesById = @{}
	foreach ($Process in (Get-WMIObject -Class Win32_Process)) {
		$ProcessesById[$Process.ProcessId] = $Process
	}

	$ProcessesWithoutParents = @()
	$ProcessesByParent = @{}
	foreach ($Pair in $ProcessesById.GetEnumerator()) {
		$Process = $Pair.Value

		if (($Process.ParentProcessId -eq 0) -or !$ProcessesById.ContainsKey($Process.ParentProcessId)) {
			$ProcessesWithoutParents += $Process
			continue
		}

		if (!$ProcessesByParent.ContainsKey($Process.ParentProcessId)) {
			$ProcessesByParent[$Process.ParentProcessId] = @()
		}
		$Siblings = $ProcessesByParent[$Process.ParentProcessId]
		$Siblings += $Process
		$ProcessesByParent[$Process.ParentProcessId] = $Siblings
	}

	function Show-ProcessTree([UInt32]$ProcessId, $IndentLevel) {
		$Process = $ProcessesById[$ProcessId]
		$Indent = " " * $IndentLevel
		if ($Process.CommandLine) {
			$Description = $Process.CommandLine
		} else {
			$Description = $Process.Caption
		}

		Write-Output ("{0,6}{1} {2}" -f $Process.ProcessId, $Indent, $Description)
		foreach ($Child in ($ProcessesByParent[$ProcessId] | Sort-Object CreationDate)) {
			Show-ProcessTree $Child.ProcessId ($IndentLevel + 4)
		}
	}

	Write-Output ("{0,6} {1}" -f "PID", "Command Line")
	Write-Output ("{0,6} {1}" -f "---", "------------")

	foreach ($Process in ($ProcessesWithoutParents | Sort-Object CreationDate)) {
		Show-ProcessTree $Process.ProcessId 0
	}
}

# Aliases
#######################################################

function pull () { & get pull $args }
function checkout () { & git checkout $args }

del alias:gc -Force
del alias:gp -Force

Set-Alias -Name gc -Value checkout
Set-Alias -Name gp -Value pull
```

</details>

#### Node and npm
A bunch of tools are powered by Node and installed via npm. This applies to you even if you don't care about Node development. If you want to install tools for React, Azure, TypeScript, or Cordova, you'll need this.

```powershell
cinst nodejs.install
```

In the future, you might want to update `npm`. On Windows, just running `npm i -g npm` doesn't always do what it should, so use `npm-windows-upgrade` instead:

```
npm install -g npm-windows-upgrade
npm-windows-upgrade
```

#### Version Control: Git
Obviously. If you want Git to be able to save credentials (so you don't have to enter SSH keys / passwords every single time you do anything), also install the Git Credential Manager for Windows.

```
cinst git.install
cinst poshgit

# Restart PowerShell / CMDer before moving on - or run
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")

cinst Git-Credential-Manager-for-Windows
cinst github
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
cinst Atom
```

```powershell
cinst SublimeText3
cinst sublimetext3-contextmenu
cinst SublimeText3.PackageControl
cinst SublimeText3.PowershellAlias
```

##### Visual Studio 2017
If you already own Visual Studio, you should obviously install the version you bought with your precious money. If you don't, do know that Visual Studio 2017 Community Edition is free (for [most people](https://www.visualstudio.com/support/legal/dn877550)).

```powershell
cinst visualstudio2017community
```

If you have development needs that require an older version, Visual Studio 2015 Community Edition can be installed with this command:

```powershell
cinst visualstudio2015community
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
Python is a complex world with a bunch of flavors, but Python 3, pip, and easy_install should have you prepared for most things.

```powershell
cinst python
cinst pip
cinst easy.install
````

If you intend to work with Node.js, know that compiling native Node modules usually requires Python 2.7 instead of Python 3. For Node development, do not install Python 3 - instead, run these commands:

```powershell
cinst python2
```

### DevOps
This stuff is really only relevant if you're interested in DevOps - but if you are, you should probably install the stuff below. It's not likely that you need any of the things below unless you're directly working with it, because none of those things are expected to be installed on a Unix machine.

#### Docker, VirtualBox, Vagrant
If you want to run Docker machines and images, you might not need VirtualBox. In fact, installing VirtualBox on your system isn't always the best idea, given that it messes with a few system components.

Docker released a version for OS X and Windows that no longer requires VirtualBox to be installed - and instead uses the default Hypervisor that comes with the operating system (on Windows, that's Hyper-V). You can [read more about it over in Docker's documentation](https://docs.docker.com/docker-for-windows/).

```powershell
cinst docker-for-windows
```

However, if you want the old VirtualBox route, go and install that stuff with:

```powershell
cinst virtualbox
cinst virtualbox.extensionpack
cinst vagrant
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
cinst DotNet3.5
```

If you're not running Windows 10, also install:

```powershell
cinst DotNet4.0 -- not needed on windows 8
cinst DotNet4.5 -- not needed on windows 10
cinst PowerShell -- not needed on windows 10
```
