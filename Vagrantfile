Vagrant.configure("2") do |config|
   # Configuration commune
   config.vm.provider "virtualbox" do |vb|
      vb.cpus = 2
      vb.memory = 2048
      vb.linked_clone = true
   end

   # Configuration du serveur AD
   config.vm.define "ad" do |ad|
      ad.vm.box = "gusztavvargadr/windows-server-2019-standard"
      ad.vm.hostname = "ad"
      ad.vm.network "private_network", ip: "192.168.56.10"
      ad.vm.provider "virtualbox" do |vb|
         vb.gui = true
      end
      
      # Configuration WinRM pour AD
      ad.vm.communicator = "winrm"
      ad.winrm.host = "127.0.0.1"
      ad.winrm.transport = "plaintext"
      ad.winrm.basic_auth_only = true
      ad.winrm.username = "vagrant"
      ad.winrm.password = "vagrant"

      # Script PowerShell pour WinRM
      ad.vm.provision "shell", inline: <<-SHELL
         Set-ExecutionPolicy Bypass -Scope Process -Force
         winrm quickconfig -q
         winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="2048"}'
         winrm set winrm/config '@{MaxTimeoutms="1800000"}'
         winrm set winrm/config/service '@{AllowUnencrypted="true"}'
         winrm set winrm/config/service/auth '@{Basic="true"}'
         Start-Service WinRM
         Set-Service WinRM -StartupType Automatic
         netsh advfirewall firewall add rule name="WinRM HTTP" dir=in localport=5985 protocol=TCP action=allow
      SHELL

      # Ansible pour AD
      ad.vm.provision "ansible" do |ansible|
         ansible.playbook = "playbook-ansible-ad.yml"
         ansible.compatibility_mode = "2.0"
         ansible.host_vars = {
            "ad" => {
               "ansible_user" => "vagrant",
               "ansible_password" => "vagrant",
               "ansible_connection" => "winrm",
               "ansible_winrm_transport" => "basic",
               "ansible_port" => 55985,
               "ansible_host" => "127.0.0.1",
               "ansible_winrm_scheme" => "http",
               "ansible_winrm_server_cert_validation" => "ignore"
            }
         }
      end
   end

   # Configuration du serveur Ubuntu
   config.vm.define "ubuntu" do |ubuntu|
      ubuntu.vm.box = "ubuntu/jammy64"
      ubuntu.vm.hostname = "mail-server"
      
      # Configuration réseau pour Ubuntu
      ubuntu.vm.network "private_network", ip: "192.168.56.20"

      # Ansible provisioning for mail server
      ubuntu.vm.provision "ansible" do |ansible|
         ansible.playbook = "playbook-ansible-mail.yml"
         ansible.compatibility_mode = "2.0"
         ansible.host_vars = {
            "ubuntu" => {
               "ansible_user" => "vagrant",
               "ansible_password" => "vagrant",
               "ansible_port" => "2200",
               "ansible_host" => "127.0.0.1",
               "ansible_become" => true
            }
         }
      end
   end

      # Configuration du Clien Ubuntu
   config.vm.define "client-linux" do |ubuntu|
      ubuntu.vm.box = "fasmat/ubuntu2204-desktop"
      ubuntu.vm.hostname = "client-linux"

      # Configuration réseau pour Ubuntu
      ubuntu.vm.network "private_network", ip: "192.168.56.30"

      # Ansible provisioning for mail server
      ubuntu.vm.provision "ansible" do |ansible|
         ansible.playbook = "playbook-ansible-client-linux.yml"
         ansible.compatibility_mode = "2.0"
         ansible.host_vars = {
            "client-linux" => {
               "ansible_user" => "vagrant",
               "ansible_password" => "vagrant",
	           "ansible_port" => "2201",
               "ansible_host" => "127.0.0.1",
               "ansible_become" => true
            }
         }
      end
   end

    # Configuration du Clien WIndows
   config.vm.define "win10-client" do |win|
      win.vm.box = "StefanScherer/windows_10"
      win.vm.hostname = "win10-client"
      win.vm.network "private_network",
         ip: "192.168.56.40",
         adapter: 2,
         netmask: "255.255.255.0",
         auto_config: true

      win.vm.provider "virtualbox" do |vb|
         vb.gui = true
         vb.memory = "4096"
         vb.cpus = 2
      end

      # Configuration WinRM
      win.vm.communicator = "winrm"
      win.winrm.host = "127.0.0.1"
      win.winrm.transport = "plaintext"
      win.winrm.basic_auth_only = true
      win.winrm.username = "vagrant"
      win.winrm.password = "vagrant"

      # Script PowerShell pour WinRM et configuration réseau
      win.vm.provision "shell", inline: <<-SHELL
         # Set network to Private
         Set-NetConnectionProfile -NetworkCategory Private

         # Configuration WinRM
         Set-ExecutionPolicy Bypass -Scope Process -Force
         winrm quickconfig -q -force
         winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="2048"}'
         winrm set winrm/config '@{MaxTimeoutms="1800000"}'
         winrm set winrm/config/service '@{AllowUnencrypted="true"}'
         winrm set winrm/config/service/auth '@{Basic="true"}'
         Start-Service WinRM
         Set-Service WinRM -StartupType Automatic
         netsh advfirewall firewall add rule name="WinRM HTTP" dir=in localport=5985 protocol=TCP action=allow

         # Configure network adapter
         $adapter = Get-NetAdapter | Where-Object {$_.Status -eq "Up" -and $_.InterfaceDescription -like "*VirtualBox*"} | Select-Object -Last 1
         if ($adapter) {
            Remove-NetIPAddress -InterfaceIndex $adapter.ifIndex -Confirm:$false -ErrorAction SilentlyContinue
            New-NetIPAddress -InterfaceIndex $adapter.ifIndex -IPAddress "192.168.56.40" -PrefixLength 24
            Set-DnsClientServerAddress -InterfaceIndex $adapter.ifIndex -ServerAddresses "192.168.56.10"
         }
      SHELL

      # Ansible provisioning
      win.vm.provision "ansible" do |ansible|
         ansible.playbook = "playbook-ansible-win10.yml"
         ansible.compatibility_mode = "2.0"
         ansible.host_vars = {
            "win10-client" => {
               "ansible_user" => "vagrant",
               "ansible_password" => "vagrant",
               "ansible_connection" => "winrm",
               "ansible_winrm_transport" => "basic",
               "ansible_port" => 2202,  # Fixed port instead of dynamic
               "ansible_host" => "127.0.0.1",
               "ansible_winrm_scheme" => "http",
               "ansible_winrm_server_cert_validation" => "ignore"
            }
         }
      end
   end

end
