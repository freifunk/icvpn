[![Build Status](https://travis-ci.org/freifunk/icvpn.svg?branch=master)](https://travis-ci.org/freifunk/icvpn)

This repository contains the tinc hosts for ICVPN-Peers and several helper scripts.

## Setup

This network requires the use of the Tinc VPN Daemon. You should use at least version 1.0.24 or higher, also the
current pre-release version 1.1pre11 seems to work just fine.

### Clone the repository
    # cd /etc/tinc/
    # git clone https://github.com/freifunk/icvpn.git
    # cd icvpn
    # cp scripts/post-merge .git/hooks/

### Create your tinc configuration

Open your favorite editor and create the /etc/tinc/icvpn/tinc.conf.

    Name=entenhausen1
    Mode=switch
More options can be found through

    # man tinc.conf
Afterwards create a keypair with tincd.

    # tincd -n icvpn -K
Hint: In version 1.1 this option was moved to the tinc binary and is called <code>generate-rsa-keys</code>.

### Execute post-merge hook
This step is necessary to populate your new configuration with infos about the metanodes.

    # ./.git/hooks/post-merge

### Set up a cronjob to update the repository in regular intervals.

    # crontab -e
and insert for example

    @daily cd /etc/tinc/icvpn/; git pull > /dev/null

## What are meta nodes?

Tinc has a ConnectTo configuration option that describes which peers on startup to connect *and* sync metadata to.

Until now (2015/4) we had roughly 74 nodes, and every node connected to each other (full mesh). Tinc however 
does not scale this way, because on each connect and disconnect all ConnectTo-lines are being notified of this
and then notify their neighbours again. Many smaller nodes seemingly could not handle the amount of metadata generated 
by this which resulted in TCP Zero Windows. They then disconnected, and reconnected, producing more metadata in
the process, which was followed by even larger nodes queueing up metadata, which resulted in all nodes taking
a massive cpu and memory hit. Memory usage of up to 1.5GB was spotted, accumulated in less than 12 hours.

However for tinc to build its network graph it is sufficient, if all nodes only exchange metadata at a few nodes,
which results in much less strain on the whole network. This is why we now use meta nodes, which are defined in
the `./metanodes` file.

Criteria for the selection of meta nodes are:

1. autonomous system diversity
2. community diversity
3. ample resources (cpu, memory, traffic)

When data needs to be transfered between two nodes, this will happen independently of those meta nodes. Through the shared
network graph a direct transfer is possible and will be tried: at first via UDP, then via TCP, then indirectly. While
indirect routing is possible the meta nodes are not required to provide forwarding for those packets.

## Contact

The maintainers can be reached at
- [icvpn@lists.funkfeuer.at](mailto:icvpn@lists.funkfeuer.at)
- [irc.hackint.org #icvpn](irc://irc.hackint.org/icvpn) ([Webchat](https://webirc.hackint.org/#ircs://irc.hackint.org/#icvpn))

We have set up IRC notifications for all repositories concerning the icvpn network.


