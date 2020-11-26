cordon and drain the node you want to downscale
get its instanceID
go to EC2-> AutoScaling Groups -> choose your autoscaling group -> instance management 
mark all the nodes, and unmark the node you drained
go to actions -> set-scalein protection
downscale to -1
wait until the node is closed
remove scale in protection from the other nodes