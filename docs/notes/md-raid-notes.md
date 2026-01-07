---
title: 'MD/RAID notes'
taxonomy:
    category:
        - docs
---

## Raid Notes

### Script
This is a test script for setting up md/raid 10far2
```
#!/bin/bash
dev1='/dev/sdd'
dev2='/dev/sde'
name='test'
offset='1M'
size='400G'
partnum='1'
parttype='fd00'

if ! [[ $(which sgdisk) =~ (sgdisk) ]]; then
    echo "install gptfdisk package"
    exit 1
fi


echo "prepare ${dev1}"
sgdisk --zapall ${dev1}
echo "sgdisk --mbrtogpt ${dev1}"
sgdisk --mbrtogpt ${dev1}
echo "sgdisk --new ${partnum}:+${offset}:+${size} --typecode ${partnum}:${parttype} --change-name ${partnum}:\"${name}1\" ${dev1}"
sgdisk --new ${partnum}:+${offset}:+${size} --typecode ${partnum}:${parttype} --change-name ${partnum}:${name}1 ${dev1}


echo "prepare ${dev2}"
echo "sgdisk --zapall ${dev2}"
sgdisk --zapall ${dev2}
echo "sgdisk --mbrtogpt ${dev2}"
sgdisk --mbrtogpt ${dev2}
echo "sgdisk --new ${partnum}:+${offset}:+${size} --typecode ${partnum}:${parttype} --change-name ${partnum}:\"${name}2\" ${dev2}"
sgdisk --new ${partnum}:+${offset}:+${size} --typecode ${partnum}:${parttype} --change-name ${partnum}:${name}2 ${dev2}
  
echo "create md/raid"
mdadm --create --verbose --level=10 --metadata=1.2 --chunk=512 --raid-devices=2 --layout=f2 /dev/md/${name} ${dev1}${partnum} ${dev2}${partnum}

echo "status ..."
cat /proc/mdstat
```
## Note on number of disk in setup
### Four disk setup
```
mdadm --create --verbose --level=10 --metadata=1.2 --chunk=512 --raid-devices=4 --layout=f2 /dev/md/webdata /dev/sda2 /dev/sdb2 /dev/sdc2 /dev/sdd2
```
### Two disk setup
```
mdadm --create --verbose --level=10 --metadata=1.2 --chunk=512 --raid-devices=2 --layout=f2 /dev/md/${name} /dev/sdc2 /dev/sdd2
```
## To memorize
Status raid building
```
cat /proc/mdstat
```
Pipe setup to `mdadm.conf`
```
mdadm --detail --scan >> /etc/mdadm.conf
```
Assembly
```
mdadm --assemble --scan
```
## Formatting raid
[Calculating the stride and stripe width][2]

* chunk `mdadm --detail /dev/mdX | grep 'Chunk Size'`
* block `4k`
* numdisks = `N`
* `N` for a *raid0* array of `N` 
* `N-1` for *raid5*
* `N*2` for *raid10,far2* array of `N*2`

```
stride = chunk / block
stripe-width = numdisks * stride
```

#### Format four disk setup
```
mkfs.ext4 -v -L ${name} -b 4096 -E stride=128,stripe-width=512 /dev/${name}
```

#### Format two disk setup
```
mkfs.ext4 -v -L ${name} -b 4096 -E stride=128,stripe-width=256 /dev/${name}
```



[1]: https://wiki.archlinux.org/index.php/RAID
[2]: https://wiki.archlinux.org/index.php/RAID#Calculating_the_stride_and_stripe_width