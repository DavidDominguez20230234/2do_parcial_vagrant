# Vagrantfile para configurar tres máquinas virtuales:
# Dos servidores Apache y un servidor Nginx para balanceo de carga

Vagrant.configure("2") do |config|
  # Configuraciones generales para todas las máquinas
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = 512
  end

  # Máquina 1: Servidor Apache 1
  config.vm.define "web1" do |web1|
    web1.vm.box = "ubuntu/bionic64"
    web1.vm.hostname = "web1"
    web1.vm.network "private_network", ip: "192.168.33.10"
    web1.vm.synced_folder ".", "/var/www/html", type: "rsync" # Carpeta compartida
    
    web1.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2
      echo "Hello from Web Server 1" > /var/www/html/index.html
      systemctl restart apache2
    SHELL
  end

  # Máquina 2: Servidor Apache 2
  config.vm.define "web2" do |web2|
    web2.vm.box = "ubuntu/bionic64"
    web2.vm.hostname = "web2"
    web2.vm.network "private_network", ip: "192.168.33.11"
    web2.vm.synced_folder ".", "/var/www/html", type: "rsync" # Carpeta compartida
    
    web2.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2
      echo "Hello from Web Server 2" > /var/www/html/index.html
      systemctl restart apache2
    SHELL
  end

  # Máquina 3: Servidor Nginx para balanceo de carga
  config.vm.define "loadbalancer" do |lb|
    lb.vm.box = "ubuntu/bionic64"
    lb.vm.hostname = "loadbalancer"
    lb.vm.network "private_network", ip: "192.168.33.12"
    
    lb.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y nginx

      # Configuración de Nginx para balanceo de carga
      cat > /etc/nginx/sites-available/default <<EOF
      upstream web_servers {
          server 192.168.33.10;
          server 192.168.33.11;
      }

      server {
          listen 80;

          location / {
              proxy_pass http://web_servers;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Proto $scheme;
          }
      }
      EOF

      systemctl restart nginx
    SHELL
  end
end
