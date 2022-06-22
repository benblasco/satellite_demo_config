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

# Prepare Satellite and the RHEL nodes using AAP Controller

1. Complete all setup tasks for Exercise 0: Setup here:
https://aap2.demoredhat.com/exercises/ansible_smart_mgmt/0-setup/

    Note: You can skip CentOS-related exercises if not demonstrating convert2rhel

2. Create a new project with the following parameters:

    Name: BBLASCO Satellite Demo Config
    SCM type GIT
    SCM URL: https://github.com/benblasco/satellite_demo_config
    Branch: aap_refactor
    Options: Clean; Update Revision on Launch

3. Configure Satellite Remote Execution by creating a template with the following parameters and then launching it:

    - Name: BBLASCO Satellite Remote Execution
    - Inventory: Workshop Inventory
    - Project: BBLASCO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: satellite_config_rex.yml
    - Credential type: Satellite_Collection
    - Credential name: Satellite Credential
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)

4. Install RHEL System Roles in Satellite by creating a template with the following parameters and then launching it.  Note: This requires your RHN username and password.  It will register the system, install the roles, and then immediately unregister the system.

    - Name: BBLASCO Satellite Install System Roles
    - Inventory: Workshop Inventory
    - Project: BBLASCO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: satellite_install_system_roles.yml
    - Credential type: Satellite_Collection
    - Credential name: Satellite Credential
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)
    - Save
    - Survey -> Add
    - Question: RHN Username
    - Answer variable name: rhn_username
    - Answer type: Text
    - Required: Yes
    - Save
    - Survey -> Add
    - Question: RHN Password
    - Answer variable name: rhn_password
    - Answer type: Password
    - Required: Yes
    - Save
    - Enable Survey via the Slider

5. Enable RHEL Remote Execution by creating a template with the following parameters and then launching it:

    - Name: BBLASCO RHEL Remote Execution
    - Inventory: Workshop Inventory
    - Project: BBLASCO Satellite Demo Config
    - Execution Environment: smart_mgmt workshop execution environment
    - Playbook: rhel_configure_rex.yml
    - Credential type: Machine
    - Credential name: Workshop Credential
    - Limit: rhel (possibly redundant as it's in the playbook)
    - Privilege escalation: yes (even if possibly redundant as it's in the playbook)

6. Continue with any other config you want to perform as per the workshop instructions.

## Config in Satellite

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

Create a hosts collection called "All RHEL hosts collection" and add all 3 RHEL hosts to it:

1. Hosts -> Host Collections -> Create Host Collection
2. Name "All RHEL hosts collection"
3. Save
4. Go to Hosts tab
5. Click Add
6. Add the RHEL hosts only


### Create host group

1. Configure -> host groups
2. Name: "All RHEL hosts group"
3. Click "Submit" button
4. Go to Hosts -> All Hosts
5. Select all RHEL hosts
6. Click Actions -> Change group
7. Select "All RHEL hosts group" from the drop down
8. Click "Submit" button
9. Go back and check that all RHEL hosts belong to the correct group (see Host Group) column

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
