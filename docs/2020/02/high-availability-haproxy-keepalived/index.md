# High Availability - HAProxy and Keepalived


![](/forgetful/images/high-availability-haproxy-keepalived-ha-diagram-animated.gif)

# HAProxy

```bash
# file /etc/haproxy/haproxy.cfg

... ...

frontend www
    bind load_balancer_anchor_IP:80 # to which the floating IP binds
    default_backend nginx_pool

backend nginx_pool
    balance roundrobin
    mode tcp
    # below two are the App Servers
    server web1 web_server_1_private_IP:80 check
    server web2 web_server_2_private_IP:80 check
```

# Keepalived

```bash
# file /etc/keepalived/keepalived.conf

vrrp_script chk_haproxy {
    script "pidof haproxy"  # how to do the health check of local
    interval 2  # every two secs
}

vrrp_instance VI_1 {
    interface eth1  # be connected via eth1

    # the initial value that keepalived will use until the daemon 
    # can contact its peer and hold an election
    state MASTER

    # used to decide which member is elected.
    # the decision is simply based on which server has the highest 
    # number for `priority` setting.
    priority 200

    virtual_router_id 33
    unicast_src_ip primary_private_IP
    unicast_peer {
        secondary_private_IP
    }

    # basic measure to ensure that the peer being contacted is
    # legitimate. auth_pass is a shared secret that will be used by
    # both nodes.
    authentication {
        auth_type PASS
        auth_pass password
    }

    track_script {
        chk_haproxy
    }

    # be executed whenever this node becomes the `master` of the 
    # pair. This script will be responsible for triggering the 
    # floating IP address reassignment.
    notify_master /etc/keepalived/master.sh
}
```
