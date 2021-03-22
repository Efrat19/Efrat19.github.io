


use managed nodegroups

Is it possible to scale-in your EKS nodes without any downtime?

1. Use Managed Nodegroups - 
   managed nodegroups provide you with 
   so once you scale in, the node will be drained automatically before closed

2. manually - if your nodegroup is unmanaged, you can still carefully scale down, using the Scale-In protection feature.

the main problem with manual scaling is that you can not know for sure which node will be closed, therefore you are not able to drain it before it goes down.

   cordon and drain the node you want to downscale
get its instanceID
go to EC2-> AutoScaling Groups -> choose your autoscaling group -> instance management 
mark all the nodes, and unmark the node you drained
go to actions -> set-scalein protection
downscale to -1
wait until the node is closed
remove scale in protection from the other nodes