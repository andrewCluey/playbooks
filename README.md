# Readme
This repository contains Ansible Playbooks and related content that can be re-used for configuration of Windows & VMware infrastructure.

While this Readme contains some high level information on how the playbooks can be used, the key information on what a Playbook does can be found in the comments of the playbook itself.


## Repository Directory Structure
This repository has been organised in such a way so as to make it easier to re-use both the generic 'Windows' playbooks, but also the 'service specific' playbooks.

By cloning the entire repository to a host with Ansible installed, it should be relatively straightforward to use the playbooks for configuring a Windows Server. Cloning the repository will result in a directory satructure that will be similar to this:

```
     ./ansible
      |
      |__playbooks
         |
         |__windows
         |   |_add-msmq.yml
         |   |_coreweb.yml
         |   |_dotnet3.yml
         |   |_dotnet4.yml
         |
         |__vmware
         |   |_
         |   |_   
         |   |_
         
```
The 'windows' directory contains playbooks that can be re-used to perform specific tasks on Windows Servers that are relatively common. Each of these playbooks will perform a specific task (such as installing .Net v3) and can easily be re-used in a 'Master' playbook using the 'Import_Playbook' parameter. Examples of this can be seen in the 'atlas.yml' and 'ISIS.yml' playbooks.

The master playbooks can be found Within the .\playbooks directory.

### Re-usable Playbooks
Within the windows directory of this repository you will find several re-usable playbooks for Windows hosts. These have been created as generic as possible.

Additional detail on what each playbook does can be found in the .yml file itself.

The intention is that this directory becomes a library of re-usable content that can be referred back to for future use. When adding a re-usuable playbook, try to pick a descriptive name for the file. For example, dotnet4.yml will install .net core framework 4.5.

## Ansible Overview

### YAML file structure
Nearly every YAML file will start with a list, identified with a '- '. All members of the list start at the same indentation level within the file (using spaces not tabs). 

Each item within the list is a key/value pair. For example, here a play has been written to install IIS on a Windows Server. The playbook itself has a :
  - name:     Give the playbook a name. Sufficiently high level to describe what the playbook is intended to do.
  - hosts:    The hosts that the playbook with be applied to (here 'Windows').
  - tasks:    A sublist of tasks (plays) that make up the playbook. Each play beneath the tasks is a separate list in itself and should be further indented (again, indent with spaces, not tabs).

 Each of the plays beneath the tasks will also require a:
  - name:                   Such as Install IIS
  - Ansible module name:    The module being used (here 'win_feature'). A detailed help file on all the supported Ansible modules can be found on the Ansible web site  [HERE](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html "Ansible module list").

Further modules can also be found on Ansible Galaxy  [HERE](https://galaxy.ansible.com/home "Ansible module list"). Galaxy is the repository for modules created by the community.

```yml
- name: Setup Web
  hosts: windows
  tasks: 
    - name: Install IIS (Core Web-Server & tools)
      win_feature:
        name: Web-WebServer, Web-Common-Http
        state: present
```


Ansible also supports the use of variables in playbooks, so rather than hardcoding elements such as the hosts to apply the play to, you can replace with a variable that can then be specified when launching the playbook. In YAML files, variables can be entered as follows:

```yml
hosts: '{{ host }}'

```
The variables can either be defined in the YAML file itself or in the command when executing the ansible playbook.

The section below shows the same Play installing IIS as we saw above, but here it is using a variable for the hosts parameter.

```yml
---                                     # Indicate the start of the yml file with "---"
- name: Setup Web                       # first play should start with "-". Give it a name.
  hosts: '{{ host }}'                   # The hosts to apply this play to. Here a variable is used.
  tasks:                                # The tasks that make up the play.
    - name: Install web server feature  # Indent the first task using spaces (not tabs). Use "-" to identify the first task. Give it a name.
      win_feature:                      # The ansible module being used. At the same level as the task name.
        name: Web-WebServer             # Indent the parameters of the module being used. Use spaces, not tabs.
        state: present                  # Additional parameters. Mostly at the same level. 
```

The command line parameter "--extra-vars", specified when executing the playbook, allows you to define the variable:

```bash
        ansible-playbook web.yml --extra-vars "host=web01.local"
```



### Ansible Inventory



### Dynamic Inventory



### Securing files with Ansible Vault

Ansible vault is a feature that allows you to encrypt data such as files and strings.

For example, you can encrypt a 'group variable' file so as to protect the cedentials that ansible uses to connect to hosts. Also, vault allows specific strings to be encrypted and the secured content then copied into clear text playbooks. Ansible v2.4 onwards allow multiple vaults to be added into a single playbook.

To create a new vault encrypted file:
```bash
        ansible-vault create foo_vars.yml
```    
When you create a new vault encrypted file, you will be required to enter the password twice. From there you will be taken straight to a blank text editor, ready to add content.

Alternatively, you can encrypt an existing file:
```bash
        ansible-vault encrypt group_vars.yml
```
Again you will be required to enter a password twice.

Once encrypted, the only way to read or edit the file will be thriogh using either the vault edit command, or to decrypt it.

To edit an encrypted file using ansible-vault:
```bash
        ansible-vault edit foo_vars.yml
```
You will need to enter the password that you set when the file was encrypted. Once authorised, the file will open in the default text editor for your system. When you've finished editing the file, save it as normal and exit the text editor. The file will remain encrypted once saved.

#### Running a playbook with a vault encrypted group_vars file
The 'group_vars' files are where specific variables and connection details are stored for groups of hosts. Common details include host usernames, password and other coonection parameters such as authentication type.

By default, ansible will look for a group_vars file with the same name as the inventory group that has been specified (here the 'Windows' group). When the windows.yml group_vars file is secured with vault, the playbook witll fail to run unless the vault password is specified when executing the command. The ` --ask-vault-pass ` parameter instructs ansible to prompt for the vault password.

```bash
        ansible-playbook playbook.yml --extra-vars "host=windows" --ask-vault-pass
```



### Securing a string using ansible vault
As well as securing whole files, ansible vault can be used to encrypt a provided string. The encrypted string can then be included in a playbook as an encrypted variable. 

To encrypt a string from the command line:

```bash
      ansible-vault encrypt_string --vault-id dev@prompt --stdin-name 'variable-name'
```


` --vault-id `

  
  The vault identity to use. Here, 'dev' is the id of the vault and 'prompt' is the method used to assign the vault secret. Other options include entering a path to a file.


` encrypt_string `
  

  Encrypt the supplied string using the provided vault secret.


` --stdin-name `

  
  Specify a name that will be used for the variable within the playbook.


The './windows/create-localuser.yml' playbook has an example of a vault encrypted variable.


### Specifiying multiple vault passwords
```bash
sudo ansible-playbook create-file.yml --extra-vars "host=atlas" --ask-vault-pass --vault-id file@prompt
```


## Managing Windows Servers

By default, Ansible uses SSH to connect to and manage hosts. As Windows does not use SSH for remote management, Ansible connects to windows hosts using WinRM.

Some additional configuration to WinRM is required however, before Ansible can connect to the Windows host, fortunately Ansible provide a Powershell script that simplifies this configuration. The script is available [HERE](https://github.com/ansible/ansible/blob/devel/examples/scripts/ConfigureRemotingForAnsible.ps1 "Configure Ansible Remoting Powershell Script") and can be run from any Windows system with Powershell v3.0, various parameters can be set when executing the script to change settings such as the default authentication mechanism.

The table below shows the authentication methods available for WinRM.


| Option       | Local Accounts  | Active Directory Account  | Credential Delegation    |
|--------------|-----------------|---------------------------|--------------------------|
|  Basic       |  Yes            |  No                       |  No                      |
|  Certificate |  Yes            |  No                       |  No                      |
|  Kerberos    |  No             |  Yes                      |  Yes                     |
|  NTLM        |  Yes            |  Yes                      |  No                      |
|  CredSSP     |  Yes            |  Yes                      |  Yes                     |

For full details on how to run the ConfigureAnsibleRemoting.ps1 script, along with explanations of the available parameters, please refer to the comments within the script itself.


## Displaying gathered facts




