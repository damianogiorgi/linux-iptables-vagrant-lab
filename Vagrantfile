# -*- mode: ruby -*-
# vi: set ft=ruby :

# IPTables Networking Lab
# This Vagrantfile creates a network topology for learning Linux networking and iptables:
# - Outside Network (10.0.10.0/24): NAT network simulating internet
# - DMZ Network (192.168.100.0/24): Exposed services network  
# - Internal LAN (192.168.200.0/24): Internal clients network
# - Firewall VM: Connected to all networks, acts as router/gateway

Vagrant.configure("2") do |config|
  # Common configuration for all VMs
  config.vm.box = "bento/ubuntu-24.04"
  
  # ======================
  # FIREWALL VM
  # ======================
  config.vm.define "firewall" do |fw|
    fw.vm.hostname = "firewall"
    
    # Outside network interface (NAT - represents internet connection)
    fw.vm.network "private_network", 
      ip: "10.0.10.254", 
      netmask: "255.255.255.0",
      virtualbox__intnet: "outside"
    
    # DMZ network interface
    fw.vm.network "private_network", 
      ip: "192.168.100.254", 
      netmask: "255.255.255.0",
      virtualbox__intnet: "dmz"
    
    # Internal LAN interface
    fw.vm.network "private_network", 
      ip: "192.168.200.254", 
      netmask: "255.255.255.0",
      virtualbox__intnet: "internal-lan"
    
    fw.vm.provider "virtualbox" do |vb|
      vb.name = "Firewall-VM"
      vb.memory = "1024"
      vb.cpus = 2
    end
    
    fw.vm.provision "shell", inline: <<-SHELL
      # Update system
      apt-get update
      apt-get install -y curl wget net-tools tcpdump
      
      # Configure /etc/hosts for all lab VMs
      cat >> /etc/hosts << 'EOF'

# IPTables Lab VMs
10.0.10.254     firewall firewall.lab
10.0.10.10      outside-server outside.lab
192.168.100.254 firewall-dmz
192.168.100.10  dmz-server dmz.lab
192.168.200.254 firewall-internal
192.168.200.10  internal-client internal.lab
EOF
      
      # Enable IP forwarding
    #   echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
    #   sysctl -p
      
      # Clear existing iptables rules
      iptables -F
      iptables -X
      iptables -t nat -F
      iptables -t nat -X
      
      # Set default policies (will be configured for lab exercises)
      iptables -P INPUT ACCEPT
      iptables -P FORWARD ACCEPT
      iptables -P OUTPUT ACCEPT
      
      # Basic NAT rule for outside network (internet simulation)
    #   iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o enp0s8 -j MASQUERADE
    #   iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -o enp0s8 -j MASQUERADE

      
      echo "=== Firewall VM Configuration ==="
      echo "Outside interface (enp0s8): 10.0.10.254/24"
      echo "DMZ interface (enp0s9): 192.168.100.254/24"
      echo "Internal interface (enp0s10): 192.168.200.254/24"
      echo "IP forwarding: enabled"
      ip addr show
    SHELL
  end

  # ======================
  # OUTSIDE NETWORK VM
  # ======================
  config.vm.define "outside-server" do |outside|
    outside.vm.hostname = "outside-server"
    
    # Connect to outside network
    outside.vm.network "private_network", 
      ip: "10.0.10.10", 
      netmask: "255.255.255.0",
      virtualbox__intnet: "outside"
    
    outside.vm.provider "virtualbox" do |vb|
      vb.name = "Outside-Server-VM"
      vb.memory = "1024"
      vb.cpus = 1
    end
    
    outside.vm.provision "shell", inline: <<-SHELL
      # Update system
      apt-get update
      apt-get install -y apache2 curl wget net-tools tcpdump
      
      # Configure /etc/hosts for all lab VMs
      cat >> /etc/hosts << 'EOF'

# IPTables Lab VMs
10.0.10.254     firewall firewall.lab
10.0.10.10      outside-server outside.lab
192.168.100.254 firewall-dmz
192.168.100.10  dmz-server dmz.lab
192.168.200.254 firewall-internal
192.168.200.10  internal-client internal.lab
EOF
      
      # Configure default route through firewall
    #   ip route del default || true
    #   ip route add default via 10.0.10.254
      
      # Configure Apache to simulate external web service
      echo "<h1>Outside Server - Simulated Internet Service</h1>" > /var/www/html/index.html
      echo "<p>This server represents an external internet service</p>" >> /var/www/html/index.html
      systemctl enable apache2
      systemctl start apache2
      
      echo "=== Outside Server Configuration ==="
      echo "IP: 10.0.10.10/24"
      echo "Gateway: 10.0.10.254 (firewall)"
      echo "Services: Apache web server on port 80"
      ip addr show
      ip route show
    SHELL
  end

  # ======================
  # DMZ VM
  # ======================
  config.vm.define "dmz-server" do |dmz|
    dmz.vm.hostname = "dmz-server"
    
    # Connect to DMZ network
    dmz.vm.network "private_network", 
      ip: "192.168.100.10", 
      netmask: "255.255.255.0",
      virtualbox__intnet: "dmz"
    
    dmz.vm.provider "virtualbox" do |vb|
      vb.name = "DMZ-Server-VM"
      vb.memory = "1024"
      vb.cpus = 1
    end
    
    dmz.vm.provision "shell", inline: <<-SHELL
      # Update system
      apt-get update
      apt-get install -y apache2 curl wget vim net-tools tcpdump
      
      # Configure /etc/hosts for all lab VMs
      cat >> /etc/hosts << 'EOF'

# IPTables Lab VMs
10.0.10.254     firewall firewall.lab
10.0.10.10      outside-server outside.lab
192.168.100.254 firewall-dmz
192.168.100.10  dmz-server dmz.lab
192.168.200.254 firewall-internal
192.168.200.10  internal-client internal.lab
EOF
      
      # Configure default route through firewall
      ip route del default || true
      ip route add default via 192.168.100.254
      
      # Configure Apache for DMZ web service
      echo "<h1>DMZ Server - Public Web Service</h1>" > /var/www/html/index.html
      echo "<p>This server is in the DMZ and provides public services</p>" >> /var/www/html/index.html
      echo "<p>Services: Web (HTTP), SSH</p>" >> /var/www/html/index.html
      systemctl enable apache2
      systemctl start apache2
      
      # Enable SSH
      systemctl enable ssh
      systemctl start ssh
      
      echo "=== DMZ Server Configuration ==="
      echo "IP: 192.168.100.10/24"
      echo "Gateway: 192.168.100.254 (firewall)"
      echo "Services: Apache (port 80), SSH (port 22)"
      ip addr show
      ip route show
    SHELL
  end

  # ======================
  # INTERNAL LAN VM
  # ======================
  config.vm.define "internal-client" do |internal|
    internal.vm.hostname = "internal-client"
    
    # Connect to internal LAN
    internal.vm.network "private_network", 
      ip: "192.168.200.10", 
      netmask: "255.255.255.0",
      virtualbox__intnet: "internal-lan"
    
    internal.vm.provider "virtualbox" do |vb|
      vb.name = "Internal-Client-VM"
      vb.memory = "1024"
      vb.cpus = 1
    end
    
    internal.vm.provision "shell", inline: <<-SHELL
      # Update system
      apt-get update
      apt-get install -y curl wget vim net-tools
      
      # Configure /etc/hosts for all lab VMs
      cat >> /etc/hosts << 'EOF'

# IPTables Lab VMs
10.0.10.254     firewall firewall.lab
10.0.10.10      outside-server outside.lab
192.168.100.254 firewall-dmz
192.168.100.10  dmz-server dmz.lab
192.168.200.254 firewall-internal
192.168.200.10  internal-client internal.lab
EOF
      
      # Configure default route through firewall
      ip route del default || true
      ip route add default via 192.168.200.254
      
      # Create a simple test script for network connectivity
      cat > /home/vagrant/test-connectivity.sh << 'EOF'
#!/bin/bash
echo "=== Network Connectivity Test ==="
echo "Testing connectivity from Internal LAN..."
echo ""
echo "1. Testing DMZ server (192.168.100.10):"
curl -s --connect-timeout 3 http://192.168.100.10 | head -1 || echo "Connection failed"
echo ""
echo "2. Testing Outside server (10.0.10.10):"
curl -s --connect-timeout 3 http://10.0.10.10 | head -1 || echo "Connection failed"
echo ""
echo "3. Current routing table:"
ip route show
EOF
      chmod +x /home/vagrant/test-connectivity.sh
      chown vagrant:vagrant /home/vagrant/test-connectivity.sh
      
      echo "=== Internal Client Configuration ==="
      echo "IP: 192.168.200.10/24"
      echo "Gateway: 192.168.200.254 (firewall)"
      echo "Test script: /home/vagrant/test-connectivity.sh"
      ip addr show
      ip route show
    SHELL
  end
  
  # ======================
  # VAGRANT BOOT ORDER
  # ======================
  # Boot firewall first, then other VMs
  config.vm.define "firewall", primary: true, autostart: true
  config.vm.define "outside-server", autostart: true  
  config.vm.define "dmz-server", autostart: true
  config.vm.define "internal-client", autostart: true
end