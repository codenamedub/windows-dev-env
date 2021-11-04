Vagrant.configure("2") do |config|
    # Specify the base image
    config.vm.box = "ubuntu/focal64"

    # Forward ports from guest to host
    config.vm.network "forwarded_port", guest: 3000, host: 3000
    config.vm.network "forwarded_port", guest: 3001, host: 3001

    config.vm.provider "virtualbox" do |vb|
        # Increase vm resources
        vb.memory = 4096
        vb.cpus = 4
    end

    # Copy host gitconfig to guest
    config.vm.provision "file", source: "~/.gitconfig", destination: ".gitconfig"

    # Execute as root
    config.vm.provision "shell", inline: <<-SHELL
        # Upgrade
        apt-get update && apt-get ugrade -y

        # Install Docker dependencies
        apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
        add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(. /etc/os-release; echo "$UBUNTU_CODENAME") stable"
        apt-get update
        apt-get -y install docker-ce

        # Install other dependencies
        sudo apt install python3-pip unzip -y
        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && unzip awscliv2.zip
        sudo ./aws/install

        # Cleanup
        apt-get autoremove && apt-get clean

        # Set the default editor
        update-alternatives --set editor /usr/bin/vim.basic

        # Increase file watchers
        if ! grep -q '^fs.inotify.max_user_watches' /etc/sysctl.conf; then
            echo "fs.inotify.max_user_watches=524288" >> /etc/sysctl.conf
            sysctl -p
        fi
    SHELL

    # Execute as non-privileged user
    config.vm.provision "shell", privileged: false, inline: <<-SHELL
      if [ ! -d "$HOME/.nvm" ]; then
        # Update github.com certs
        sudo bash -c "openssl s_client -showcerts -servername github.com -connect github.com:443 </dev/null 2>/dev/null | sed -n -e '/BEGIN\ CERTIFICATE/,/END\ CERTIFICATE/,/END\ CERTIFICATE/ p' > /usr/local/certificates/cert-github.com.crt"
        sudo update-ca-certificates

        # Install nvm
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
        export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
        [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

        # Install node
        nvm install node

        # Install npm
        npm install -g npm
      fi
    SHELL
end
