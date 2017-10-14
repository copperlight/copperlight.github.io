# Running Ansible Playbooks on Windows

<div class="meta">
  <span class="date"><small>2014-06-29</small></span>
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

## But First, Some History

In early 2006, running almost a thousand servers for Blackboard Product Development that were evenly
distributed across Windows, Linux and Solaris, we needed an automation tool that would allow us to
quickly deploy and configure the Blackboard Learning Management System (LMS).  The real sticking
point for us was managing the Windows ecosystem.

The [first commit](https://github.com/puppetlabs/puppet/commit/54e9b5e3561977ea063417da12c46aad2a4c1332)
of PuppetLabs Puppet occurred in April 2005 and the [first tagged release](https://github.com/puppetlabs/puppet/tree/0.9.3)
was on Jan 3, 2006.  The [first commit](https://github.com/opscode/chef/commit/b5117775e86cff40399187b6292c98fba9dc5034)
of OpsCode Chef occurred in March 2008 and the [first tagged release](https://github.com/opscode/chef/tree/0.5.2)
was on Jan 31, 2009.  Needless to say, the modern configuration management ecosystem was much sparser
in 2006 then as compared to today.  Puppet was still very new and in the process of gaining mind share
and adding functionality; it did not support Windows at first.  [CFEngine](http://cfengine.com/) was
available, but it also did not support Windows in the open source version.

I worked with Dave Carter at the beginning of 2006 to develop our own in-house configuration
management system that we called Fusion.  It was based on [Ant](http://ant.apache.org/) and
[Ant-Contrib](http://ant-contrib.sourceforge.net/tasks/tasks/index.html).  Since Blackboard was a
Java based application, we always had one or more versions of JDKs installed on our systems as a
part of the imaging or virtual machine cloning process, so picking a tool that ran on the JDK made
sense and offered us the platform independence we needed.  Dave developed a state machine with a
socket listener that would accept XML-formatted messages and then kick off various tasks.  I
developed a library and property inheritance hierarchy for the the system, along with a parallel job
execution client and added the set of scripts that deployed and configured the Blackboard LMS.  I
figured out that by cherry-picking a few key utilities out of the [UnxUtils distribution](http://unxutils.sourceforge.net/),
I could write Windows batch scripts in a manner similar to Linux bash scripts and this lent to a
somewhat manageable level of consistency between the disparate operating systems without completely
abandoning the hooks we needed for Windows.

## Motivation

Anyone who has spoken with me in the past year about my work knows that [Ansible](http://www.ansible.com/home)
is hands-down my favorite piece of software tooling.  Using Ansible, I was able to effectively
manage 500-odd Ubuntu Linux systems at Blackboard, half of which were deployed in a VPC at AWS and
half of which were deployed in the Blackboard Managed Hosting data centers.  Part of the challenge
that we had at Blackboard in the Product Development department is that 30-40% of the several
thousand servers used for development and testing were Windows, which made it difficult to choose a
single configuration management system to rule them all.  For the better part of a year, I kept
saying that it was a terrible shame that Ansible did not support Windows and that Opscode Chef would
probably be the best choice for configuration management of all systems, since it arguably had the
best support for that platform in 2013.  After meeting with [Michael DeHaan](https://www.linkedin.com/in/michaeldehaan),
creator and CTO of Ansible, at Blackboard headquarters, we talked through their rough plans for
Windows support.  In short, it was something they wanted to be thoughtful about.  Some time later,
we had a hack day organized at Blackboard and I decided that I would attempt to develop an Ansible
playbook that could install and uninstall a JDK on Windows, using my previous experience with
building the Fusion configuration management system.

It turns out that this works. Quite well. Even though it wasn't intended to.

## Ansible Roadmap Update

On June 19, 2014, Michael DeHaan announced [Windows Is Coming](http://www.ansible.com/blog/windows-is-coming).
PowerShell remoting is a far cleaner solution and I am looking forward to seeing it hit the release
branch, although I don't have to worry about Windows machines so much these days.  I learned this
nifty fact from [Ansible Weekly Issue 38](https://devopsu.com/newsletters/ansible-weekly/38.html);
this is not a bad way to keep up on the latest Ansible news.

## Pre-Requisites

Windows + Cygwin + SSHd + Python

## Playbooks

Ansible inventory file `hosts`:

```ini
[w7x64-jf]
w7x64-jf.pd.local
```

JDK7 installation playbook `jdk7_install.yml`:

```yaml
- name: install oracle jdk7
  hosts:
  - w7x64-jf
  user: administrator
  gather_facts: false
  vars:
    version: 7u45
    build: b18
    version_padded: 1.7.0_45
    dodrootca2: c:\\jdk\{{version_padded}}\\jre\\lib\\security\\dod.root-ca-2.pem
    cacerts: c:\\jdk{{version_padded}}\\jre\\lib\\security\\cacerts
  tasks:
  - command: wget -P /usr/local/src --no-cookies --no-check-certificate --header Cookie:gpw_e24=http%3A%2F%2Fwww.oracle.com http://download.oracle.com/otn-pub/java/jdk/{{version}}-{{build}}/jdk-{{version}}-windows-x64.exe creates=/usr/local/src/jdk-{{version}}-windows-x64.exe

  - file: path=/usr/local/src/jdk-{{version}}-windows-x64.exe mode=0755

  - shell: /usr/local/src/jdk-{{version}}-windows-x64.exe /s INSTALLDIR=c:\\jdk{{version_padded}} /INSTALLDIRPUBJRE=c:\\jre{{version_padded}} REBOOT=Suppress ADDLOCAL=ToolsFeature,SourceFeature,PublicjreFeature AUTOUPDATE=0 SYSTRAY=0 SYSTRAY=0 /L c:\\cygwin\\usr\\local\\src\\jdk-install-log.txt creates=/cygdrive/c/jdk{{version_padded}}

  - copy: src=../certs/dod.root-ca-2.pem dest=/cygdrive/c/jdk{{version_padded}}/jre/lib/security/dod.root-ca-2.pem

  - command: /cygdrive/c/jdk{{version_padded}}/bin/keytool -import -trustcacerts -alias dodrootca2 -file {{dodrootca2}} -keystore $cacerts -storepass changeit -noprompt
    ignore_errors: true

  - copy: src=files/cygdrive_c_java_jre_lib_security_US_export_policy.jar dest=/cygdrive/c/jdk{{version_padded}}/jre/lib/security/US_export_policy.jar

  - copy: src=files/cygdrive_c_java_jre_lib_security_local_policy.jar dest=/cygdrive/c/jdk{{version_padded}}/jre/lib/security/local_policy.jar

  - shell: /cygdrive/c/jdk{{version_padded}}/bin/java -version 2>&1 |head -1 |awk '{print $3}' |sed -e 's/"//g'
    register: java_version

  - fail: msg="The Java version does not match the expected value {{ version_padded }}."
    when: "'{{ java_version.stdout }}' != '{{ version_padded }}'"
```

JDK7 uninstallation playbook `jdk7_uninstall.yml`:

```yaml
- name: uninstall oracle jdk7
  hosts:
  - w7x64-jf
  user: administrator
  gather_facts: false
  vars:
    version: 7u45
    version_padded: 1.7.0_45
    version_text: "7 Update 45"
  tasks:
  - template: src=templates/usr_local_src_remove-programs.vbs dest=/usr/local/src/remove-programs.vbs

  - shell: cscript remove-programs.vbs |grep "Java {{version_text}} (64-bit)" chdir=/usr/local/src
    register: result
    ignore_errors: true

  - command: cscript remove-programs.vbs /uninstall "Java {{version_text}} (64-bit)" chdir=/usr/local/src
    when: result|success

  - shell: cscript remove-programs.vbs |grep "Java SE Development Kit {{version_text}} (64-bit)" chdir=/usr/local/src
    register: result
    ignore_errors: true

  - command: cscript remove-programs.vbs /uninstall "Java SE Development Kit {{version_text}} (64-bit)" chdir=/usr/local/src
    when: result|success

  - file: path={{item}} state=absent
    with_items:
    - /usr/local/src/jdk-{{version}}-windows-x64.exe
    - /usr/local/src/jdk-install-log.txt
    - /usr/local/src/remove-programs.vbs
```

The `remove-programs.vbs` helper script:

```vbscript
If Wscript.Arguments.Count = 0 Then
  inventory_software()
ElseIf Wscript.Arguments.Count = 2 Then
  If Wscript.Arguments(0) = "/uninstall" Then
    'Expecting: cscript remove-programs.vbs /uninstall "Java(TM) 6 Update 26"
    'Expecting: cscript remove-programs.vbs /uninstall "Java(TM) SE Development Kit 6 Update 26"
    uninstall_software(Wscript.Arguments(1))
  Else
    Wscript.Echo "Usage: remove-programs.vbs [/uninstall ""software""]"
  End If
Else
  Wscript.Echo "Usage: remove-programs.vbs [/uninstall ""software""]"
End If

Sub inventory_software()
  strComputer = "."

  Set objWMIService = GetObject("winmgmts:" _
    & "{impersonationLevel=impersonate}!\\" _
    & strComputer & "\root\cimv2")
  Set colSoftware = objWMIService.ExecQuery _
    ("Select * from Win32_Product")

  For Each objSoftware in colSoftware
    Wscript.Echo "Name: " & objSoftware.Name
    'Wscript.Echo "Version: " & objSoftware.Version
  Next
End Sub

Sub inventory_java_software()
  strComputer = "."

  Set objWMIService = GetObject("winmgmts:" _
    & "{impersonationLevel=impersonate}!\\" _
    & strComputer & "\root\cimv2")
  Set colSoftware = objWMIService.ExecQuery _
    ("Select * from Win32_Product " _
        & "Where Name Like 'Java%'")

  For Each objSoftware in colSoftware
    Wscript.Echo "Name: " & objSoftware.Name
    Wscript.Echo "Version: " & objSoftware.Version
  Next
End Sub

Sub uninstall_software(strApplicationName)
  'Make sure to run this with Administrator privileges
  strComputer = "."

  Set objWMIService = GetObject("winmgmts:" _
    & "{impersonationLevel=impersonate}!\\" _
    & strComputer & "\root\cimv2")
  Set colSoftware = objWMIService.ExecQuery _
    ("Select * From Win32_Product Where Name = '" _
    & strApplicationName & "'")

  For Each objSoftware in colSoftware
    objSoftware.Uninstall()
  Next
End Sub
```

## Demonstration

File system state before install:

```bash
administrator@W7X64-JF~
$ ls -l /cygdrive/c |egrep "jdk|jre"
drwx------+ 1 SYSTEM SYSTEM 0 Jun 24 2010 jdk5
drwx------+ 1 SYSTEM SYSTEM 0 Oct 4 2012 jdk6
drwx------+ 1 SYSTEM SYSTEM 0 Oct 4 2012 jdk7
```

Install JDK7 on Windows:

```bash
copperpro:windows-hack mjohnson$ ansible-playbook -i hosts jdk7_install.yml

PLAY [install oracle jdk7] ****************************************************

TASK: [command wget -P /usr/local/src --no-cookies --no-check-certificate --header Cookie:gpw_e24=http%3A%2F%2Fwww.oracle.com http://download.oracle.com/otn-pub/java/jdk/7u45-b18/jdk-7u45-windows-x64.exe creates=/usr/local/src/jdk-7u45-windows-x64.exe] ***
changed: [w7x64-jf.pd.local]

TASK: [file path=/usr/local/src/jdk-7u45-windows-x64.exe mode=0755] ***********
changed: [w7x64-jf.pd.local]

TASK: [shell /usr/local/src/jdk-7u45-windows-x64.exe /s INSTALLDIR=c:\\jdk1.7.0_45 /INSTALLDIRPUBJRE=c:\\jre1.7.0_45 REBOOT=Suppress ADDLOCAL=ToolsFeature,SourceFeature,PublicjreFeature AUTOUPDATE=0 SYSTRAY=0 SYSTRAY=0 /L c:\\cygwin\\usr\\local\\src\\jdk-install-log.txt creates=/cygdrive/c/jdk1.7.0_45] ***
changed: [w7x64-jf.pd.local]

TASK: [copy src=../certs/dod.root-ca-2.pem dest=/cygdrive/c/jdk1.7.0_45/jre/lib/security/dod.root-ca-2.pem] ***
changed: [w7x64-jf.pd.local]

TASK: [command /cygdrive/c/jdk1.7.0_45/bin/keytool -import -trustcacerts -alias dodrootca2 -file c:\\jdk1.7.0_45\\jre\\lib\\security\\dod.root-ca-2.pem -keystore c:\\jdk1.7.0_45\\jre\\lib\\security\\cacerts -storepass changeit -noprompt] ***
changed: [w7x64-jf.pd.local]

TASK: [copy src=files/cygdrive_c_java_jre_lib_security_US_export_policy.jar dest=/cygdrive/c/jdk1.7.0_45/jre/lib/security/US_export_policy.jar] ***
changed: [w7x64-jf.pd.local]

TASK: [copy src=files/cygdrive_c_java_jre_lib_security_local_policy.jar dest=/cygdrive/c/jdk1.7.0_45/jre/lib/security/local_policy.jar] ***
changed: [w7x64-jf.pd.local]

TASK: [shell /cygdrive/c/jdk1.7.0_45/bin/java -version 2>&1 |head -1 |awk '{print $3}' |sed -e 's/"//g'] ***
changed: [w7x64-jf.pd.local]

TASK: [fail msg="The Java version does not match the expected value 1.7.0_45."] ***
skipping: [w7x64-jf.pd.local]

PLAY RECAP ********************************************************************
w7x64-jf.pd.local : ok=8 changed=8 unreachable=0 failed=0
```

File system state after install and before uninstall:

```bash
administrator@W7X64-JF ~
$ ls -l /cygdrive/c |egrep "jdk|jre"
drwx------+ 1 SYSTEM SYSTEM 0 Dec 12 23:17 jdk1.7.0_45
drwx------+ 1 SYSTEM SYSTEM 0 Jun 24 2010 jdk5
drwx------+ 1 SYSTEM SYSTEM 0 Oct 4 2012 jdk6
drwx------+ 1 SYSTEM SYSTEM 0 Oct 4 2012 jdk7
drwx------+ 1 SYSTEM SYSTEM 0 Dec 12 23:17 jre1.7.0_45
```

Uninstall JDK7 on Windows:

```bash
copperpro:windows-hack mjohnson$ ansible-playbook -i hosts jdk7_uninstall.yml

PLAY [uninstall oracle jdk7] **************************************************

TASK: [template src=templates/usr_local_src_remove-programs.vbs dest=/usr/local/src/remove-programs.vbs] ***
changed: [w7x64-jf.pd.local]

TASK: [shell cscript remove-programs.vbs |grep "Java 7 Update 45 (64-bit)" chdir=/usr/local/src] ***
changed: [w7x64-jf.pd.local]

TASK: [command cscript remove-programs.vbs /uninstall "Java 7 Update 45 (64-bit)" chdir=/usr/local/src] ***
changed: [w7x64-jf.pd.local]

TASK: [shell cscript remove-programs.vbs |grep "Java SE Development Kit 7 Update 45 (64-bit)" chdir=/usr/local/src] ***
changed: [w7x64-jf.pd.local]

TASK: [command cscript remove-programs.vbs /uninstall "Java SE Development Kit 7 Update 45 (64-bit)" chdir=/usr/local/src] ***
changed: [w7x64-jf.pd.local]

TASK: [file path=$item state=absent] ******************************************
changed: [w7x64-jf.pd.local] => (item=/usr/local/src/jdk-7u45-windows-x64.exe)
changed: [w7x64-jf.pd.local] => (item=/usr/local/src/jdk-install-log.txt)
changed: [w7x64-jf.pd.local] => (item=/usr/local/src/remove-programs.vbs)

PLAY RECAP ********************************************************************
w7x64-jf.pd.local : ok=6 changed=6 unreachable=0 failed=0
```

File system state after uninstall:

```bash
administrator@W7X64-JF ~
$ ls -l /cygdrive/c |egrep "jdk|jre"
drwx------+ 1 SYSTEM SYSTEM 0 Jun 24 2010 jdk5
drwx------+ 1 SYSTEM SYSTEM 0 Oct 4 2012 jdk6
drwx------+ 1 SYSTEM SYSTEM 0 Oct 4 2012 jdk7
```
