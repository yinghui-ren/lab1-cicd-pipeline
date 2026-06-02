Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  # Port forwarding: Jenkins UI and deployed app
  config.vm.network "forwarded_port", guest: 8080, host: 8080   # Jenkins
  config.vm.network "forwarded_port", guest: 8081, host: 8081   # PetClinic app
  config.vm.network "forwarded_port", guest: 22,   host: 2222, id: "ssh", auto_correct: true

  config.vm.provider "virtualbox" do |vb|
    vb.name   = "lab1-jenkins"
    vb.memory = "3072"
    vb.cpus   = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    set -e
    export DEBIAN_FRONTEND=noninteractive

    echo "=== Updating system packages ==="
    apt-get update -y
    apt-get upgrade -y

    echo "=== Installing Java 21 (Jenkins >= 2.426 requires Java 21+) ==="
    apt-get install -y openjdk-21-jdk

    echo "=== Installing Maven ==="
    apt-get install -y maven

    echo "=== Installing Git and curl ==="
    apt-get install -y git curl

    echo "=== Installing Jenkins ==="
    curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key \
      | tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
      https://pkg.jenkins.io/debian-stable binary/" \
      | tee /etc/apt/sources.list.d/jenkins.list > /dev/null
    apt-get update -y
    apt-get install -y jenkins
    systemctl enable jenkins
    systemctl start jenkins

    echo "=== Waiting for Jenkins to start ==="
    sleep 30
    echo "Jenkins initial admin password:"
    cat /var/lib/jenkins/secrets/initialAdminPassword || echo "(not yet available)"

    echo "=== Provisioning complete ==="
    echo "Jenkins: http://localhost:8080"
    echo "App:     http://localhost:8081"
  SHELL
end
