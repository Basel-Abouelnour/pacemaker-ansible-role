# pacemaker-ansible-role
## Inventory file
```ini
	[all]
	target-1 http_service=httpd
	target-2

	[targets]
	target-1
	target-2
```
## Deploy the cluster
playbook
```yaml
	---
	- hosts: targets
  	  become: yes
	  roles:
      - pacemaker
```
```bash
	ansible-playbook deploy-cluster.yml -i hosts.ini 
```

## Delete the cluster
```yaml
	- hosts: targets
		become: true
		gather_facts: false
		tasks:
		- name: Stop cluster
		  command: pcs cluster stop --all
		  run_once: true
		  failed_when: false

		- name: Disable cluster at boot
		  command: pcs cluster disable --all
		  run_once: true
	      ailed_when: false

		- name: Destroy cluster configuration
		  command: pcs cluster destroy --force
	      run_once: true
		  failed_when: false

```
```bash
	ansible-playbook delete-cluster.yml -i hosts.ini 
```
