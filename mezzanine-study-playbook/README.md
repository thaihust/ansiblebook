### Useful notes

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

- Config vagrant file to use ansible as a provisioner:

```sh
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Use the same key for each machine
  config.ssh.insert_key = false
  config.ssh.forward_agent = true
  
  config.vm.define "vagrant1" do |vagrant1|
    vagrant1.vm.box = "ubuntu/trusty64"
    vagrant1.vm.network "forwarded_port", guest: 80, host: 8080
    vagrant1.vm.network "forwarded_port", guest: 443, host: 8443
    config.vm.network "private_network", ip: "192.168.33.10"
  end
  
  config.vm.define "vagrant2" do |vagrant2|
    vagrant2.vm.box = "ubuntu/trusty64"
    vagrant2.vm.network "forwarded_port", guest: 80, host: 8081
    vagrant2.vm.network "forwarded_port", guest: 443, host: 8444
    config.vm.network "private_network", ip: "192.168.33.11"
  end
  
  config.vm.define "vagrant3" do |vagrant3|
    vagrant3.vm.box = "ubuntu/trusty64"
    vagrant3.vm.network "forwarded_port", guest: 80, host: 8082
    vagrant3.vm.network "forwarded_port", guest: 443, host: 8445
    config.vm.network "private_network", ip: "192.168.33.12"
    vagrant3.vm.provision "ansible" do |ansible|
      ansible.limit = 'all'
      ansible.playbook = "playbook.yml"
      ansible.groups = {
        "web"  =>  ["vagrant1"],
        "task" =>  ["vagrant2"],
        "redis" => ["vagrant3"]
      }
    end
  end
end
```
