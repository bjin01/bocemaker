# bocemaker
collection of pacemaker stuff

## boping agent

crm configure edit
```
primitive rsc_ping ocf:pacemaker:ping \
        op stop interval=0 \
        op start timeout=60 interval=0s \
        op monitor interval=90 timeout=60 start-delay=60 on-fail=fence \
        params peer=pxesap03 failure_score=1000 debug=true pidfile="/var/run/ping.pid" dampen=10s multiplier=1000 host_list="192.168.122.1 192.168.122.220" name=pingval
clone cln_ping rsc_ping \
        meta maintenance=false target-role=Started
```

## Parameters

* *__peer__*: the peer to ping. This is my custom parameter added to default ping resource agent. In a >2 node cluster we can specify one node as peer. The purpose is if local ping fails a triage function will be done to check if peer node also fails to ping. If peer node also fails to ping or the peer node is already unavailable then we assume the issue might not be a local and we set the ping score as successful and don't reduce the failure-score, and avoid fencing the node. It is safer to not cause fail-over if the issue is not local. If the peer node is available then the ping score will be reduced by the failure_score. If the score is under exceeded then the node will be fenced. 

If the peer host itself ping fails then it will not run the triage function. The failure-score will be reduced and the node will be fenced if the score is under exceeded. So ideally the peer host should not run critical resources and should be a dedicated node for triage function. E.g. sbd diskless setup.

The triage function in boping is done by using ssh to the peer node and run the ping command. The ssh key is required to be set up between the nodes so that passwordless ssh can be used.

* failure_score: the score to be added to the node if the ping fails. This is the default parameter of the ping resource. E.g. if two ping destinations are specified then the score can be set to 2000. If the failure_score will be under exceeded then the node will be fenced. Each successful ping to one destination will get the score by 1000 because of the multiplier 1000. 




