Vagrant.configure("2") do |config|
  config.vm.define "edosign3" do |edosign|
    edosign.vm.box = "bento/ubuntu-22.04"
    edosign.vm.hostname = "edosign3-ubuntu"

    # 🔹 ОДИН спільний namespace: edosign.vm.*
    edosign.vm.network "forwarded_port", guest: 7090, host: 7090, auto_correct: true
    edosign.vm.network "forwarded_port", guest: 7275, host: 7275, auto_correct: true

    edosign.vm.provision "shell", inline: <<-SHELL
      echo "=== Оновлення системи ==="
      apt-get update -y
      apt-get install -y wget git apt-transport-https ca-certificates lsb-release

      echo "=== Встановлення .NET SDK 9.0 ==="
      wget https://packages.microsoft.com/config/ubuntu/22.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
      dpkg -i packages-microsoft-prod.deb
      apt-get update -y
      apt-get install -y dotnet-sdk-9.0

      echo "=== Клонування репозиторію Edo-Sign3 ==="
      su - vagrant -c "rm -rf ~/Edo-Sign3 && git clone https://github.com/onyshchenkodmytro/Edo-Sign3 ~/Edo-Sign3"

      echo "=== Виправлення прав доступу ==="
      chmod -R 777 /home/vagrant/Edo-Sign3

      echo "=== Налаштування NuGet джерел ==="
      mkdir -p /home/vagrant/.nuget/NuGet
      cat > /home/vagrant/.nuget/NuGet/NuGet.Config <<'CFG'
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="baget" value="http://192.168.56.10:5555/v3/index.json" />
  </packageSources>
  <config>
    <add key="allowInsecureConnections" value="true" />
  </config>
</configuration>
CFG

      echo "=== Побудова і запуск EdoSign.Lab-3 ==="
      cd "/home/vagrant/Edo-Sign3/EdoSign.Lab-3"
      dotnet restore
      dotnet build -c Release
      dotnet publish -c Release -o /app

      echo "=== Запуск веб-застосунку на http://0.0.0.0:7275 ==="
      nohup dotnet /app/EdoSign.Lab-3.dll --urls=http://0.0.0.0:7275 > /var/log/edosign3.log 2>&1 &
      sleep 5
      echo " Веб-застосунок запущено! Відкрий у браузері: http://localhost:7275"
    SHELL
  end
end
