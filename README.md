# The "IConrad" Linux SysAdmin Bootcamp

This repository serves as an archived record of an incredibly effective curriculum that has been shared among enthusiasts for years. While it only consists of 20 seemingly straightforward steps, it manages to encapsulate almost everything needed for learning to become a Linux systems administrator.

### :no_entry: NOTICE

This guide is now several years old, and while many of its tasks and lessons are still very relevant, the inexorable passing of time has taken its toll, and I do not recommend this anymore as it currently exists. Things like CentOS being ended make much of the specifics uncertain. So I am keeping this as a historical record of my own education, and potentially as a loose rubric for authoring a similar guide with up-to-date information.

_The content below was taken from a Reddit comment ( [`link`](https://www.reddit.com/r/linuxadmin/comments/2s924h/comment/cnnw1ma/) ). It has been modified slightly for readability and spelling, but has not been meaningfully altered._

## Background

In 2012, I personally used this as a first-pass guide to creating a curriculum to teach myself systems administration fundamentals, and to begin an in-depth education in Linux and servers to broaden my professional credentials and experience, as well as to improve my existing capabilities as a website developer. It turned out to be crucial to those goals, and is one of the more successful endeavors I have ever undertaken. To help with the effort, I opted to install a Linux desktop distribution on my daily-driver PC and immerse myself with the Linux ecosystem. It took me roughly six months from the weekend I set up my home lab to when I took most of it down and moved my servers onto Linode VPS’s.

I began my career a few years later, and to this day the knowledge I picked up remains relevant and invaluable.


## The Guide
 
1. Set up a KVM hypervisor.
2. Inside of that KVM hypervisor, install a Spacewalk server. Use CentOS 6 as the distro for all work below. (For bonus points, set up errata importation on the CentOS channels, so you can properly see security update advisory information.)
3. Create a VM to provide named (DNS) and `dhcpd` service to your entire environment. Set up the `dhcp` daemon to use the Spacewalk server as the `pxeboot` machine (thus allowing you to use Cobbler to do unattended OS installs). Make sure that every forward zone you create has a reverse zone associated with it. Use something like _ internal.virtnet_ (but not _ .local_) as your internal DNS zone.
4. Use that Spacewalk server to automatically (without touching it) install a new pair of OS instances, with which you will then create a Master/Master pair of LDAP servers. Make sure they register with the Spacewalk server. Do not allow anonymous bind, do not use unencrypted LDAP.
5. Reconfigure all 3 servers to use LDAP authentication.
6. Create two new VMs, again unattended, which will then be `postgresql` VMs. Use `pgpool-II` to set up master/master replication between them. Export the database from your Spacewalk server and import it into the new `pgsql` cluster. Reconfigure your Spacewalk instance to run off of that server.
7. Set up a Puppet master. Plug it into the Spacewalk server for identifying the inventory it will need to work with. (Cheat and use Ansible for deployment purposes, again plugging into the Spacewalk server.)
8. Deploy another VM. Install `iscsitgt` and `nfs-kernel-server` on it. Export a LUN and an NFS share.
9. Deploy another VM. Install `bakula` on it, using the `postgresql` cluster to store its database. Register each machine on it, storing to flatfile. Store the `bakula` VM's image on the `iscsi` LUN, and every other machine on the NFS share.
10. Deploy two more VMs. These will have `httpd` (Apache2) on them. Leave essentially default for now.
11. Deploy two more VMs. These will have tomcat on them. Use JBoss Cache to replicate the session caches between them. Use the `httpd` servers as the frontends for this. The application you will run is JBoss Wiki.
12. You guessed right, deploy another VM. This will utilize `iptables` for NAT/round robin load-balancing between the two `httpd` servers.
13. Deploy another VM. On this VM, install `postfix`. Set it up to use a Gmail account to allow you to have it send emails, and receive messages only from your internal network.
14. Deploy another VM. On this VM, set up a Nagios server. Have it use `snmp` to monitor the communication state of every relevant service involved above. This means doing a "is the right port open" check, and a "I got the right kind of response" check and "We still have filesystem space free" check.
15. Deploy another VM. On this VM, set up a syslog daemon to listen to every other server's input. Reconfigure each other server to send their logging output to various files on the syslog server. (For extra credit, set up `logstash` or `kibana` or `greylog` to parse those logs.)
16. Document every last step you did in getting to this point in your brand new Wiki.
17. Now go back and create Puppet manifests to ensure that every last one of these machines is authenticating to the LDAP servers, registered to the Spacewalk server, and backed up by the `bakula` server.
18. Now go back, reference your documents, and set up a Puppet Razor profile that hooks into each of these things to allow you to recreate, from scratch, each individual server.
19. Destroy every secondary machine you've created and use the above profile to recreate them, joining them to the clusters as needed.
20. **Bonus exercise**: Create three more VMs. A CentOS 5, 6, and 7 machine. On each of these machines, set them up to allow you to create custom RPMs and import them into the Spacewalk server instance. Ensure your Puppet configurations work for all three and produce like-for-like behaviors.

---

## Proposed Updates

The following are tentative ideas for modernizing this to fit more up-to-date standards.

- The above should work with CentOS 7 instead of CentOS 6 (mostly because the majority is still on `rhel7` / `centos7`)*.
- Spacewalk has also been discontinued, consider using Cockpit, Foreman, or Cloudmin as a systems manager.
- Run the `dhcpd` either from the systems management interface or FreeIPA. Bonus Points for setting up HA on 2 separate VMs ... because why not, I guess.
- FreeIPA instead of OpenLDAP.
- Replace Puppet with Saltstack.
- Use Ansible across the board for configuration management.
- Setup a server that hosts AWX so you can have Tower-like functionality with Ansible.
- Bonus points for a Gitlab instance you host yourself for your IAC.
- Host your own mail server dedicated to only that environment and configure all servers to use this server for mail transfers
- In terms of load balancing, I'd go with Traefik or Nginx instead of `iptables`

* Once done with all the tasks, migrate the servers group by group to CentOS 8 or Rocky Linux, as long-term support allows.

---

> _This repo includes information from [https://github.com/labdaddy/IConrad](https://github.com/labdaddy/IConrad) and the original Reddit comment can be found [here](https://www.reddit.com/r/linuxadmin/comments/2s924h/comment/cnnw1ma/)._
> 
> _Support is unavailable for this repo, and all information is subject to change._
