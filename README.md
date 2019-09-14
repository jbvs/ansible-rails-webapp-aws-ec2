# Ansible: Deploy Ruby on Rails (RoR) App on AWS EC2 Instance
These playbooks create a new _AWS EC2 instance_ running _GNU/Linux Ubuntu 18.04_ in the same RoR application with the following different settings:

1. Playbook: ```aws_provisioning_solution1.yml```:

* RoR
* MySQL database community server
* Node.js
* RVM
* _Passenger_
* Nginx: http and https
* ufw
* Fail2ban

2. Playbook: ```aws_provisioning_solution2.yml```:

* RoR
* MySQL database community server
* Node.js
* RVM
* _Puma App Server_
* Nginx: http and https
* ufw
* Fail2ban

The first playbook implements _Passenger_ that integrates with _Nginx_, allowing the App to speak HTTP and manages the app's processes and resources while the second playbook implements _Puma_ App Server that was built for speed and parallelism using _Nginx_ running in reverse proxy mode.

## Prerequisites & Configurations
Modules to install on host machine that runs the playbooks 

- Python 3.x
	- Modules: 
  ```boto``` and ```boto3```

	Install python modules with pip module manager:

	```$ pip install boto boto3```

	or

	```$ pip3 install boto boto3```

- Ansible 2.8
    - Modules: 
    ```rvm1-ansible```

    Install ansible module:

    ```$ ansible-galaxy install rvm.ruby```

- Rename template files

  Rename the following template files and adjust them as shown in the next topics:

  ```
  $ cd nsible-rails-webapp-aws-ec2
  $ mv ansible.cfg.template ansible.cfg
  $ mv ./group_vars/all.template ./group_vars/all
  ```

- Edit ansible configuration file (```ansible.cfg```) and add the instance key pair
	- How to create a key pair on [AWS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
	I tried to create the key pair automaticlly through the playbook, but showed some errors... :(
	- Example, put the key pair name in ```ansible.cfg``` on ```private_key_file``` directive:
	```
	[defaults]
    #interpreter_python = auto_silent
    host_key_checking = False
    #private_key_file = !!!!PUT KEY PAIR NAME HERE WITH EXTENTION NAME!!!!
    private_key_file = my_key_pair.pem
    deprecation_warnings = False
	```

- Save your key pair file in the same directory of ```ansible.cfg``` config file and set permitions:

	```$ chmod 400 my_key_pair.pem```

- Save your AWS keys (access and secret) from IAM account:

	- How to create an AWS keys from [IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access_keys.html#Using_CreateAccessKey) account
	After create the AWS keys you may save ```.csv``` file.

	- Use _Ansible Vault_ to security save sensitive data:

	  Inside ```ansible-rails-webapp-aws-ec2``` main directory execute:

	  ```$ ansible-vault create aws_keys.yml```

	  Edit and add the following to it:
	
    ```
	  aws_access_key: JKTVI6IM34O4VZJPT54X
	  aws_secret_key: jWcFEU8vtf8ZEILVq3+zyZO+Z2JjzM1JjieoRZSz
	  ```
	  Save the file all content will be encrypted.
	  ```$ cat aws_keys.yml```

- Create SSH key pair for authenticating ```deploy``` user.

  This is useful when ansible is running to make the tasks and after for ssh logon passwordless.

  ```$ ssh-keygen -t rsa -b 4096 -C "your_email@domain.com"```

- Add SSH private key to the ```ssh-agent```:

  IMPORTANT: Very single playbook executation you must add SSH private key to the agent, because after, playbook creates the new instance and uses to connect through ssh with ```deploy```user.

  ```$ ssh-add```

- Edit ansible global configuration file ```group_vars\all```:

  - Adjust ```key_pair_aws``` with the name of key pair created without extension name.
  - Adjust ```ssh_public_key_files``` with full path of ssh private key.

```
---
# AWS:
# Keypair AWS Instance
# key_pair_aws: !!! PUT YOUR KEY PAIR NAME HERE WITHOUT EXTENSION NAME!!!
key_pair_aws: my_key_pair
aws_region: sa-east-1
aws_instance_type: t2.micro
aws_security_group: webservers_sg
aws_image: ami-02a3447be1ec3a38f
#
# General settings
deploy_app_name: webapp
deploy_dir: /home/deploy/
deploy_user: deploy
deploy_passwd: 5tgb6yhn
# group who runs passenger
deploy_group: www-data
deploy_password: 4esz5rdx
create_dirs:
  - log
  - tmp
#
# MySQL
mysql_root_db_password: 1qaz2wsx
mysql_app_db_password: 4esz5rdx
#
# Authorized Hosts
# This copies your local public key to the remote machine
# for passwordless login. Modify this!
ssh_public_key_files:
#  - /home/SET_YOUR_LOCAL_USERNAME_HERE/.ssh/id_rsa.pub
  - /Users/juliano/.ssh/id_rsa.pub
#
# Ruby: RVM
ruby_version: 'ruby-2.6.3'
#
# github: App Repo to deploy
github_repo: 'https://github.com/jbvs/webapp.git'
```

## Execute Playbook
1. To create instance and configurations using playbook 1:
   ```
   ansible-playbook -i hosts --ask-vault-pass aws_provisioning_solution1.yml
   Vault password:
   ```
   Type ansible vault password

2. To create instance and configurations using playbook 2:
   ```
   ansible-playbook -i hosts --ask-vault-pass aws_provisioning_solution2.yml
   Vault password:
   ```
   Type ansible vault password

3. To destroy the instance (delete image):
   ```
   ansible-playbook -i hosts --ask-vault-pass aws_destroy_ALL.yml
   Vault password:
   ```


Now access the new fresh AWS EC2 instances running WebApp

```http://<public_ip_address>``` or ```https://<public_ip_address>```

```
$ ssh deploy@<public_ip_address>
$ sudo su -
```

## Feedback
Feel free to send feedback or report problems via GitHub.

## License
[MIT](https://en.wikipedia.org/wiki/MIT_License)
