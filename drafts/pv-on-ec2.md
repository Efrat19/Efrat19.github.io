sudo amazon-linux-extras install epel -y
sudo yum install pv
sudo dd if=/dev/zero bs=50M count=100000 | pv | sudo dd of=100GiB_file_4
$(sudo dd if=/dev/zero bs=8M count=300000 of=300GiB_file)&