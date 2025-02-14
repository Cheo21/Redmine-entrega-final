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
     
  
    su - vagrant -c "git config --global advice.detachedHead false"
    su - vagrant -c "[ -d ~/.asdf ] || git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.15.0"
        su - vagrant -c "grep -q asdf.sh ~/.bashrc || echo -e '. \"\\$HOME/.asdf/asdf.sh\"\n. \"\\$HOME/.asdf/completions/asdf.bash\"' >> .bashrc"
    su - vagrant -c ". ~/.asdf/asdf.sh && for i in direnv java ruby python nodejs; do echo Install \\$i plugin && asdf plugin add \\$i; done"
    su - vagrant -c ". ~/.asdf/asdf.sh && echo Install latest direnv plugin && asdf install direnv latest && asdf global direnv latest && asdf direnv setup --shell bash --version latest"
   su - vagrant -c ". ~/.asdf/asdf.sh && echo Allow direnv work directories && cd /vagrant && direnv allow && asdf direnv install"
  SHELL
end
