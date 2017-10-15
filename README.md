[![Build Status](https://travis-ci.org/thecjharries/ansible-include_vars-issue.svg?branch=master)](https://travis-ci.org/thecjharries/ansible-include_vars-issue)

This repo pulls together all of the files necessary to reproduce a bug with [`include_vars`](http://docs.ansible.com/ansible/latest/include_vars_module.html). [`role2`'s tasks](provisioning/roles/role2/tasks/main.yml) illustrate the issue. From [the docs](http://docs.ansible.com/ansible/latest/include_vars_module.html#options), `dir` is
> The directory name from which the variables should be loaded.
> If the path is relative, it will look for the file in vars/ subdirectory of a role or relative to playbook.

`ansible-playbook` is currently just looking in the `vars` directory of the calling role, e.g. in `roles/role2/vars`. This may be a feature; if it is, the docs could be updated to read something like

> The directory name from which the variables should be loaded.
> If the path is relative and the task is inside a role, it will look inside the role's vars/ subdirectory. If the path is relative and not inside a role, it will be parsed relative to the playbook.

<!-- MarkdownTOC -->

- [Usage](#usage)
    - [Windows](#windows)
- [Results](#results)
- [Provisioning Note](#provisioningnote)

<!-- /MarkdownTOC -->

## Usage

You'll need [Vagrant](https://www.vagrantup.com/downloads.html) and [`git`](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git). This should be platform-independent ([mostly](#windows)).
```
$ git clone https://github.com/thecjharries/ansible-include_vars-issue && cd ansible-include_vars-issue
$ vagrant up
```
### Windows

You'll either need [to change your boot config](http://www.hanselman.com/blog/SwitchEasilyBetweenVirtualBoxAndHyperVWithABCDEditBootEntryInWindows81.aspx) or use `--provider=hyperv` to get this to work.

## Results

The output below is from PowerShell on a Windows 10 Insider build. It will either show up with the first `vagrant up`, or you can run `vagrant provision` to isolate it.
```
$ vagrant provision
==> default: Running provisioner: file...
==> default: Running provisioner: shell...
    default: Running: <path to file>
==> default: /bin/ansible
==> default: Skipping ansible installation
==> default:
==> default: PLAY [all] *********************************************************************
==> default:
==> default: TASK [Gathering Facts] *********************************************************
==> default: ok: [localhost]
==> default:
==> default: TASK [role1 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "role1_var": "This is from role 1"
==> default: }
==> default:
==> default: TASK [role2 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "role2_var": "This is from role 2"
==> default: }
==> default:
==> default: TASK [role1 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "role1_var": "This is from role 1"
==> default: }
==> default:
==> default: TASK [role2 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "msg": "A message above this line implies the tasks were executed"
==> default: }
==> default:
==> default: TASK [role2 : include_vars] ****************************************************
==> default: fatal: [localhost]: FAILED! => {"ansible_facts": {}, "ansible_included_var_files": [], "changed": false, "failed": true, "message": "/tmp/provisioning/roles/role2/vars/role1 directory does not exist"}
==> default:
==> default: TASK [role2 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "msg": "Not found"
==> default: }
==> default:
==> default: TASK [role2 : include_vars] ****************************************************
==> default: fatal: [localhost]: FAILED! => {"ansible_facts": {}, "ansible_included_var_files": [], "changed": false, "failed": true, "message": "/tmp/provisioning/roles/role2/vars/roles/role1/vars directory does not exist"}
==> default:
==> default: TASK [role2 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "msg": "Not found"
==> default: }
==> default:
==> default: TASK [role2 : include_vars] ****************************************************
==> default: ok: [localhost]
==> default:
==> default: TASK [role2 : debug] ***********************************************************
==> default: ok: [localhost] => {
==> default:     "role1_var": "This is from role 1"
==> default: }
==> default:
==> default: PLAY RECAP *********************************************************************
==> default: localhost                  : ok=9    changed=0    unreachable=0    failed=2
```

## Provisioning Note

I'm using [the shell provisioner](https://www.vagrantup.com/docs/provisioning/shell.html) instead of [the Ansible provisioner](https://www.vagrantup.com/docs/provisioning/ansible.html) because the issue originated on a Windows Vagrant host. I was using that script to run Ansible, so I figured it should be included.
