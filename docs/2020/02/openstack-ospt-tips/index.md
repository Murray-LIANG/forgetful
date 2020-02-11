# OpenStack ospt Tips


## SSH to the VM where ospt is installed

```bash
$ ssh guest@10.245.48.66
# use password: welcome

$ ssh stack@ubuntu-perf-test
# use password: welcome

$ cd ~/perf-test

# Replace the <mgmt_ip> in `ospt_*.sh`.

# create hosts and luns.
$ ./ospt_create.sh
# it outputs a `create-servers.log` and `create-volumes.log` where you can see the time used.

# attach and detach luns to hosts
$ ./ospt_attach_detach.sh 
# it outputs lots of log files with format `attach-x-x.log` and `detach-x-x.log`.
# you can compare the time used.
```

## More tips

### 1. Use the DETAIL images
