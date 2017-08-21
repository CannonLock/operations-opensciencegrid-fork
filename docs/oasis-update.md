
# Updating Software on OASIS

The applicable software versions for this document are
   *osg-version 3.1.13*
(or higher)

# About OASIS
OASIS is the OSG Application Software Installation Service. It is the recommended method to install software on the Open Science Grid. It is implemented using CernVM FileSystem (CVMFS) technology.

# About this Document
This document is a step by step explanation of how a Virtual Organization (VO) Software Adminstrator can enable the OASIS service and use it to publish and update software on OSG Worker Nodes under /cvmfs/oasis.opensciencegrid.org. For information on how to configure a client for OASIS see the [OSG CVMFS Installation documentation](cvmfs-install.md). For information on hosting your own opensciencegrid.org repository see [Oasis external repositories](OasisExternalRepositories).

# Requirements
To begin the process to distribute software on OASIS using the service hosted at the OSG GOC, you must:
   * have a personal grid certificate. The process for getting one is detailed [here](https://www.opensciencegrid.org/bin/view/Documentation/Release3/CertificateUserGet).
   * register that certificate in [OIM](http://oim.grid.iu.edu/oim/home). In order to register, the certificate has to be
   * be associated with a VO that is registered in OIM. The list of registered VOs appears [here](http://oim.grid.iu.edu/oim/vo).
   * register your certificate in the %LINK_GLOSSARY_VOMS% for your %LINK_GLOSSARY_VO%. Click on the VO name that you're associated with in the above list and in the VO page click on the _"Membership Services URL"_ to register with that VO VOMS.

# How to use OASIS
## Enable OASIS
When you are ready to distribute your software with OASIS, submit a [GOC ticket](https://ticket.grid.iu.edu/goc/submit) with a request to enable OASIS for your VO. In your request, please specify your VO and provide a list of people who will install and administer the VO software in OASIS.

The GOC will enable OASIS for your VO in [OIM](https://oim.grid.iu.edu/oim/home) and add your list of administrators to the "OASIS Managers" list (which is near the bottom of the page of information about each VO in OIM). oasis-login will then grant access to the people who are listed as OASIS managers. Any time the list is to be modified, submit another GOC ticket.

## Log in with GSISSH
The next step is to generate a proxy and log into =oasis-login.opensciencegrid.org= with =gsissh=. These commands should be run on a computer that has the [[InstallOSGClient][OSG client]] software. First make sure that your grid certificate is installed in =~/.globus/usercred.p12= on that computer and that it is mode 600, then run these commands:
<pre class="screen">
[user@client ~]$ voms-proxy-init -voms VO
[user@client ~]$ gsissh -o GSSAPIDelegateCredentials=yes oasis-login.opensciencegrid.org
</pre>

In case the user can be mapped to more than one account, specify it explicitly in a command like this
<pre class="screen">
</pre>


Instead of putting <verbatim>-o GSSAPIDelegateCredentials=yes</verbatim> on the command line, you can put it in your =~/.ssh/config= like this:
<pre class="screen">
Host oasis-login.opensciencegrid.org
GSSAPIDelegateCredentials yes
</pre>


## Install and update software
Once you log in, you can add/modify/remove content on a staging area at =/stage/oasis/$VO= where $VO is the name of the VO represented by the manager.

Files here are visible to both =oasis-login= and the Stratum 0 server (oasis.opensciencegrid.org)

NOTE that =/stage/oasis/$VO= is not your home directory, which you can use for staging purposes but is not visible in OASIS.

As OASIS manager for the VO requests an oasis update with:
<pre class="screen">
</pre>
This command queues a process to sync the content of OASIS with the content of =/stage/oasis/$VO=

=osg-oasis-update= returns immediately, but only one update can run at a time (across all VOs); your request may be queued behind a different VO. If you encounter severe delays before the update is finished being published (more than 4 hours), please file a GOC ticket.


## Limitations on repository content

Although CVMFS provides a POSIX filesystem, it does not work well with all types of content. Content in OASIS is expected to adhere to the [CVMFS repository content limitations](http://cernvm.cern.ch/portal/filesystem/repository-limits) so please review those guidelines carefully.

## Testing

After =osg-oasis-update= completes and the changes have been propagated to the CVMFS stratum 1 servers (typically between 0 and 60 minutes, but possibly longer if the servers are busy with updates of other repositories) then the changes can be visible under =/cvmfs/oasis.opensciencegrid.org= on a computer that has the [[InstallCvmfs][CVMFS client installed]]. A client normally only checks for updates if at least an hour has passed since it last checked, but people who have superuser access on the client machine can force it to check again with
<pre class="screen">
</pre>
This can be done while the filesystem is mounted (despite what the name sounds like it does not do umount/mount). If the filesystem is not mounted, it will automatically check for new updates the next time it is mounted.

In order to find out if an update has reached the CVMFS stratum 1 server, you can find out the latest =osg-oasis-update= time seen by the stratum 1 most favored by your CVMFS client with the following long command on your client machine:

<pre class="screen">
cat -v|sed -n '/^T/{s/^T//p;q;}') sec"
</pre>

# References
[CERN CVMFS home page](https://twiki.cern.ch/twiki/bin/view/CvmFS)
