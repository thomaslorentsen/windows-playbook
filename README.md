__This is for experimentation and not a production ready solution.__

# Manage Windows Servers in Ansible
This page documents a minimal way of managing windows servers with ansible.

If you are starting an instance in AWS.
You can find Windows Server quite easiy, I'd suggest Windows Server 2019.
Make sure you create the private key so you can decrypt the password.

You can open the following ports in security group
- 3389 (So you can use Microsoft Remote Desktop to connect)
- 5986 (WinRM over HTTPS)
- 5985 (WinRM over HTTP)

You can read how to set up [WinRM](https://docs.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management#quick-default-configuration) but follow the guide found on [Ansible](https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html#winrm-setup) website. 

It's worth running this command first to check the powershell version.
It is likely that you are running a compatible version of powershell and no patching or upgrade is required.
```powershell
Get-Host | Select-Object Version
```
Run this script
```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
$url = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

powershell.exe -ExecutionPolicy ByPass -File $file
```
You can check it is working
```powershell
winrm enumerate winrm/config/Listener
```

Configure the hosts
```
[windows-server]
windows ansible_winrm_server_cert_validation=ignore
```
I [ignore](https://stackoverflow.com/questions/45381063/ansible-winrm-server-cert-validation-https-security) the self signed certificate.

You can [Use Ansible with Chocolatey](https://www.ntweekly.com/2020/09/09/install-chocolatey-with-ansible-on-windows-hosts/).
When the playbook runs it will install chocolatey.

```yaml
- hosts: windows-server
  tasks:
    - debug:
        msg: Hello, world!
    - name: Install chocolatey
      win_chocolatey:
        name:
        - chocolatey
        - chocolatey-core.extension
        state: present
  vars:
    # Specifies winrm as the connection method
    ansible_connection: winrm
    # Username and password
    ansible_user: Administrator
    ansible_password: "secret"
```

In OSX you need to pass ```no_proxy=*``` to disable kerberos.
Run the playbook with:
```bash
no_proxy=* ansible-playbook -i hosts.ini playbook.yml
```

You get the following:
```
➜  windows no_proxy=* ansible-playbook -i hosts.ini playbook.yml

PLAY [windows-server] ******************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************
ok: [52.210.138.41]

TASK [debug] ***************************************************************************************************************************************************
ok: [52.210.138.41] => {
    "msg": "Hello, world!"
}

TASK [Install chocolatey] **************************************************************************************************************************************
[WARNING]: Chocolatey was missing from this system, so it was installed during this task run.
changed: [52.210.138.41]

PLAY RECAP *****************************************************************************************************************************************************
52.210.138.41              : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Todo
Install docker using the [powershell package]
(https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/set-up-environment?tabs=Windows-Server).
