# -*- mode: ruby -*-
# vi: set ft=ruby :

# Load .env
if File.exist?('.env')
  File.foreach('.env') do |line|
    key, value = line.strip.split('=', 2)
    ENV[key] = value if key && value
  end
end

Vagrant.configure("2") do |config|
  config.vm.box = "spox/ubuntu-arm"
  config.vm.box_version = "1.0.0"

  # GitLab VM
  config.vm.define "gitlab" do |gitlab|
    gitlab.vm.hostname = "Gitlab"
    gitlab.vm.network "public_network", ip: ENV['GITLAB_IP'], bridge: "en0: Wi-Fi (AirPort)"
    # In private_network mode - Redirection ports
    # gitlab.vm.network "forwarded_port", guest: 80, host: 8080
    # gitlab.vm.network "forwarded_port", guest: 443, host: 4443
    # gitlab.vm.network "forwarded_port", guest: 22, host: 222
    gitlab.vm.provider :vmware_fusion do |gitlab_ressources|
      gitlab_ressources.cpus = 2
      gitlab_ressources.memory = "4096"
    end

    # Shell automatisation script for GitLab
    gitlab.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get upgrade -y
      sudo apt-get install -y curl openssh-server ca-certificates tzdata perl
      curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
      sudo EXTERNAL_URL="http://#{ENV['GITLAB_IP']}" apt-get install -y gitlab-ce
      echo '------------------------------'
      sudo cat /etc/gitlab/initial_root_password | grep Password:
      echo '------------------------------'
    SHELL
  end

  # GitLab Runner VM
  config.vm.define "runner" do |runner|
    runner.vm.hostname="Gitlab.runner"
    runner.vm.network "public_network", ip: ENV['GITLAB_RUNNER_IP'], bridge: "en0: Wi-Fi (AirPort)"
    runner.vm.provider :vmware_fusion do |runner_ressources|
      runner_ressources.cpus = 2
      runner_ressources.memory = "2048"
    end

    # Shell automatisation script
    runner.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update && sudo apt-get upgrade -y
        sudo apt-get install -y curl
        curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
        sudo apt-get install -y gitlab-runner chromium-browser

        # Install Node
        curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
        sudo apt-get install -y nodejs

        sudo gitlab-runner register --non-interactive \
          --url "http://#{ENV['GITLAB_IP']}" \
          --registration-token "#{ENV['GITLAB_RUNNER_TOKEN']}" \
          --executor "#{ENV['GITLAB_RUNNER_EXECUTOR']}" \
          --description "#{ENV['GITLAB_RUNNER_NAME']}"
    SHELL

    runner.vm.synced_folder "./.ssh", "/home/vagrant/.ssh", owner: "vagrant", group: "vagrant"

    runner.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~/.ssh
      cp /home/vagrant/.ssh/id_rsa ~/.ssh/id_rsa
      cp /home/vagrant/.ssh/id_rsa.pub ~/.ssh/id_rsa.pub
      chmod 600 ~/.ssh/id_rsa
      chmod 644 ~/.ssh/id_rsa.pub
      chown vagrant:vagrant ~/.ssh/id_rsa*
    SHELL
  end

  # Deployer
  config.vm.define "deployer" do |deployer|
    deployer.vm.hostname = "Deployer"
    deployer.vm.network "public_network", ip: ENV['DEPLOYER_IP'], bridge: "en0: Wi-Fi (AirPort)"
    deployer.vm.provider :vmware_fusion do |deployer_ressources|
      deployer_ressources.cpus = 2
      deployer_ressources.memory = "2048"
    end

    # Shell automatisation script
    deployer.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update && sudo apt-get upgrade -y
      sudo apt-get install -y nginx
      sudo mkdir -p /var/www/html
      sudo chown -R vagrant:vagrant /var/www/html
    SHELL

    deployer.vm.synced_folder "./.ssh", "/home/vagrant/.ssh", owner: "vagrant", group: "vagrant"

    deployer.vm.provision "shell", inline: <<-SHELL
      mkdir -p ~/.ssh
      cp /home/vagrant/.ssh/id_rsa ~/.ssh/id_rsa
      cp /home/vagrant/.ssh/id_rsa.pub ~/.ssh/id_rsa.pub
      chmod 600 ~/.ssh/id_rsa
      chmod 644 ~/.ssh/id_rsa.pub
      chown vagrant:vagrant ~/.ssh/id_rsa*
    SHELL
  end
end
