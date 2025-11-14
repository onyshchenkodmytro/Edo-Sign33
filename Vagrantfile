Vagrant.configure("2") do |config|
  config.vm.define "edosign33" do |edosign|
    edosign.vm.box = "bento/ubuntu-22.04"
    edosign.vm.hostname = "edosign33-ubuntu"

    # Проброс портів
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

      echo "=== Клонування репозиторію Edo-Sign33 ==="
      su - vagrant -c "rm -rf ~/Edo-Sign33 && git clone https://github.com/onyshchenkodmytro/Edo-Sign33 ~/Edo-Sign33"

      echo "=== Копіювання коректного NuGet.Config ==="
      su - vagrant -c "mkdir -p ~/.nuget/NuGet"
      cp /vagrant/NuGet.Config /home/vagrant/.nuget/NuGet/NuGet.Config
      chown vagrant:vagrant /home/vagrant/.nuget/NuGet/NuGet.Config

      echo "=== Restore ==="
      su - vagrant -c "dotnet restore --no-dependencies ~/Edo-Sign33/EdoAuthServer/EdoAuthServer.csproj"
      su - vagrant -c "dotnet restore --no-dependencies ~/Edo-Sign33/EdoSign.Lab-3/EdoSign.Lab-3.csproj"


      echo "=== Publish EdoAuthServer ==="
      su - vagrant -c "dotnet publish ~/Edo-Sign33/EdoAuthServer/EdoAuthServer.csproj -c Release -o ~/auth"

      echo "=== Publish EdoSign.Lab-3 ==="
      su - vagrant -c "dotnet publish ~/Edo-Sign33/EdoSign.Lab-3/EdoSign.Lab-3.csproj -c Release -o ~/main"

      echo "=== Запуск EdoAuthServer (порт 7090) ==="
      su - vagrant -c "nohup dotnet ~/auth/EdoAuthServer.dll --urls=http://0.0.0.0:7090 > ~/auth.log 2>&1 &"

      echo "=== Запуск EdoSign.Lab-3 (порт 7275) ==="
      su - vagrant -c "nohup dotnet ~/main/EdoSign.Lab-3.dll --urls=http://0.0.0.0:7275 > ~/main.log 2>&1 &"

      echo "==========================================================================="
      echo "=== ГОТОВО! ==="
      echo "=== Сервер аутентифікації: http://localhost:7090 ==="
      echo "=== Основний застосунок:   http://localhost:7275 ==="
      echo "==========================================================================="
    SHELL
  end
end
