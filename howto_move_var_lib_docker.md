# How to move /var/lib/docker

If you'll ever run out of space on the partition where docker stores its data, the simplest solution is to move the complete /var/lib/docker folder somewhere else and insert a symlink to its new location.

Of course you could also reconfigure docker to use an other directory, but exporting and importing all the stuff from your local repository becomes time consuming fast.

Additionally I strongly opt against modifying default configurations when it can be easily fixed with symlinks. There are always waiting update problems down the line.

```bash
# change to superuser 
sudo su

# the new location of the docker folder (MODIFY THIS!)
dest=/var/data/vm

# stop all containers
docker ps -q | xargs docker kill

# stop service/daemon
service docker stop

# unmount any leftovers
cd /var/lib/docker/devicemapper/mnt
umount ./*

# move and symlink
cd
mv /var/lib/docker $dest
ln -s $dest/docker /var/lib/docker

# start service/daemon again
service docker start
```

