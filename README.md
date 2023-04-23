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
<br>

Выключить виртуальную машину штатным образом:
```bash
vagrant halt
```
<br>

Перезапустить ВМ:
```bash
vagrant reload
```
<br>

Если наша машина уже была запущена, но нужно применить обновление из Vagrantfile, то нужно выполнить команду:
```bash
vagrant reload --provision
```
<br>

Подключиться к SSH ВМ:
```bash
vagrant ssh
```
<br>

Узнать статус машины:
```bash
vagrant status
```
<br>

Удалить ВМ:
```bash
vagrant destroy -f
```
<br>

Запуск с логированием debug:
```bash
vagrant up --debug vagrant.log 2>&1
```
<br>

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
<br>

  
---
SSH access (for images aagrebeshkov):
	
  root / root
	
  vagrant / vagrant
<br>

---

### Конфигурирование нескольких машин по отдельности
```bash
Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: <<-SHELL
    apt-get install -y net-tools
    echo "192.168.50.1  node01" >> /etc/hosts
    echo "192.168.50.2  node02" >> /etc/hosts
  SHELL

  config.vm.define "node01" do |n1|
    n1.vm.box = "bento/ubuntu-20.04"
    n1.vm.hostname = "node01"
    n1.vm.provider "virtualbox" do |v|
      v.name = "node01"
      v.memory = "2048"
      v.cpus = 2
    end
    n1.vm.network "private_network", ip: "192.168.50.1"

    #n1.vm.provision "shell", inline: <<-SHELL
    #  apt-get install -y apache2
    #SHELL
  end

  config.vm.define "node02" do |n2|
    n2.vm.box = "bento/ubuntu-20.04"
    n2.vm.hostname = "node02"
    n2.vm.provider "virtualbox" do |v|
      v.name = "node02"
      v.memory = "2048"
      v.cpus = 2
    end
    n2.vm.network "private_network", ip: "192.168.50.2"
  end
end
```
<br>


### Конфигурирование нескольких машин по отдельности с переменными
```bash
$vm_1 = "node01"
$ip_1 = "192.168.50.1"
$cpu_1 = 1
$mem_1 = "1024"
$box_1 = "bento/ubuntu-20.04"

$vm_2 = "node02"
$ip_2 = "192.168.50.2"
$cpu_2 = 2
$mem_2 = "2048"
$box_2 = "bento/ubuntu-20.04"

Vagrant.configure("2") do |config|
  config.vm.provision "shell" do |s|
    s.inline = "echo $1 $2 >> /etc/hosts; echo $3 $4 >> /etc/hosts; apt-get install -y net-tools"
    s.args   = $ip_1, $vm_1, $ip_2, $vm_2
  end

  config.vm.define $vm_1 do |n1|
    n1.vm.box = $box_1
    n1.vm.hostname = $vm_1
    n1.vm.provider "virtualbox" do |v|
      v.name = $vm_1
      v.memory = $mem_1
      v.cpus = $cpu_1
    end
    n1.vm.network "private_network", ip: $ip_1
    #n1.vm.provision "shell", inline: <<-SHELL
    #  apt-get install -y apache2
    #SHELL
  end

  config.vm.define $vm_2 do |n2|
    n2.vm.box = $box_2
    n2.vm.hostname = $vm_2
    n2.vm.provider "virtualbox" do |v|
      v.name = $vm_2
      v.memory = $mem_2
      v.cpus = $cpu_2
    end
    n2.vm.network "private_network", ip: $ip_2
  end
end
```
<br>


### Конфигурирование нескольких машин в цикле
```bash
$number_vm = 2

Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: <<-SHELL
    apt-get install -y net-tools
  SHELL

  (1..$number_vm).each do |i|
    config.vm.define "node0#{i}" do |node|
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.hostname = "node0#{i}"
      node.vm.provider "virtualbox" do |v|
        v.name = "node0#{i}"
        v.memory = "2048"
        v.cpus = 2
      end
      node.vm.network "private_network", ip: "192.168.50.#{i}"
    end
  end
end
```
<br>

или

```bash
boxes = {
  'node01' => '1',
  'node02' => '2'           ### попробовать убрать запятую
}

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-20.04"
  config.vm.box = "bento/ubuntu-20.04"

  boxes.each do |name_vm, i|
    config.vm.define name_vm do |node|
      node.vm.hostname = name_vm
      node.vm.provider "virtualbox" do |v|
        v.name = name_vm
        v.memory = "2048"
        v.cpus = 2
      end
      node.vm.network "private_network", ip: "192.168.50.#{i}"
      node.vm.provision "shell", inline: <<-SHELL
        apt-get install -y net-tools
        #echo "192.168.50.1  node01" >> /etc/hosts
        #echo "192.168.50.2  node02" >> /etc/hosts
      SHELL
    end
  end
end
```