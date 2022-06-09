# Deploying the Environment


## RHPDS

1. Log in to RHPDS at https://rhpds.redhat.com/
2. In the catalog, search for "AAP2 Ansible & Smart Management Workshop"
3. Select Size "Training" so you only get one instances, and fill out the other values before submitting
4. Wait for the workshop to deploy
5. Click the link provided in the resulting email e.g. http://ac21.example.opentlc.com
6. Enter your name and an email address (note that these aren't recorded anywhere).
7. Note the workbench SSH user, likely to be student1, and the workbench password
8. Note the workbench SSH control node DNS access command, e.g. `ssh student1@student1.ac21.example.opentlc.com`

# Prepare Satellite and the RHEL nodes

1. Complete all setup tasks for Exercise 0: Setup here:
https://aap2.demoredhat.com/exercises/ansible_smart_mgmt/0-setup/

Note: You can skip CentOS-related exercises if not demonstrating convert2rhel

## On your Linux host with git Ansible installed

1. Copy your SSH key to the control node using the workbench password to log in:
```
ssh-copy-id student1@student1.ac21.example.opentlc.com .
```
2. Test the SSH access, which should work without a password
```
ssh student1@student1.ac21.example.opentlc.com
```
3. Clone this repo on the control node:
```
cd
git clone https://github.com/benblasco/satellite_demo_config.git
```

4. Change into the repo directory
```
cd satellite_demo_config
```

5. Grab the SSH private key from the control node
```
scp student1@student1.ac21.example.opentlc.com:.ssh/id_rsa .
```

6. Edit the inventory file, `hosts` updating the ansible_ssh_common_args with the correct control node SSH user and DNS name e.g.
```
ansible_ssh_common_args='-o ProxyCommand="ssh -p 22 -W %h:%p -q student1@student1.ac21.example.opentlc.com"'
```

7. Edit `vars/main.yml`, replacing the password with the workbench password.  Take care to preserve the single quotes around the password.

8. Install the pre-requisite Ansible collections
```
ansible-galaxy collection install -r requirements.yml
```

9. Run the Satellite configuration playbook in "check" mode, and then in regular mode if there are no issues in "check" mode:
```
ansible-playbook -i hosts satelliteconfigure.yml --check
ansible-playbook -i hosts satelliteconfigure.yml
```

10. Run the RHEL node configuration playbook:
```
ansible-playbook -i hosts rhelconfigure.yml
```
Note: This playbook cannot be run in "check" mode without generating failures.

## Config in satellite

### Create a new version of the CV

Put a filter that includes all packages without errata (ie the base versions of the RHEL packages)

Note ticking of "include all RPMs with no errata" below

Add another filter to only include erratum up to 2020-12-31


Now you can see what version 2.0 looks like, as it has significantly less packages available and less errata.


Publish

Edit and rename the filter to include errata up to 2021-02-28
Publish

Edit and rename the filter to include errata up to 2021-06-30
Publish again

Promote 2020-12-30 to prod
Promote 2021-02-28 to QA
Promote 2020-06-30 to Dev

Outcome:


### Register the servers using AAP template:
Next, run the SERVER / RHEL7 - Register job template by clicking thelaunchto launch.

Node1 -> prod
Node2 -> dev
Node1 -> qa



Node1 updates
Install    1 Package  (+2 Dependent packages)
Upgrade  116 Packages

Node2 updates
Install    1 Package  (+2 Dependent packages)
Upgrade  117 Packages

Node3 updates
Install    1 Package  (+2 Dependent packages)
Upgrade  118 Packages

### ALREADY AUTOMATED Configure the remote execution correctly on the hosts

```
sudo -i
yum install -y katello-host-tools.noarch katello-host-tools-tracer.noarch katello-host-tools-fact-plugin.noarch

katello-package-upload
katello-enabled-repos-upload
```


Configure the SSH key for the service account rexuser
```
sudo -i
useradd -d /home/rexuser -m -c "Remote execution user" -G wheel rexuser
passwd rexuser
(redhat123)
su - rexuser
ssh-keygen
curl https://satellite.example.com:9090/ssh/pubkey >> ~rexuser/.ssh/authorized_keys
chmod 700 ~rexuser/.ssh/authorized_keys
```

Edit the sudoers file so that members of wheel group don't need a password, and this includes rexuser
```
visudo
## Same thing without a password
%wheel  ALL=(ALL)       NOPASSWD: ALL
```

### ALREADY AUTOMATED Configure remote execution successfully in satellite


Administer->settings->remoteexecution
Change ssh user from root to rexuser

```
# hammer settings set --name remote_execution_ssh_user --value rexuser
```

### ALREADY AUTOMATED Remove remote execution requirement so that katello agent is not required

Bug in Satellite 6.10 at the moment

https://access.redhat.com/solutions/6667031

Another way to enable this is:
```
# hammer settings set --name remote_execution_by_default --value true
```


### ALREADY AUTOMATED Fix view of errata with content views

administer -> settings -> content -> "Installable errata from content view"
change from no to yes

Another way to enable this is:
```
# hammer settings set --name errata_status_installable --value true
```

### Create host collection

Create a hosts collection called "All RHEL hosts collection" and add all 3 RHEL hosts to it

### Create host group

Configure -> host groups
Name: "All RHEL hosts group"
Save
Go to hosts -> all hosts
Add all hosts
Actions: Change groups
Add all the RHEL hosts to the "All RHEL hosts group"
go back and check the host group

# Make sure you can show SCAP compliance

Import the foreman-scap-client role as per instructions here

https://www.redhat.com/en/blog/deploying-openscap-satellite-using-ansible

# Install ansible roles

1. register the satellite server via subscription-manager using your own account
2. Follow instructions here: 
https://www.redhat.com/en/blog/satellite-host-configuration-rhel-system-roles-powered-ansible
or https://www.jazakallah.info/post/how-to-use-ansible-roles-for-remote-management-from-satellite-server


# Timesync system role

https://www.redhat.com/en/blog/satellite-host-configuration-rhel-system-roles-powered-ansible

https://www.redhat.com/en/blog/advanced-ansible-variables-satellite

Be sure to select the rhel system roles not the upstream linux system roles

Configure -> Ansible -> variables

[{"hostname":"0.au.pool.ntp.org","iburst":"yes"},{"hostname":"1.au.pool.ntp.org","iburst":"yes"}]


# Fix subscription manifest if it goes stale

Content -> subscriptions

Manifest refresh

# Pull updates on demand?

Check if you want to pull updates on demand for each repo, or whether you would prefer to keep a downloaded copy for speed and reliability.
