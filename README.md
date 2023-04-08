# Vagrant


### Дополнительные материалы

1. [Дистрибутив](https://hashicorp-releases.yandexcloud.net/vagrant)
2. [Конфигурация VirtualBox через Vagrant](https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/configuration).
3. [Офф образы](https://app.vagrantup.com/bento).

---

### Основные команды vagrant
Выключить виртуальную машину с сохранением ее состояния (т.е., при следующем vagrant up будут запущены все процессы внутри, которые работали на момент вызова suspend):
```bash
vagrant suspend
```

Выключить виртуальную машину штатным образом:
```bash
vagrant halt
```

Перезапустить ВМ:
```bash
vagrant reload
```

Если наша машина уже была запущена, но нужно применить обновление из Vagrantfile, то нужно выполнить команду:
```bash
vagrant reload --provision
```

Подключиться к SSH ВМ:
```bash
vagrant ssh
```

Узнать статус машины:
```bash
vagrant status
```

Удалить ВМ:
```bash
vagrant destroy -f
```

Запуск с логированием debug:
```bash
vagrant up --debug vagrant.log 2>&1
```

---

### Vagrantfile
```bash
Vagrant.configure("2") do |config|
	# Image name
	config.vm.box = "aagrebeshkov/CentOS8" # "aagrebeshkov/CentOS8"; "aagrebeshkov/CentOS7"; "bento/ubuntu-20.04"
	config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".vagrant/"

	# Hostname
	config.vm.hostname = "vagrant"

	# Network bridge
	config.vm.network "public_network", ip: "192.168.1.101"

	# Forward port
	config.vm.network "forwarded_port", guest: 19999, host: 19999

	# Install pacages
	config.vm.provision "shell", inline: <<-SHELL
		echo "Upgrade system!!!"
		sudo yum update -y
		sudo yum install -y wget curl net-tools
	SHELL

	config.vm.provider "virtualbox" do |v|
		# Name VM
		v.name = "CentOS8"

		# Customize the amount of memory on the VM:
		v.memory = "4096"

		# Customize the amount of CPU on the VM:
		v.cpus = 2

		# the VM is modified to have a host CPU execution cap of 50%, meaning that no matter how much CPU is used in the VM, no more than 50% would be used on your owhost machine
		v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
	end
end
```  


  
---
SSH access (for images aagrebeshkov):
	
  root / root
	
  vagrant / vagrant
