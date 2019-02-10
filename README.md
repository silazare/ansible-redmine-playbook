## AWS Redmine test deployment (Infrastructure + Services)

### Project description properties and prerequisites

This is a test task to deploy Redmine application on EC2+RDS+CDN in AWS.
Hence things are oversimplified and credentials are stored as plain text vars without vault.

- Prerequisites:
  - Ansible 2.7.5, playbook will not work on earlier versions like 2.6.X. If you are using clean host or Vagrant to run playbook - you can use requirements.txt to install needed dependencies.
  - AWS account with API access credentials.

- Notices and possible issues:
  - Cloudfront distribution has no destroy tasks, so after destroy all the infrastructure stack, need to disable and delete it manually in AWS console.
  - RDS creation will take time, timeout is defined to 15 minutes just in case.
  - Addition of a local Host Key of the new created EC2 is used to avoid manual steps during playbook probably it could be not working if you have not permissions to change ~/.ssh/known_hosts on a local machine.

### Project structure

 * [group_vars](./group_vars) -- Playbook variables folder
   * [all.yml_example](./group_vars/all.yml_example) -- Playbook variables example
 * [roles](./roles) -- Roles
   * [common](./roles/common) -- Basic packages installation
   * [infra](./roles/infra) -- AWS infrastructure create/destroy
   * [ruby](./roles/ruby) -- Ruby packages installation
   * [redmine](./roles/ruby) -- Redmine service deploy
   * [jdauphant.nginx](./roles/jdauphant.nginx) -- Galaxy role for Nginx installation
 * [site.yml](./site.yml) -- Main playbook to deploy all services

### Run steps

- Clone this repository and go to its folder (install requirements if needed):
```sh
$ git clone https://github.com/silazare/ansible-redmine-playbook.git
$ cd ansible-redmine-playbook
$ pip install -r requirements.txt
```

- Put your own AWS ssh key file (like 'EC2.pem') in the main folder, beside the playbook. File permissions should be '0600' otherwise playbook will fail.

- Prepare your own group_vars/all.yml file based on group_vars/all.yml_example:
```sh
aws_region: <set your region here, default is us-east-1
secret_key: <set your AWS secret_key here
access_key: <set your AWS access_key here

aws_keypair: <set your working AWS ssh keypair here for EC2 access, for example 'EC2'
ansible_ssh_private_key_file: <set your working AWS ssh key file name here for EC2 access, for example 'EC2.pem'
```

- Run playbook to create all services and wait until it will completed:
```sh
ansible-playbook site.yml --extra-vars="vpc_state=present"
```

- Check the Cloudfront distribution status in AWS console, its creation will take time. When status becomes Deployed and Enabled then open the service by domain name, like dxxxxxxxxxxxxx.cloudfront.net.

- Default admin credentials to Redmine are: 'admin/admin'. Service will force to set new password after first login, after it you may start use the service.

- Run playbook to cleanup all created AWS services when needed and wait until it will completed:
```sh
ansible-playbook site.yml --extra-vars="vpc_state=absent"
```

- Go to AWS console Cloudfront - disable and then destroy the distribution.
