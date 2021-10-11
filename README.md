# Ansible Compliance and Configuration Drift Demo

Example of using Ansible handlers to raise an error when configuration drift or noncompliance occurs.

## Problem

Typically with configuration validation or compliance detection is that we want to run Ansible in what is called "check mode" to determine if the targeted system complies with the expected or desired configuration defined by the Ansible tasks.

However, the problem with using "check mode" is that even when it detects changes are required on the targeted system (aka, system is non-compliant or has configuration drift), Ansible playbook completes with SUCCESS. What we would prefer is the following:

- Run all Ansible tasks against the targeted server
- If 1 or more configuration changes are required, raise an error but not until we run through all Ansible tasks to determine the full scope of the configuration drift/non-compliance.
- Allow notification when playbook fails, so we can inform a team about the compliance or config issue
- Allow re-running the playbook to apply or make the config changes
- Allow reporting mechanisms to show how many configuration changes are required (before making the changes)

## Solution

Using a trick using a handler event and checking the `check_mode` variable allows us to produce a failure when configuration drift or compliance issue occurs. See details in the code as to how this was implemented.

Below you will find a demo that shows the 3 main benefits: 

(1) Initially, check mode shows configuration drift or non-compliance by failing after checking all configuration items
(2) Run mode allows remediation on the configuration drift
(3) Finally, check mode now shows compliance and does not raise an error

## Examples

Use check mode to show changes are required and playbook returns with failure message

```yaml
$ ansible-playbook playbook.yml --check
PLAY [whatever] ***********************************************************************************************************************************************************************

TASK [Perform configuration validation] ***********************************************************************************************************************************************

TASK [ansible-role-validation : Include validation tasks and apply the notify keyword] ************************************************************************************************
included: roles/ansible-role-validation/tasks/validate.yml for localhost

TASK [ansible-role-validation : Make a configuration change] **************************************************************************************************************************
changed: [localhost]

TASK [ansible-role-validation : Debug message] ****************************************************************************************************************************************
ok: [localhost] => {
    "msg": "handler called yet?"
}

RUNNING HANDLER [ansible-role-validation : Raise error when changes are required in check mode] ***************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Changes are required - system is noncompliant"}

NO MORE HOSTS LEFT ********************************************************************************************************************************************************************

PLAY RECAP ****************************************************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

```

Apply the changes as reported from above

```yaml
$ ansible-playbook playbook.yml

PLAY [whatever] ***********************************************************************************************************************************************************************

TASK [Perform configuration validation] ***********************************************************************************************************************************************

TASK [ansible-role-validation : Include validation tasks and apply the notify keyword] ************************************************************************************************
included: roles/ansible-role-validation/tasks/validate.yml for localhost

TASK [ansible-role-validation : Make a configuration change] **************************************************************************************************************************
changed: [localhost]

TASK [ansible-role-validation : Non config validation task] ***************************************************************************************************************************
ok: [localhost] => {
    "msg": "handler called yet?"
}

RUNNING HANDLER [ansible-role-validation : Raise error when changes are required in check mode] ***************************************************************************************
skipping: [localhost]

PLAY RECAP ****************************************************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

```

Show system is now compliant and no changes required and playbook returns success in both check mode and normal mode.

```yaml
$ ansible-playbook playbook.yml --check

PLAY [whatever] ***********************************************************************************************************************************************************************

TASK [Perform configuration validation] ***********************************************************************************************************************************************

TASK [ansible-role-validation : Include validation tasks and apply the notify keyword] ************************************************************************************************
included: roles/ansible-role-validation/tasks/validate.yml for localhost

TASK [ansible-role-validation : Make a configuration change] **************************************************************************************************************************
ok: [localhost]

TASK [ansible-role-validation : Non config validation task] ***************************************************************************************************************************
ok: [localhost] => {
    "msg": "handler called yet?"
}

PLAY RECAP ****************************************************************************************************************************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

## Tips

### Using Diff

Try using the `diff` mode when running check mode to better understand the difference in state. 

- [More information on diff mode](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html#using-diff-mode).
- [Enforcing or preventing diff mode on tasks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html#enforcing-or-preventing-diff-mode-on-tasks)

```yaml
ansible-playbook playbook.yml --check --diff
```

### Skipping or Ignoring Tasks

Try skipping tasks or ignoring errors in check mode

- https://docs.ansible.com/ansible/latest/user_guide/playbooks_checkmode.html#skipping-tasks-or-ignoring-errors-in-check-mode

### Try using include_tasks for handlers

[More information](https://medium.com/opsops/using-block-for-handlers-in-ansible-a55f45b62a96)
