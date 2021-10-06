# Ansible Configuration Validation Demo

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

Demo playbook to produce a failure when check mode shows changes are required

## Examples

```yaml
# Show changes are required and playbook returns with failure message
$ ansible-playbook playbook.yml --check
PLAY [whatever] ***********************************************************************************************************************************************************************

TASK [Perform configuration validation] ***********************************************************************************************************************************************

TASK [ansible-role-validation : Include validation tasks and apply the notify keyword] ************************************************************************************************
included: /Users/jwadleig/Projects/ansible-config-validation/roles/ansible-role-validation/tasks/validate.yml for localhost

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

```yaml
# Make the changes as required
ansible-playbook playbook.yml
```

```yaml
# Show server is now compliant and no changes required and playbook returns success
ansible-playbook playbook.yml --check
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
