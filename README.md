## bird-snmp-agent

### What?

* implements an SNMP-AgentX extension for the bird-routing-daemon
* currently exposes a subset of BGP4- and OSPF-MIB (RFC4273/RFC4750)

### How?

To collect its data this agent:

* uses `birdcl` CLI of bird
* calls netstat to query information about used tcp-ports
* reads the bird-configuration-files

### Requirements

The BIRD configuration needs to have these attributes to be parsable by the script and the timestamp of protocols has to be set to seconds.

Example:
```
log syslog { debug, trace, info, remote, warning, error, auth, fatal, bug };

timeformat protocol "%s";

protocol kernel {
        learn;
        merge paths;
        persist;
        scan time 20;
        export all;
}

protocol device {
        scan time 10;           # Scan interfaces every 10 seconds
}

function check_prefix()
prefix set allowed_prefix;
{
        allowed_prefix = [ 192.168.0.0{32,32} ];
        if net ~ allowed_prefix then return true;
                return false;
}

filter filter_out {
        if check_prefix() then accept;
        else reject;
}

protocol bgp drmaster {
        local 192.168.20.1 as 64512;
        neighbor 192.168.20.2 as 65000;
        import all;
        export filter filter_out;
        source address 192.168.20.1;
}

```
