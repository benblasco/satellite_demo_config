# Intro

Explain we won’t be talking about provisioning today, but how we can do it on VMWare, public cloud, bare metal of course etc.
1. Go to hosts -> content hosts and show all the hosts
2. Go to content -> Lifecycle environments and explain paths, and how hosts are associated with them
3. Explain the lifecycle environments.  Explain that we have versions for these: dev/qa/prod, but they could have 
    a. Database servers
    b. Web servers
    c. etc.
4. Go to content -> content views.  Show how the software is versioned and promoted through the lifecycle environments

# Applying a patch (errata).  This functionality is currently broken in Satellite 6.10 due to a bug.  You will see an error message saying: "This action uses katello-agent, which is currently disabled. Use remote execution instead."

1. Go to Content -> Errata
2. Select only RHEL 7 server repos
3. Explain Applicable and Installable
4. Click applicable
5. Select 100 per page
6. Click on the applicable checkbox
7. Find the first installable errata for all 3 hosts: RHSA-2020:5566 Important: openssl security update
8. Note the date is Dec 16
9. Click the Apply Errata button to install it and fix the issues on all impacted hosts
10. Note that it uses Ansible
11. Click and show the log for one of the hosts

Optional tasks:

1. Go back and click on RHSA-2021:2575 Moderate: lz4 security update
2. Run it and fix the issues on just the one host
3. Go to content views to explain why the content is not accessible to the prod host

# Installing/Updating a package

1. Go to the Hosts -> Host Collections -> All (RHEL) hosts collection
2. Click "Package Installation, Removal, and Update"
3. Enter “nano” package name (editor) (or screen, vim-enhanced, bash-completion)
4. Install -> via remote execution
5. Point out that you could do an “update all packages” but it would take a long time, so we won’t be doing that!
6. Click Done
7. Watch the install

# Check if a reboot is required

1. Go to Hosts -> All Hosts
2. Add filter “trace_status=reboot_needed” (without the quotation marks), and press "Search".

Refer to https://access.redhat.com/discussions/3175851

# Run a remote job

1. Go to hosts -> All Hosts
2. Select action -> schedule remote job
3. Fill in text: hostname; uptime; whoami
4. Submit
5. Wait for success
6. Click on a host and check the output

# Demonstrate system roles

Using instructions from: 
https://www.redhat.com/en/blog/satellite-host-configuration-rhel-system-roles-powered-ansible
and
https://www.redhat.com/en/blog/advanced-ansible-variables-satellite


0. Open a terminal CLI session on node1
1. Start with Hosts -> all hosts view
2. Configure -> Host groups. Explain all hosts are here
3. Click the host group
4. Show the hosts in the host group
5. Back
6. Click Ansible roles
7. Add rhel-system-roles.timesync
8. Submit
9. Configure -> Ansible Variables
10. Filter on timesync
11. Point out the flag, then click the config. Ensure that the line below is pasted exactly as is, and then saved:
`[{"hostname":"0.au.pool.ntp.org","iburst":"yes"},{"hostname":"1.au.pool.ntp.org","iburst":"yes"}]`
13. Go back to hosts -> all hosts
14. Alt tab to CLI of node1
15. Run the following commands on the host
`more /etc/chrony.conf`
`chronyc sources`
16. Back to GUI
17. Ensure the hosts are selected
18. Action -> Run all Ansible roles
19. Watch it execute on a specific node
20. Go back and check all completed successfully
21. Check on the CLI again

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


