MACHINES = {
:inetRouter => {
:box_name => "centos7",
:vm_name => "inetRouter",
#:public => {:ip => '10.10.10.1', :adapter => 1},
:net => [
{ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252",
virtualbox__intnet: "router-net"},
{ip: '192.168.50.10', adapter: 8},
]
},
:centralRouter => {
:box_name => "centos7",
:vm_name => "centralRouter",
:net => [
{ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252",
virtualbox__intnet: "router-net"},
{ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240",
virtualbox__intnet: "dir-net"},
{ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240",
virtualbox__intnet: "local"},
{ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192",
virtualbox__intnet: "mgt-net"},
{ip: '192.168.255.9', adapter: 6, netmask: "255.255.255.252",
virtualbox__intnet: "office1-central"},
{ip: '192.168.255.5', adapter: 7, netmask: "255.255.255.252",
virtualbox__intnet: "office2-central"},
{ip: '192.168.50.11', adapter: 8},
]
},
:testClient1 => {
:box_name => "ubuntu/focal64",
:vm_name => "testClient1",
:net => [
{adapter: 2, virtualbox__intnet: "local"},
{ip: '192.168.255.9', adapter: 8, netmask: "255.255.255.252"},
]
},
:testClient2 => {
:box_name => "ubuntu/focal64",
:vm_name => "testClient2",
:net => [
{adapter: 2, virtualbox__intnet: "local"},
{ip: '192.168.255.13', adapter: 8, netmask: "255.255.255.252"},
]
},
:testServer1 => {
:box_name => "ubuntu/focal64",
:vm_name => "testServer1",
:net => [
{adapter: 2, virtualbox__intnet: "local"},
{ip: '192.168.255.17', adapter: 8, netmask: "255.255.255.252"},
]
},
:testServer2 => {
:box_name => "ubuntu/focal64",
:vm_name => "testServer2",
:net => [
{adapter: 2, virtualbox__intnet: "local"},
{ip: '192.168.255.21', adapter: 8, netmask: "255.255.255.252"},
]
}
}
Vagrant.configure("2") do |config|
    MACHINES.each do |boxname, boxconfig|
        config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
	  if boxconfig[:vm_name] == "testServer2"
		box.vm.provision "ansible" do |ansible|
		ansible.playbook = "ansible/main.yaml"
		ansible.host_key_checking = "false"
		ansible.limit = "all"
		end
	  end
        boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", ipconf
        end
        if boxconfig.key?(:public)
        box.vm.network "public_network", boxconfig[:public]
        end
        box.vm.provision "shell", inline: <<-SHELL
        mkdir -p ~root/.ssh
        cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        case boxname.to_s
        when "inetRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
        sysctl net.ipv4.conf.all.forwarding=1
        iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
        SHELL
        when "centralRouter"
        box.vm.provision "shell", run: "always", inline: <<-SHELL
        sysctl net.ipv4.conf.all.forwarding=1
        echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
        echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
        systemctl restart network
        SHELL
        end
        end
        end
	end
