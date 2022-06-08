# Intro

Explain we won’t be talking about provisioning today, but how we can do it on VMWare, public cloud, bare metal of course etc.
1. Go to hosts -> content hosts and show all the hosts
2. Go to content -> Lifecycle environments and explain paths, and how hosts are associated with them
3. Explain the lifecycle environments.  Explain that we have versions for these: dev/qa/prod, but they could have 
    a. Database servers
    b. Web servers
    c. etc.
4. Go to content -> content views.  Show how the software is versioned and promoted through the lifecycle environments

# Applying a patch (errata)

1. Go to content -> Errata
2. Select only RHEL 7 server repos
3. Explain Applicable and Installable
4. Click applicable
5. Select 100 per page
6. Click on the applicable checkbox
7. Find the first installable errata for all 3 hosts: RHSA-2020:5566 Important: openssl security update
8. Note the date is Dec 16
9. Run it and fix the issues on both hosts
10. Note that it uses Ansible
11. Click and show the log for one of the hosts

Optional tasks:

1. Go back and click on RHSA-2021:2575 Moderate: lz4 security update
2. Run it and fix the issues on just the one host
3. Go to content views to explain why the content is not accessible to the prod host

# Installing/Updating a package

1. Go to the All hosts collection
2. Click package install removal update
3. Enter “nano” package name (editor) (or screen, vim-enhanced, bash-completion)
4. Install -> via remote execution
5. Point out that you could do an “update all packages” but it would take a long time, so we won’t be doing that!
6. Click Done
7. Watch the install

# Check if a reboot is required

1. Go to hosts -> all hosts
2. Add filter “trace_status=reboot_needed”

Refer to https://access.redhat.com/discussions/3175851

# Run a remote job

1. Go to hosts -> all hosts
2. Select action -> schedule remote job
3. Fill in text: hostname; uptime; whoami
4. Submit
5. Wait for success
6. Click on a host and check the output

# Demonstrate system roles

1. Log in to the prod host CLI
2. Show motd configuration (ie the message upon login)
3. Run chronyc commands (chronyc sources, head /etc/chrony.conf)
4. Go to configure -> host group and select the host group
5. Add the relevant roles, show the timesync and ntp parameters
6. Go back to the prod host via “All hosts” OR go to the configure -> Ansible Variables
7. Run all (Ansible) system roles
8. It takes a while depending on how many roles you added

# Show SCAP compliance

1. Go to Hosts -> Policies
2. Click on Dashboard
3. Explain that this is better for a truly disconnected environment… but ask how disconnected they need to be.
4. Can switch to cloud.redhat.com as needed and explain that there you can run remediation here if you have a smart management subscription

# Demonstrate Insights

Demonstrate this via cloud.redhat.com because our lab hosts are not connected to Insights.

# Other capabilities to discuss

- Tailoring SCAP policies
- Generating reports (Monitor -> Report Templates)
    https://www.redhat.com/en/blog/getting-started-satellite-65-reporting-engine
- Remote execution jobs


