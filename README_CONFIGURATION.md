# Config in satellite

Create a new version of the CV

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


Register the servers using AAP template:
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

# Configure the remote execution correctly on the hosts

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

# Configure remote execution successfully in satellite


Administer->settings->remoteexecution
Change ssh user from root to rexuser

```
# hammer settings set --name remote_execution_ssh_user --value rexuser
```

# Remove remote execution requirement so that katello agent is not required

Bug in Satellite 6.10 at the moment

https://access.redhat.com/solutions/6667031

Another way to enable this is:
```
# hammer settings set --name remote_execution_by_default --value true
```


# Fix view of errata with content views

administer -> settings -> content -> "Installable errata from content view"
change from no to yes

Another way to enable this is:
```
# hammer settings set --name errata_status_installable --value true
```

# Create host collection

Create a hosts collection called "all hosts collection" and add all 3 hosts to it

# Create host group

Configure -> host groups
Name: "All hosts group"
Save
Go to hosts -> all hosts
Add all hosts
Actions: Change groups
Add all the hosts to the "all hosts group"
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
