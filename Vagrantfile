# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT
GOVERSION="1.7"
SRCROOT="/opt/go"
SRCPATH="/opt/gopath"
# Get the ARCH
ARCH=`uname -m | sed 's|i686|386|' | sed 's|x86_64|amd64|'`
# Install Prereq Packages
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y build-essential curl git-core libpcre3-dev mercurial pkg-config zip
# Install Go
cd /tmp
wget --quiet https://storage.googleapis.com/golang/go${GOVERSION}.linux-${ARCH}.tar.gz
tar -xvf go${GOVERSION}.linux-${ARCH}.tar.gz
sudo mv go $SRCROOT
sudo chmod 775 $SRCROOT
sudo chown vagrant:vagrant $SRCROOT
# Setup the GOPATH; even though the shared folder spec gives the working
# directory the right user/group, we need to set it properly on the
# parent path to allow subsequent "go get" commands to work.
sudo mkdir -p $SRCPATH
sudo chown -R vagrant:vagrant $SRCPATH 2>/dev/null || true
# ^^ silencing errors here because we expect this to fail for the shared folder
cat <<EOF >/tmp/gopath.sh
export GOPATH="$SRCPATH"
export GOROOT="$SRCROOT"
export PATH="$SRCROOT/bin:$SRCPATH/bin:\$PATH"
EOF
cat <<EOF >>~/.bashrc
## Make sure we have the source for Docker Machine itself
go get -u github.com/docker/machine
## After login, change to docker-machine-driver-terraform directory
cd /opt/gopath/src/github.com/timperman/docker-machine-driver-terraform
EOF
sudo mv /tmp/gopath.sh /etc/profile.d/gopath.sh
sudo chmod 0755 /etc/profile.d/gopath.sh
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/ubuntu-14.04"
  config.vm.hostname = "docker-machine-driver-ddcloud"

  config.vm.provision "shell", inline: $script, privileged: false
  config.vm.synced_folder '.', '/opt/gopath/src/github.com/timperman/docker-machine-driver-terraform'

  ["vmware_fusion", "vmware_workstation"].each do |p|
    config.vm.provider p do |v|
      v.vmx["memsize"] = "4096"
      v.vmx["numvcpus"] = "2"
    end
  end

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.provider "parallels" do |prl|
    prl.memory = 4096
    prl.cpus = 2
  end
end
