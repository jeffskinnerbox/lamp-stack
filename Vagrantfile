# -*- mode: ruby -*-
# vi: set ft=ruby :

# Maintainer:   jeffskinnerbox@yahoo.com / www.jeffskinnerbox.me
# Version:      0.0.1


# Vagrantfile API/syntax version.  The "2" is the Vagrant configuration version.
VAGRANTFILE_API_VERSION = "2"

# All Vagrant configuration is done below.
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # Every Vagrant development environment requires a box
  # You can search for boxes at https://vagrantcloud.com/search
  #config.vm.box = "ubuntu/xenial64"          # ubuntu 16.04 guest vm
  #config.vm.box = "ubuntu/bionic64"          # ubuntu 18.04 guest vm
  #config.vm.box = "ubuntu/eoan64"            # ubuntu 19.10 guest vm
  #config.vm.box = "ubuntu/focal64"           # ubuntu 20.04 guest vm
  config.vm.box = "ubuntu-headless"          # ubuntu 20.04 guest vm, customized with my tools
  config.vm.hostname ="lamp.vm"              # set hostname

  # set the disk size to be allocated
  config.disksize.size = "15GB"

  # X forwarding support, using port 2222
  config.ssh.forward_agent = true        # if true, agent forwarding over SSH connections is enabled
  config.ssh.forward_x11 = true          # if true, X11 forwarding over SSH connections is enabled

  # set auto_update to false, if you do NOT want to check the correct
  # guest additions version when booting this machine
  if Vagrant.has_plugin?("vagrant-vbguest") then
        config.vbguest.auto_update = false
  end

  # time in seconds that vagrant will wait for the machine to boot and be accessible, default 300 sec
  config.vm.boot_timeout = 300

  # If set to 'false', boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = true

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  #config.vm.network "forwarded_port", guest: 80, host: 8080       # port for web server

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  #config.vm.network "forwarded_port", guest: 8888, host: 8888, host_ip: "127.0.0.1" # port for jupyter notebook
  #config.vm.network "forwarded_port", guest: 1880, host: 1880, host_ip: "127.0.0.1" # port for node-red
  #config.vm.network "forwarded_port", guest: 8086, host: 8086, host_ip: "127.0.0.1" # port for influxdb
  #config.vm.network "forwarded_port", guest: 3000, host: 3000, host_ip: "127.0.0.1" # port for grafana
  config.vm.network "forwarded_port", guest: 3306, host: 3306, host_ip: "127.0.0.1"  # port for mysql
  config.vm.network "forwarded_port", guest: 80, host: 8306, host_ip: "127.0.0.1"    # port for apache php phpmyadmin

 # Create a private network, which allows host-only access to the machine
  #config.vm.network "private_network", ip: "192.168.10.222"   # static ip addess
  #config.vm.network "private_network", type: "dhcp"           # DHCP assigned ip address

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on your network.
  $def_net = `ip route | grep -E "^default" | awk '{printf "%s", $5; exit 0}'`   # default network interface
  #config.vm.network "public_network", bridge: "#$def_net", ip: "192.168.1.218"   # static ip addess
  config.vm.network "public_network", bridge: "#$def_net"                        # DHCP assigned ip address

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "shared-data/", "/home/vagrant/shared-data", create: true, type: "virtualbox", :mount_options => ["dmode=777", "fmode=666"]

  # customize the configuration of VM - see https://www.virtualbox.org/manual/ch08.html
  # virtual hardware configuration
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false                                                           # true = GUI, false = headless
    vb.customize ["modifyvm", :id, "--cpus", 2]                              # number of cpu used
    vb.customize ["modifyvm", :id, "--memory", 4096]                         # memory used

    # enable USB and add a filter based on the desired device manufacturer / product
    #vb.customize ["modifyvm", :id, "--usb", "on"]           # usb 1.1 (CHCI) controller
    #vb.customize ["modifyvm", :id, "--usbehci", "on"]       # usb 2.0 (EHCI) controller
    #vb.customize ["modifyvm", :id, "--usbxhci", "on"]       # usb 3.0 (xHCI) controller
    #vb.customize ['usbfilter', 'add', '0', '--target', :id,
        #'--name', 'QuickCam Orbit/Sphere AF',
        #'--vendorid', '0x046d',
        #'--productid', '0x0994']
    #vb.customize ['usbfilter', 'add', '0', '--target', :id,
        #'--name', 'Adafruit Console Cable',
        #'--vendorid', '0x10c4',
        #'--productid', '0xea60']
  end



  # ----------------------------------------------------------------------------
  # Linux System Environment
  # ----------------------------------------------------------------------------

  # --------------------- update linux packages (as root) ----------------------
  config.vm.provision "shell", name: "update linux packages (as root)", run: "always",
     inline: "apt-get -y update && apt-get -y dist-upgrade"



  # ----------------------------------------------------------------------------
  # LAMP Environment
  # ----------------------------------------------------------------------------

  # ------------------ create your apache environment (as root) ----------------
  config.vm.provision "shell", name: "create your apache environment (as root)", run: "once", inline: <<-SHELL
    # install the latest version of apache
    apt-get -y install apache2 apache2-utils
    
    # enable apache2 to automatically start at boot time
    systemctl enable apache2

    # enable ufw without prompt
    ufw --force enable

    # only allow traffic on port 80 and 22
    ufw allow in "Apache"
    ufw allow in "OPENSSH"
SHELL

  # -------------------- create your php environment (as root) -----------------
  config.vm.provision "shell", name: "create your php environment (as root)", run: "once", inline: <<-SHELL
    # install php and some common PHP modules
    apt-get -y install php libapache2-mod-php php-mysql php-common php-cli php-common php-json php-opcache php-readline

    # get the php version number
    PHP_VERSION=$(php --version | head -n1 | awk '{ print $2 }' | awk -F . '{ print $1 "." $2 }')
    
    # enable the apache php module (using version number from above)
    a2enmod php$PHP_VERSION

    # disable the Apache PHP module
    a2dismod php$PHP_VERSION

    # install php-fpm
    apt install php$PHP_VERSION-fpm

    # enable proxy_fcgi and setenvif module
    a2enmod proxy_fcgi setenvif

    # enable the /etc/apache2/conf-available/php7.4-fpm.conf file
    a2enconf php$PHP_VERSION-fpm

    # restart apache for the changes to take effect
    systemctl restart apache2
SHELL

  # ------------------- create your mysql environment (as root) ----------------
  config.vm.provision "shell", name: "create your mysql environment (as root)", run: "once", inline: <<-SHELL
    # set env variables
    # use single quotes instead of double quotes to make it work with special-characters
    DBHOST='localhost'
    DBNAME='mysql-db-name'
    DBUSER='mysql-user'
    DBPASSWD='mysql-password'

    # when installing mysql, this will give password to installer
    debconf-set-selections <<< "mysql-server mysql-server/root_password password $DBPASSWD"
    debconf-set-selections <<< "mysql-server mysql-server/root_password_again password $DBPASSWD"

    # acquire and install mysql and other required software
    apt-get -y install mysql-server libmysqlclient-dev build-essential npm

    # enable mmysql to automatically start at boot time
    systemctl enable mysql
SHELL

  config.vm.provision "shell", name: "create mysql configure file to avoid inputing password (as vagrant)", run: "once", inline: <<-SHELL
    # become the vagrant user
    su --login vagrant --shell /bin/bash <<'EOF'

    # setup mysql configure file to avoid inputing password
    DBPASSWD='mysql-password'
    echo -e "[client]\nuser = root\npassword = $DBPASSWD" >> ~/.my.cnf
EOF
SHELL

  # ---------------- create your phpmyadmin environment (as root) --------------
  config.vm.provision "shell", name: "create your phpmyadmin environment (as root)", run: "once", inline: <<-SHELL
    # set env variables
    # use single quotes instead of double quotes to make it work with special-characters
    DBPASSWD='mysql-password'
    PHPPASSWD='php-password'

    # when installing phpmyadmin, this will give password and other data to installer
    debconf-set-selections <<< "phpmyadmin phpmyadmin/dbconfig-install boolean true"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/app-password-confirm password $PHPPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/admin-pass password $DBPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/mysql/app-pass password $DBPASSWD"
    debconf-set-selections <<< "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2"

    # install phpmyadmin
    apt-get -y install phpmyadmin php-mbstring php-zip php-gd php-json php-curl

    # enable the mbstring php extensio
    phpenmod mbstring

    # restart apache
    systemctl restart apache2
SHELL



  # ----------------------------------------------------------------------------
  # Restart Linux OS
  # ----------------------------------------------------------------------------

  # ------------- reboot to make sure everything is set (as root) --------------
  config.vm.provision "shell", name: "reboot to make sure everything is setup (as root)", run: "once", inline: <<-SHELL
    shutdown -r now
SHELL

end
