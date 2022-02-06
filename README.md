# Домашнее задание - Обновление ядра Linux
<ol>
  <li>Развернуть VirtualBox</li>
  <li>Развернуть Vagrant</li>
  <li>Развернуть Packer</li>
  <li>Развернуть Git</li>
  <li>Подключить и склонировать репозиторий git</li>
  <li>Обновить ядро</li>
  <li>Создать свой образ системы</li>
  <li>Загрузить образ в Vagrant Cloud</li>
</ol>

# 1.Развернуть VirtualBox
<ul>
  <li>Устанавливаем необходимые утилиты</li>
    sudo apt install wget
  <li>Качаем .deb пакет и сохраняем его в директрории /tmp под именем virtualbox.deb</li>
    wget https://download.virtualbox.org/virtualbox/6.1.32/virtualbox-6.1_6.1.32-149290~Ubuntu~eoan_amd64.deb -O virtualbox.deb
  <li>Устанавливаем .deb пакет</li>
    sudo apt-get install ./virtualbox.deb
</ul> 

# 2.Развернуть Vagrant
<ul>
  <li>Скачиваем файл в директрию /tmp под именем vagrant.zip</li>
     wget https://releases.hashicorp.com/vagrant/2.2.19/vagrant_2.2.19_linux_amd64.zip -O vagrant.zip   
  <li>Распаковываем vagrant.zip</li>
    unzip vagrant.zip
  <li>Перемещаем биарный файл в директорию /bin</li>  
    sudo mv vagrant /bin/
</ul>

# 3.Развернуть Packer
<ul>
<li>Устанавливаем необходимые утилиты</li>
    sudo apt install curl
<li>Переходим на сайт https://www.packer.io/downloads и копируем ссылку и выполняем комманды</li>
  curl https://releases.hashicorp.com/packer/1.7.10/packer_1.7.10_windows_amd64.zip | \
  sudo gzip -d > /usr/local/bin/packer && \
  sudo chmod +x /usr/local/bin/packer
<li>Проверяем установку</li>  
  Packer --version
</ul>

# 4.Развернуть Git
<ul>
<li>Устанавливаем Git</li>
  sudo apt install Git
<li>Генерируем ssh</li> 
  ssh-keygen
<li>Переходим в директорию ssh</li>
  cd ~/.ssh
<li>Проверяем сгенерированные ключи</li>
  ls
  
  Если ключи сгенерировались то мы увидим два файла id_rsa и id_rsa.pub
<li>Открываем public key ssh</li>
  cat id_rsa.pub
<li>Делаем fork репозитория: https://github.com/dmitry-lyutenko/manual_kernel_update</li>  
<li>Добавляем полученные данные в GitHub</li>
</ul>

# 5.Подключить и склонировать репозиторий git
<ul>
<li>В Домашнем каталоге создаём директорию для работы с Git</li>
  <p>cd ~<br>
  mkdir /vagrant</p>
  cd /vagrant</p>
<li>Клонируем репозиторий к себе на машину</li>
  <p>git clone git@github.com:<user_name>/manual_kernel_update.git<br>
  После успешного выполнения в директории /vagrant появиться каталог manual_kernel_update</p>
<li>Проверим каталог manual_kernel_update</li>
  <p>cd ~<br>
  <p>cd vagrant/manual_kernel_update<br>
  <p>ls<br>
  <p>В каталоге должны быть две папки "manual", "packer" и один файл "Vagrantfile"<br>
</ul>

# 6.Обновить ядро
<ul>
<li>Запускаем VM</li>
  vagrant up
<li>Убеждаемся, что VM развернулась</li>
  vagrant status
<li>Логинимся</li>
  vagrant ssh kernel-update
<li>Подключаем репозиорий с последней версией ядра</li>  
  sudo yum install -y http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
<li>Устанавливаем последнюю версию ядра</li>
  sudo yum --enablerepo elrepo-kernel install kernel-ml -y
<li>Обновляем конфигурацию загрузчика</li> 
  sudo grub2-mkconfig -o /boot/grub2/grub.cfg
<li>Выбираем загрузку с новым ядром по-умолчанию</li>
  sudo grub2-set-default 0
<li>Перезагружаем VM</li>
  sudo reboot  
<li>После перезагрузки проверяем версию ядра</li>
  uname -r
</ul>

# 7.Создать свой образ системы
<ul>
  <li>Запускаем packer</li>
    packer build centos.json
  <li>При удачном выполнении в ранне созданной директории /vagrant/manual_kernel_update/packer появиться файл centos-7.7.1908-kernel-5-x86_64-Minimal.box</li>
  <li>Для тестирование созданного образа выполняем его импорт в vagrant</li>
    vagrant box add --name centos-7-5 centos-7.7.1908-kernel-5-x86_64-Minimal.box
  <li>Проверим список имеющихся образов</li>
    vagrant box list
  <li>Тестирование полученного образа</li>
    <ul>
      <li>Создадим новую директорию /vagrant/manual_kernel_update/test</li>
        <p>cd ~<br>
        cd /vagrant/manual_kernel_update/<p>
        mkdir /test<p>
      <li>Выполним команду vagrant init centos-7-5</li>
      <li>Заменим значение box_name на имя импортированного образа</li>
        :box_name => "centos-7-5"
      <li>Запустим VM</li>
        vagrant up
      <li>Выполним vagrant status</li>
      <li>Залогинимся на VM</li>
        vagrant ssh
      <li>Проверим версию ядра</li>
        uname -r
      <li>Удалим тестовый образ из локального хранилища</li>
        vagrant box remove centos-7-5
    </ul>
</ul>
# 8.Загрузить образ в Vagrant Cloud
<ul>
<li>Авторизуемся и загружаем в Vagrant Cloud</li>
</ul>
vagrant cloud publish --release sergeyorb/centos-7-5 1.0 virtualbox \centos-7.7.1908-kernel-5-x86_64-Minimal.box
