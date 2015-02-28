## Squeegee [tech.]

Provisioning nice and shiny **Windows** operating system environments has not always been a easy task. Now, with squeegee in your toolbox you can feel the spirit of **Microsoft** again.

**Squeegee** by itself does not do much. It glues together the properties of some supergood opensource projects. So what do we got here?

- _Template-based_ creation of windows boxen ([Veewee](https://github.com/jedi4ever/veewee))
- Ruby-esque configuration of [VirtualBox](http://www.virtualbox.org/) boxen provider
- _Unattended_ installation utilizing [Windows Assessment and Deployment Kit (ADK)](http://www.microsoft.com/en-us/download/details.aspx?id=30652)
- Full support of Windows Remote Management ([WinRM](https://msdn.microsoft.com/en-us/library/aa384426(v=vs.85).aspx))
- Integrated installation of [Opscode Chef](https://www.chef.io/) 12 ([for Windows](https://www.chef.io/solutions/windows/))
- [PowerShell]() 3, [CMD.EXE](http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/cmd.mspx) and [Cygwin](http://cygwin.com/) supported
- Repeatable, _reboot resilient_ software package management using [Chocolatey](https://chocolatey.org/) packages ([Boxstarter](http://boxstarter.org/))
- No hacks or dirty tricks, industry-standard _open-source_ software (except [Windows](https://www.youtube.com/watch?v=cif674rUyNE) :)

![Building a boxen](https://raw.githubusercontent.com/gretel/squeegee/master/doc/build.gif "bundle exec veewee vbox build winbox")

## Supported Source Platform

- Apple OS X ([Yosemite](https://en.wikipedia.org/wiki/OS_X_Yosemite))

## Supported Target Platform

- Microsoft Windows [Server 2012 R2](http://www.microsoft.com/en-us/server-cloud/products/windows-server-2012-r2/default.aspx)

## Installation

### Requirements

- Ruby 2
- Git
- [Homebrew Cask](http://caskroom.io/)


#### Homebrew

If you have not installed Homebrew previously now is a good time to do so:

```
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

#### Homebrew Cask

To install binary blobs in a managed we just add a Cask room:

```
$ brew install caskroom/cask/brew-cask

```

#### VirtualBox

Now, VirtualBox can be installed easily:

```
$ brew update
$ brew cask install virtualbox
```

To update VirtualBox in a crontab or so:

```
$ brew cask update virtualbox
```

If you intend to manually interfere with VirtualBox you can lookup it's info:

```
$ brew cask info virtualbox
virtualbox: x.y.zz-bbbbb
VirtualBox
http://www.virtualbox.org
/opt/homebrew-cask/Caskroom/virtualbox/x.y.zz-bbbbb (4 files, 110M)
https://github.com/caskroom/homebrew-cask/blob/master/Casks/virtualbox.rb
==> Contents
  /Applications/VirtualBox.app/Contents/MacOS/VBoxManage (binary)
  /Applications/VirtualBox.app/Contents/MacOS/VBoxHeadless (binary)
  VirtualBox.pkg (pkg)
```

#### Clone and bundle

To resolve Rubygem dependencies [Bundler](http://bundler.io/) is used, so let's give it a call. OS X system's default Rubygems requires superuser priviliges to install gems, doh:

```
$ sudo gem install bundler
```

If you use a Ruby version from Homebrew (`brew install ruby; brew link --force ruby`) or prefer to use `ry`, `rbenv` or `direnv` you might not need no `sudo`:

```
$ gem install bundler
```

Now, have a clone of **Squeegee** from GitHub and actually install the dependencies:

```
$ git clone --depth 1 https://github.com/gretel/squeegee.git
$ cd squeegee
$ bundle
```

The result should look as satisfying as:

```
Bundle complete! 8 Gemfile dependencies, 104 gems now installed.
```

Mind the wrapping calls to `bundle` to ensure **Veewee** will be able to resolve it's dependencies. Check if the template supplied is defined as expected:

```
$ bundle exec veewee vbox list
The following definitions are available in /Users/hensel/Sync/prjcts/squeegee:
- winbox
```

Great! We are ready to go on and provision a boxen.

## Provisioning

### Session Configuration

Veewee is configured in Ruby at `vw/definitions/winbox/definition.rb`. Please see the [documentation](http://www.veewee.org/docs/vbox/) for in-depth information.

### Unattended Installation

The script generated by the ADK is at `vw/definitions/winbox/Autounattend.xml`. Please see and edit `ProductKey` and `SkipAutoActivation` to automagically activate your copies of Windows. Additional configuration keys of interest are:

```
AdministratorPassword
ComputerName
FullName
Organization
TimeZone
```

### Build and run boxen

Now the fun part starts! To begin we need to build a boxen:

```
$ bundle exec veewee vbox build winbox
```
Please wait a bit until the VirtualBox window comes up.

If you would like to overwrite a previous build do:

```
$ bundle exec veewee vbox build winbox --force
```

*Be careful!* Any changes to `vw/definitions/winbox` will be overwritten when `--force` is applied - better commit them first.

## Usage Examples

Bring up the boxen we have built, again:

```
$ bundle exec veewee vbox up
```

Take a screenshot and save it to current (local) directory:

```
$ bundle exec veewee vbox screenshot winbox screen.png
Saving screenshot of vm winbox in screen.png
```

Veewee includes a PowerShell-based approach to interactive management - you should check it out:

```
$ bundle exec veewee vbox winrm winbox
This is a simple interactive shell
To exit interactive mode, use 'quit!'
veewee> VER
Executing winrm command: VER

Microsoft Windows [Version 6.2.9200]
veewee> quit!
```

To bring up the boxen and shut it down five minuter later:

```
$ bundle exec veewee vbox up winbox; sleep 300s; bundle exec veewee vbox halt winbox
Finding unused TCP port in range: 5985 - 6025
Selected TCP port 5985
Waiting for winrm login on 127.0.0.1 with user vagrant to windows on port => 5985 to work, timeout=10000 sec
.
Executing winrm command: shutdown /s /t 10 /f /d p:4:1 /c "Vagrant Shutdown"
```

## Management

While **WinRM** offers a SSH-like approach to remote management using **Chef** nicely meets our contemporary demands. Invoke `chef-client` like this:

```
$ bundle exec veewee vbox winrm winbox chef-solo
Executing winrm command: chef-solo
{:config_missing=>true}
[2015-02-28T22:14:00-08:00] WARN: *****************************************
[2015-02-28T22:14:00-08:00] WARN: Did not find config file: C:\chef\solo.rb, using command line options.
[2015-02-28T22:14:00-08:00] WARN: *****************************************
[2015-02-28T22:14:05-08:00] INFO: *** Chef 12.0.3 ***
[2015-02-28T22:14:05-08:00] INFO: Chef-client pid: 2240
[2015-02-28T22:14:36-08:00] INFO: Run List is []
[2015-02-28T22:14:36-08:00] INFO: Run List expands to []
[2015-02-28T22:14:36-08:00] INFO: Starting Chef Run for winbox
[2015-02-28T22:14:36-08:00] INFO: Running start handlers
[2015-02-28T22:14:36-08:00] INFO: Start handlers complete.
[2015-02-28T22:14:36-08:00] FATAL: None of the cookbook paths set in Chef::Config[:cookbook_path], ["C:\\chef\\cookbooks", "C:\\chef\\site-cookbooks"], contain any cookbooks
[2015-02-28T22:14:36-08:00] ERROR: Running exception handlers
[2015-02-28T22:14:36-08:00] ERROR: Exception handlers complete
[2015-02-28T22:14:36-08:00] FATAL: Stacktrace dumped to C:/chef/cache/chef-stacktrace.out
[2015-02-28T22:14:36-08:00] FATAL: Chef::Exceptions::CookbookNotFound: None of the cookbook paths set in Chef::Config[:cookbook_path], ["C:\\chef\\cookbooks", "C:\\chef\\site-cookbooks"], contain any cookbooks
```

Whoa! Looks like somebody forget to specify the `run-list`.

Shells commands can be executed easily, like to get their result. Let's go for `SYSTEMINFO`:

```
$ bundle exec veewee vbox winrm winbox SYSTEMINFO
Executing winrm command: SYSTEMINFO

Host Name:                 WINBOX
OS Name:                   Microsoft Windows Server 2012 Standard Evaluation
OS Version:                6.2.9200 N/A Build 9200
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:
Registered Organization:   Adobe Systems GmbH
Product ID:                00183-90000-00001-AA422
Original Install Date:     2/28/2015, 1:39:43 PM
System Boot Time:          2/28/2015, 10:07:20 PM
System Manufacturer:       innotek GmbH
System Model:              VirtualBox
System Type:               x64-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: Intel64 Family 6 Model 42 Stepping 7 GenuineIntel ~2008 Mhz
BIOS Version:              innotek GmbH VirtualBox, 12/1/2006
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     2,048 MB
Available Physical Memory: 1,422 MB
Virtual Memory: Max Size:  2,997 MB
Virtual Memory: Available: 2,316 MB
Virtual Memory: In Use:    681 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              N/A
Hotfix(s):                 3 Hotfix(s) Installed.
                           [01]: KB2887535
                           [02]: KB2887536
                           [03]: KB2887537
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Desktop Adapter
                                 Connection Name: Ethernet
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.0.2.2
                                 IP address(es)
                                 [01]: 10.0.2.15
                                 [02]: fe80::6d8d:97e0:dae3:e495
Hyper-V Requirements:      VM Monitor Mode Extensions: No
                           Virtualization Enabled In Firmware: No
                           Second Level Address Translation: No
                           Data Execution Prevention Available: Yes
```

More in-depth information can be provided by calling `ohai`:

```
$ bundle exec veewee vbox winrm winbox ohai
Executing winrm command: ohai
```
```json
{
  "cpu": {
    "0": {
      "vendor_id": "GenuineIntel",
      "family": "2",
      "model": "10759",
      "stepping": null,
      "physical_id": "CPU0",
      "cores": 2,
      "model_name": "Intel64 Family 6 Model 42 Stepping 7",
      "mhz": "2008",
      "cache_size": " KB"
    },
    "total": 2,
    "real": 1
  },
  "filesystem": {
    "A:": {
      "kb_size": 0,
      "kb_available": 0,
      "kb_used": 0,
      "percent_used": 0,
      "mount": "A:",
      "volume_name": null
    },
    "C:": {
      "kb_size": 10485755,
      "kb_available": 29253,
      "kb_used": 10456502,
      "percent_used": 99,
      "mount": "C:",
      "fs_type": "ntfs",
      "volume_name": "Windows 2012"
    },
```

## Bootstrap cookbook

In addition to **Squeegee** a Chef cookbook allows to further configure and setup the Windows system in a industry-standard manner. By default a cookbook named `squeegee-run` is run by **Chef**.

Please see the documentation on [squeegee-run](https://github.com/gretel/squeegee-run) for further information.

## License and Authors

Author:: Tom Hensel (adobe@jitter.eu (tom87213@adobe.com))
