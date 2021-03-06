# Installing Shaken Fist

This guide will assume that you want to install Shaken Fist on a single local machine (that is, the one you're going to run ansible on). This is by no means the only installation option, but is the most simple to get people started. There is additional detail in the README.md file of the deploy repository if you need more help.

Shaken Fist only supports Ubuntu 18.04 or later, so if you're running on localhost that implies that you must be running a recent Ubuntu on your development machine. Note as well that the deployer installs software and changes the configuration of your networking, so be careful when running it on machines you are fond of.

Create a directory for Shaken Fist, and then checkout the deployer git repository there:

```bash
mkdir shakenfist
cd shakenfist
git clone https://github.com/shakenfist/deploy
cd deploy/ansible
```

And install some dependancies:

```bash
sudo apt-get update
sudo apt-get -y dist-upgrade
sudo apt-get -y install ansible tox pwgen build-essential python3-dev python3-wheel curl
ansible-galaxy install andrewrothstein.etcd-cluster andrewrothstein.terraform andrewrothstein.go
```

## A local developer installation

Shaken Fist uses ansible as its installer, with terraform to bring up cloud resources. Because we're going to install Shaken Fist on localhost, there isn't much terraform in this example. Installation is run by a simple wrapper called "deployandtest.sh". This wrapper checks out required git repositories, runs ansible plays, and then performs CI testing of the installation. It should be noted that CI testing is currently destructive (don't run it against production!), and not supported for installs with fewer than three machines. It is therefore skipped for a localhost install.

We also make the assumption that developer laptops move around more than servers. In a traditional install we detect the primary NIC of the machine and then use that to build VXLAN meshes. For localhost single node deploys we instead create a bridge called "brsf" and then use that as our primary NIC. This means your machine can move around and have different routes to the internet over time, but it also means its fiddly to convert a localhost install into a real production cluster. Please only use localhost installs for development purposes.

```bash 
CLOUD=localhost ./deployandtest.sh
```

<aside>The deployer clones a number of git repositories that it needs to build a working Shaken Fist installation. As a developer, you might want to move these out of shakenfist/deploy/gitrepos to somewhere more obvious once the installer has finished running. You can just symlink the repositories to the location that the deployer users and things will work as expected. Note that the deloyer does not clone all repositories, just those it needs, so you might still need to clone other repositories.</aside>

If you want to install a specific release, you can set the RELEASE environment variable. Possible options are:

* Any valid pypi release version number.
* "git:master" for the current master branch of each repository.
* "git:branch" for a specific branch. If that branch does not exist in a given repository, master is used instead.