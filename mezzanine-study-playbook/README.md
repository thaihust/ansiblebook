### Useful commands

- Create roles directory structure: `ansible-galaxy init -p roles mezzanine-installer`
- Pre-tasks and Post-tasks:

```sh
---

- hosts: mezzanine

  pre_tasks:
    - name: update the apt cache
      apt: update_cache=yes

  roles:
    - mezzanine_installer

  post_tasks:
    - name: notify Slack that the servers have been updated
      local_action: >
        slack
        domain=acme.slack.com
        token={{ slack_token }}
        msg="web server {{ inventory_hostname }} configured"

  tags:
    - mezzanine
```
