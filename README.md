# lab-09

Для работы будет использована ОС Windows.

Сначала скачиваем необходимое ПО:
- vagrant
- PuTTY
- Virtual Box

Затем создаем `Vagrantfile`

содердание: `Vagrantfile`:
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'json'
require 'yaml'

VAGRANTFILE_API_VERSION ||= "2"
confDir = $confDir ||= File.expand_path(File.dirname(__FILE__))

homesteadYamlPath = confDir + "/Homestead.yaml"
homesteadJsonPath = confDir + "/Homestead.json"
afterScriptPath = confDir + "/after.sh"
customizationScriptPath = confDir + "/user-customizations.sh"
aliasesPath = confDir + "/aliases"

require File.expand_path(File.dirname(__FILE__) + '/scripts/homestead.rb')

Vagrant.require_version '>= 2.2.4'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    if File.exist? aliasesPath then
        config.vm.provision "file", source: aliasesPath, destination: "/tmp/bash_aliases"
        config.vm.provision "handle_aliases", type: "shell" do |s|
            s.inline = "awk '{ sub(\"\r$\", \"\"); print }' /tmp/bash_aliases > /home/vagrant/.bash_aliases && chown vagrant:vagrant /home/vagrant/.bash_aliases"
        end
    end

    if File.exist? homesteadYamlPath then
        settings = YAML::load(File.read(homesteadYamlPath))
    elsif File.exist? homesteadJsonPath then
        settings = JSON::parse(File.read(homesteadJsonPath))
    else
        abort "Homestead settings file not found in #{confDir}"
    end

    Homestead.configure(config, settings)

    if File.exist? afterScriptPath then
        config.vm.provision "Run after.sh", type: "shell", path: afterScriptPath, privileged: false, keep_color: true
    end

    if File.exist? customizationScriptPath then
        config.vm.provision "Run customize script", type: "shell", path: customizationScriptPath, privileged: false, keep_color: true
    end

    if Vagrant.has_plugin?('vagrant-hostsupdater')
        config.hostsupdater.remove_on_suspend = false
        config.hostsupdater.aliases = settings['sites'].map { |site| site['map'] }
    elsif Vagrant.has_plugin?('vagrant-hostmanager')
        config.hostmanager.enabled = true
        config.hostmanager.manage_host = true
        config.hostmanager.aliases = settings['sites'].map { |site| site['map'] }
    elsif Vagrant.has_plugin?('vagrant-goodhosts')
        config.goodhosts.aliases = settings['sites'].map { |site| site['map'] }
    end

    if Vagrant.has_plugin?('vagrant-notify-forwarder')
        config.notify_forwarder.enable = true
    end
end
```
![Screenshot 2023-05-03 13-29-59](https://user-images.githubusercontent.com/125737299/235969195-b0ceb67d-4e26-46ec-9b02-f7c2756534fa.png)

Теперь нужно исползовать PuTTY чтобы создать SSH ключ ( ~/.ssh). И затем подключаемся с виртуальной машиной, использующей Vargant

![Screenshot 2023-05-03 13-31-10](https://user-images.githubusercontent.com/125737299/235969995-462ecafc-d639-44c3-b27a-db626fd5da80.png)
![Screenshot 2023-05-03 13-30-05](https://user-images.githubusercontent.com/125737299/235969657-d981893d-ea2f-4141-bb49-6563b66682cb.png)

Когда пробуем поменяь содержание папки `Code` через терминал виртуальной машины, содержание меняется также и на главном компьютере.

