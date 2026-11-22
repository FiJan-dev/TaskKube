Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"
  config.vm.hostname = "taskkube"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.network "forwarded_port", guest: 8080, host: 8084, host_ip: "127.0.0.1"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = 3072
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-'SHELL'
    set -eu
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get install -y --no-install-recommends ca-certificates curl git docker.io
    systemctl enable --now docker
    usermod -aG docker vagrant

    if ! command -v minikube >/dev/null; then
      binary=$(mktemp)
      trap 'rm -f "$binary"' EXIT
      arch=$(dpkg --print-architecture)
      curl -fsSL --retry 3 "https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-$arch" -o "$binary"
      install -m 0755 "$binary" /usr/local/bin/minikube
    fi
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-'SHELL'
    set -eu
    cd "$HOME"
    if [ ! -d TaskKube/.git ]; then
      git clone https://github.com/FiJan-dev/TaskKube.git TaskKube
    fi
    cd TaskKube
    # A sessão SSH do Vagrant pode ter sido aberta antes do usermod.
    sg docker -c 'minikube start --driver=docker --nodes=1 --cpus=2 --memory=1944mb'
    sg docker -c 'minikube addons enable ingress'
  SHELL
end
