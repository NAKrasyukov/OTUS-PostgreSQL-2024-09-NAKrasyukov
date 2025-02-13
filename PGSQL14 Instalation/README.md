# Установка Postgresql 14 из исходных кодов на ВМ с monjaro linux (Arch) 

## Создание виртуальной машины в VirtualBox

  1) Скачиваю образ дистрибутива Linux с Официального сайта
  2) Создаю новую вм со стандартными настройками:
     
     <img src="https://github.com/user-attachments/assets/5e1d6cd0-f5fb-49e8-b9b5-cd742df7c2ea" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/15b0e3c6-3b76-4924-93be-17608f1566a5" alt="drawing" width="500"/>
     
     <img src="https://github.com/user-attachments/assets/c9523dd1-5490-4c54-a784-4e50908c7899" alt="drawing" width="500"/>

  3) Запускаю ВМ и устанавливаю ОС

     <img src="https://github.com/user-attachments/assets/4219c519-c9b7-4beb-898b-fc67e6145904" alt="drawing" width="500"/>


## Установка Postgres14
**Для установки на Arch Linux, Postgres версии 14, необходимо использовать AUR. Для этого:**
  
  1) Устанавливаем python setup tools для возможности сборки бинарников из исходного кода:
     
     ``sudo pacman -S python-setuptools ``
     
  2) Устанавливаем cmake, ninja и gcc:

     ``sudo pacman -S cmake ninja gcc ``

  3) Устанавливаем base devel:

     ``sudo pacman -S base-devel ``

  4) Собираем из исходников зависимости llvm15  и clang15: 
    
     ``yay -S llvm15``
     
     ``yay -S clang15 ``

  5) Собираем из исходников сам Postgres-14:

     Так как при сборке пакетов определенных версий из исходников они устанавливаются в нестандартные директории а переменные среды не устанавливаются автоматически (пример: Мой путь для бинарного конфига llwm15 "/usr/bin/llvm-config-15" вместо "/usr/bin/llvm-config"), то необходимо задать эти параметры вручную в момент установки.
     
     ``yay -S postgresql14 --editmenu ``
     
     Параметр ``--editmenu`` позволяет отредактировать PKGBUILD перед началом установки.
     В функцию build() добавляю создание переменных, LLVM_CONFIG и CLANG

     ```
     build() {
       
     cd postgresql-${pkgver}
    
     #Указал путь к clang и llvm
     export LLVM_CONFIG=/usr/bin/llvm-config-15
     export CLANG=/usr/bin/clang-15
    
     local options=(
      --prefix=/opt/${pkgbase}
      --sysconfdir=/etc
      --with-gssapi
      --with-libxml
      --with-openssl
      --with-perl
      --with-python
      --with-tcl
      --with-pam
      --with-system-tzdata=/usr/share/zoneinfo
      --with-uuid=e2fs
      --with-icu
      --with-systemd
      --with-ldap
      --with-llvm
      --with-libxslt
      --enable-nls
      --enable-thread-safety
      --disable-rpath
     )
  
     ./configure "${options[@]}"
     make
     }
     ```

     Также сдесь я отключил функцию проверки установки check(), тк из-за нее установка вылетала в ошибку на моменте проверки часового пояса. 
     Далее происходит установка Postgres14.

  6) Запуск сервиса postgersql:

        Создаю символическую ссылку на libpq.so.5
        
        ``sudo ln -s /opt/postgresql14/lib/libpq.so.5 /usr/lib64/libpq.so.5``

        Инициализирую базу данных с указанием каталога хранения

        ``sudo -u postgres /opt/postgresql14/bin/initdb --locale=en_US.UTF-8 -D /var/lib/postgres/data14 ``

        Запускаю сервис ``sudo systemctl start postgresql14.service`` ``sudo systemctl enable postgresql14.service``

        Проверяю сервис ``sudo systemctl status postgresql14.service``
        
        <img src="https://github.com/user-attachments/assets/5c2af5de-2f74-4d92-8b73-e6ea240cc523" alt="drawing" width="500"/>

        Все работает!

  ## Трудности с которыми столкнулся
  Сборка из иходников - очень долгий и ресурсоемкий процесс. Нексолько раз при сборке зависисмостей ВМ зависала и крашилась. Изначальной конфигураций ВМ оказалось     недостаточно для этого процесса. Пришлось увеличить количество ядер вм до 5, а оперативной памяти до 11ГБ. Также изменил нараметры конфигурации /etc/makepkg.conf установив параметр ``MAKEFLAGS="-j4"``, позволяя использовать 4 ядра процессора для сборки (изначально значение было 2). 
  Только после этого сборка бинарников из исходного кода прошла успешно.
