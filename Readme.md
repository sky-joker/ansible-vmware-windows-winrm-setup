# Ansible VMware Windows WinRM setup

This Playbook can do the following:

* Create certificate for WinRM
* Import certificate for WinRM to Windows running on VMware ESXi
* Confirming Remote Host Communication Using WinRM

Certificates and powershells for import are copied using the vSphere API.  
After copying, powershell is execute with vSphere API.

## Requirements

* [vmware-guest-file-operation](https://github.com/sky-joker/vmware-guest-file-operation)

## Install

Clone this repository.

```
$ git clone https://github.com/sky-joker/ansible-vmware-windows-winrm-setup.git
```

Clone [vmware-guest-file-operation](https://github.com/sky-joker/vmware-guest-file-operation) repository.

```
$ git clone https://github.com/sky-joker/vmware-guest-file-operation.git
```

Install the required python module.

```
$ pip install -r vmware-guest-file-operation/requirements.txt
$ pip install pyOpenSSL pywinrm
```

Move the directory by giving execute permission to `vmware-guest-file-operation.py`.

```
$ chmod +x vmware-guest-file-operation/vmware-guest-file-operation.py
$ mv vmware-guest-file-operation/vmware-guest-file-operation.py ansible-vmware-windows-winrm-setup
```

Move to `ansible-vmware-windows-winrm-setup` directory.

```
$ cd ansible-vmware-windows-winrm-setup
```

## Usage example

Please rewrite `vars/vmware_parameters.yml` and `vars/certificate_parameters.yml` and `inventory` according to the environment.

`vmware_parameters.yml` is a VMware parameter vars file.

```yaml
---
hostname: change to vCenter IP or hostname.
username: administrator@vsphere.local
password: change to vCenter login user password.
datacenter: change to data center name where Windows VM running.
```

`certificate_parameters.yml` is a server certificate parameter vars file.

```yaml
# certificate directory.
root_dir: change to directory name for saving certificate related files.
 
# windows
windows_save_path: C:\Users\Administrator\Desktop
powershell_absolute_path: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
 
# Parameters to sign a certificate.
certificate_term: server certificate expiration date.
ca_certificate_term: ca expiration date.
 
# Parameters to csr files.
server_certificate_parameters:
  - common_name: common name(Host name displayed by $ENV:COMPUTERNAME).
    private_key_password: private key password.
    private_key_chipher: private key chipher.
    private_key_size: private key size(bits).
    friendly_name: friendly name.
    pfx_password: password for PKCS12 for import.
    vm_name: vm name displayed on VMware.
    vm_username: user name for guest os.
    vm_password: user password for guest os.
```

If more than one certificate is required, add to `server_certificate_parameters`.  
for example:

```yaml
# Parameters to csr files.
server_certificate_parameters:
  - common_name: WINDOWS01
    private_key_password: password
    private_key_chipher: des3
    private_key_size: 2048
    friendly_name: ansible
    pfx_password: password
    vm_name: Windows01
    vm_username: Administrator
    vm_password: secret
 
  - common_name: WINDOWS02
    private_key_password: password
    private_key_chipher: des3
    private_key_size: 2048
    friendly_name: ansible
    pfx_password: password
    vm_name: Windows02
    vm_username: Administrator
    vm_password: secret
```

inventory file example.  
Add host to windows group and change `ansible_host` and `ansible_password`.

```
[windows]
windows01 ansible_host=192.168.0.155 ansible_user=Administrator ansible_password=secret
windows02 ansible_host=192.168.0.156 ansible_user=Administrator ansible_password=secret
 
[windows:vars]
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_port=5986
ansible_winrm_server_cert_validation=ignore
```

Run Playbook after modifying the vars file.

```
$ ansible-playbook main.yml -i inventory
(snip)
TASK [Wait for listening on 5986 port.] *********************************************************************************************************************************************************
ok: [windows02 -> localhost]
ok: [windows01 -> localhost]
 
TASK [Win Ping.] ********************************************************************************************************************************************************************************
ok: [windows02]
ok: [windows01]
 
PLAY RECAP **************************************************************************************************************************************************************************************
localhost                  : ok=13   changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
windows01                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
windows02                  : ok=2    changed=0    unreachable=0    failed=0    s
```

The powershell created for server certificate and certificate import is saved as follows.

```
$ tree certificate/
certificate/
|-- WINDOWS01
|   |-- WINDOWS01.crt
|   |-- WINDOWS01.csr
|   |-- WINDOWS01.key
|   `-- WINDOWS01.pfx
|-- WINDOWS02
|   |-- WINDOWS02.crt
|   |-- WINDOWS02.csr
|   |-- WINDOWS02.key
|   `-- WINDOWS02.pfx
`-- powershell
    |-- WINDOWS01.ps1
    `-- WINDOWS02.ps1
```

## License

This project is licensed under the MIT License - see the [LICENSE.txt](https://github.com/sky-joker/ansible-vmware-windows-winrm-setup/blob/master/LICENSE.txt) file for details.

