Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "fsggs"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  
  config.vm.synced_folder "./fsggs.server", "/home/vagrant/fsggs.server"
  
  config.vm.provider "virtualbox" do |vb|
    host = RbConfig::CONFIG['host_os']
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
      mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
    elsif host =~ /linux/
      cpus = `nproc`.to_i
      mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
    else
      cpus = 2
      mem = 1024
    end
    vb.customize ["modifyvm", :id, "--memory", mem]
    vb.customize ["modifyvm", :id, "--cpus", cpus]
  end
  
  config.vm.provision "shell", run: "always", privileged: true, inline: <<-SHELL
	sudo -i
	sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile
	
	if [ ! -f /home/vagrant/fsggs.server/vagrant.lock ]; then
		mkdir /home/vagrant/fsggs.server
		apt-get update
		apt-get upgrade
		apt-get dist-upgrade
		apt-get clean all
		curl -sSL https://get.docker.com/ | sh
		usermod -aG docker vagrant
		curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
		chmod +x /usr/local/bin/docker-compose
		echo ".lock" > /home/vagrant/fsggs.server/vagrant.lock
	fi
	if [ -f /home/vagrant/fsggs.server/update.sh ]; then
		/home/vagrant/fsggs.server/update.sh
		/home/vagrant/fsggs.server/restart.sh
	fi
  SHELL
end
