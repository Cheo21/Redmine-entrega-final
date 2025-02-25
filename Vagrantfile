Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.network "public_network"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 4096
  end
  config.vm.provision "shell", inline: <<-SHELL
    set -ex
    apt-get update && apt-get upgrade -y
    apt install -y nginx build-essential libmysqlclient-dev mysql-server \
    libonig-dev tk-dev libgdbm-dev libyaml-dev libffi-dev \
    libssl-dev libreadline6-dev zlib1g-dev rustc
     
  
    cp -r /vagrant/redmine-5.0.11 /opt
    chown -R vagrant:vagrant /opt/redmine-5.0.11


    su - vagrant -c "git config --global advice.detachedHead false"
    su - vagrant -c "[ -d ~/.asdf ] || git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.15.0"
    su - vagrant -c "grep -q asdf.sh ~/.bashrc || echo -e '. \"\\$HOME/.asdf/asdf.sh\"\n. \"\\$HOME/.asdf/completions/asdf.bash\"' >> .bashrc"
   
    su - vagrant -c ". ~/.asdf/asdf.sh && asdf plugin add ruby"
    su - vagrant -c ". ~/.asdf/asdf.sh && cd /opt/redmine-5.0.11 && asdf install ruby 3.2.1 && asdf local ruby 3.2.1"
  SHELL
end
