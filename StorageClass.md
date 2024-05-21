# Some notes regarding NVSHAS-8534 - To block the usage of specific storage classes

## Links

-- [setup nfs on ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)
-- [instll nfs dynamic provisioner for nfs](https://fabianlee.org/2022/01/12/kubernetes-nfs-mount-using-dynamic-volume-and-storage-class/)

## Notes I took back in 2023

[Some notes I took with scree shots](./materials/How-to-Set-Up-an-NFS-Server-on-Ubuntu.pdf)

## Existing environment

Feel free to login to play with it.

- my NFS server in the labs, IP: 10.1.45.49
- my cluster has install the nfs dynamic provisioner installed. IP: 10.1.45.40

## Some yaml
