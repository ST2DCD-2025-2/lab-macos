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
    gitlab.vm.network "private_network", ip: ENV['GITLAB_IP']
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
      curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
      sudo apt-get install -y gitlab-runner
      echo '------------------------------'
      sudo cat /etc/gitlab/initial_root_password | grep Password:
      echo '------------------------------'
    SHELL
  end

  # GitLab Runner VM
  config.vm.define "runner" do |runner|
    runner.vm.hostname="Gitlab.runner"
    runner.vm.network "private_network", ENV['GITLAB_RUNNER_IP']
    runner.vm.provider :vmware_fusion do |runner_ressources|
      runner_ressources.cpus = 2
      runner_ressources.memory = "2048"
    end

    # Shell automatisation script
    runner.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update && sudo apt-get upgrade -y
        sudo apt-get install -y curl
        curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
        sudo apt-get install -y gitlab-runner

        sudo gitlab-runner register --non-interactive \
          --url "http://#{ENV['GITLAB_VM_IP']}" \
          --registration-token "#{ENV['GITLAB_RUNNER_TOKEN']}" \
          --executor "#{ENV['GITLAB_RUNNER_EXECUTOR']}" \
          --description "#{ENV['GITLAB_RUNNER_NAME']}"
    SHELL
  end
end
