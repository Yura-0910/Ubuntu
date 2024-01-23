# Ubuntu_команды

1. "sudo watch -n 5 free -m"   ::: мониторинг свободной оперативной памяти в реальном времени.
2. "sudo lshw -short -C memory"::: информация об оперативной памяти.
3. "sudo dmidecode"            ::: системная информация о ноутбуке.
4. "https://losst.pro/sbor-informatsii-o-sisteme-ubuntu"::: Сбор информации о системе Ubuntu
5. "sudo cat /proc/cpuinfo"    ::: информация о процессоре.
6. "sudo gnome-control-center" ::: запуск меню настройки.
7. Вот тут на видео (https://wifigid.ru/reshenie-problem-i-oshibok/ubuntu-ne-vidit-wi-fi):: описана моя проблема с wi-fi.
   Решение (когда Ubuntu не видет wi-fi сети):: "sudo service network-manager restart"
8. "sudo lspci"                ::: тут можно найти информацию, в том числе, про мой wi-fi модуль
9. Как установить драйвер именно для моей сетевой описано тут::: https://zalinux.ru/?p=4657
10. Решение проблемы с багом в драйвере wi-fi описано вот тут: 
    пользователь "GuyWithNvidia" сообщение от "2022-01-23 17:54:20" -> https://bbs.archlinux.org/viewtopic.php?id=273440
    https://aur.archlinux.org/packages/rtl8821ce-dkms-git
    https://github.com/tomaspinho/rtl8821ce#wi-fi-not-working-for-kernel--59
    https://issuehint.com/issue/lwfinger/rtw88/69   -> ".... Worked around it by disabling Power Management and setting Power Management default to Off."
    https://qna.habr.com/q/952731
    https://zalinux.ru/?p=4657
    https://yandex.ru/search/?text=rtw_8821ce+0000%3A02%3A00.0%3A+failed+to+send+h2c+command&lr=10777
    https://codeby.net/threads/ne-podkljuchaetsja-k-wifi-rtw_pci-0000-02-00-0-failed-to-send-h2c-command.74976/
    ЕЩЕ В ЖУРНАЛЕ UBUNTU С СООБЩЕНИЯМИ ОБ ОШИБКЕ, "rtw_8821ce" РЕГУЛЯРНО ВЫДАЕТ СООБЩЕНИЕ: "rtw_8821ce 0000:02:00.0: failed to send h2c command"
    Загрузитьт драйвер для сетевой -> "Обновления"  -> "Дополнительные драйверы" -> "Использовать dkms Realtek RTL8821CE"
11. "sudo lspci -nnk"         ::: Расширенная информация об устройствах, включая драйверы
12. Когда интернет отключается, то "systemd-resolve" выдает "Failed to send hostname reply: Invalid argument"
    Когда включаю ноут: "Failed to send hostname reply: Invalid argument"
                        "Failed to send hostname reply: Invalid argument"
                        "rtw_8821ce 0000:02:00.0: firmware failed to leave lps state"
                        "rtw_8821ce 0000:02:00.0: sta 1c:61:b4:d1:ab:20 with macid 0 left"
                        "rtw_8821ce 0000:02:00.0: sta 1c:61:b4:d1:ab:20 joined with macid"
13. Установка network-manager-gnome::: sudo apt update
                                       sudo apt install network-manager-gnome
    Запуск network-manager-gnome: "sudo nm-connection-editor"
14. опцию "Connectivity Check" в "NetworkManager" ставим "false", 
    описано вот тут-> 
              https://askubuntu.com/questions/1029108/how-do-i-programmatically-disable-connectivity-checking 
              https://askubuntu.com/questions/1029108/how-do-i-programmatically-disable-connectivity-checking 
    Использовал способ:: sudo crudini --set /var/lib/NetworkManager/NetworkManager-intern.conf "connectivity" ".set.enabled" "false"
15. Занесение модуля в черный список::
    Смотрим список загруженных модулей: "sudo lsmod", находим модули, что загружены: "rtw88_8821c            94208  1 rtw88_8821ce"
                                                                                     "rtw88_pci              32768  1 rtw88_8821ce"
                                                                                     "rtw88_core            258048  2 rtw88_pci,rtw88_8821c"
                                        "sudo lsmod | grep rtw88_8821ce", находим модули, что загружены:
                                                                                     rtw88_8821ce           16384  0
                                                                                     rtw88_8821c            94208  1 rtw88_8821ce
                                                                                     rtw88_pci              32768  1 rtw88_8821ce
    Проверяем, есть ли зависимости у модуля "rtw88_8821ce":: "sudo modinfo -F depends rtw88_8821ce" 
    Оказывается, у модуля "rtw88_8821ce" две зависимости: rtw88_pci,rtw88_8821c
    Вносим в черный список модуль "rtw88_8821ce" без зависимостей:
                            Создаем новый файл и Открываемем его на редоктирование: "sudo nano /etc/modprobe.d/rtw88_8821ce.conf"
                            В редакторе nano, сохранить: Ctrl+o
                                              выйти    : Ctrl+x
                                              Подробно о редакторе: https://routerus.com/how-to-use-nano-text-editor/
    Источник -> https://itsecforu.ru/2021/07/14/blacklist-%D0%B2-ubuntu-debian-linux/                                             
16. Интернет (wi-fi) не стабильный. Решение:
    1. сделал как описано вот тут: https://github.com/tomaspinho/rtl8821ce#wi-fi-not-working-for-kernel--59
    а именно как описано в разделе "Unstable connection - slowdowns or dropouts" и в разделе "Wi-Fi not working for kernel >= 5.9"
    2. А именно сделал: в "NetworkManager-intern.conf" добавил::
                                                                [connectivity]
                                                                .set.enabled=false  
      Сделал это с помощью команды:
     sudo crudini --set /var/lib/NetworkManager/NetworkManager-intern.conf "connectivity" ".set.enabled" "false" 
     Это описано вот тут:   https://askubuntu.com/questions/1029108/how-do-i-programmatically-disable-connectivity-checking
   3. Еще я создал файл /etc/modprobe.d/rtw88_8821ce.conf и заблакировал загрузку модуля rtw88_8821ce, с помощью команды : "blacklist rtw88_8821ce" (в файле rtw88_8821ce.conf). Потом проверил ("sudo lsmod | grep rtw88_8821ce"): модуль  rtw88 и его подмодули - не загрузились.
   ВНИМАНИЕ:::: P.S: перед выполнением настроек - сделать резервную копию всего диска на внешний
   P.S: еще не далал, сейчас зависимые модули от rtw88_8821ce: автоматически не загрузились, а в файле  /etc/modprobe.d/rtw88_8821ce.conf можно
   можно явно прописать, чтоб не загружались зависимые модули: вроде вот так добавить:: "install usbcore /bin/true", потом выполниить
   "update-initramfs -u" и перезагрузиться. Это описано вот тут: https://itsecforu.ru/2021/07/14/blacklist-%D0%B2-ubuntu-debian-linux/
   4. Если это не поможет, есть еще вариант сделать так, как описано у "GuyWithNvidia" в посте от "2022-01-23 17:54:20" вот тут он писал про такую же проблему: https://bbs.archlinux.org/viewtopic.php?id=273440
   5. Или попробовать сделать как описано в начале статьи:: https://github.com/tomaspinho/rtl8821ce#wi-fi-not-working-for-kernel--59 (из этого github ничего не устанавливать, так как это самописный драйвер) 
   ВНИМАНИЕ:::: P.S: перед выполнением настроек - сделать резервную копию всего диска на внешний HDD.   
17. Просмотреть содержимое файла: "sudo cat /var/lib/NetworkManager/NetworkManager-intern.conf"
18. "sudo grep -r "root" /etc/" - поиск текста во всех файлах папки "/etc/"
    "-r": чтоб во всех файлах искал
    "root" - строка поиска
    "/etc/" - папка поиска
19. Копирование файла в папку: "sudo cp откуда(путь-К-файлу) куда(путь-к-папке)"
20. Меняем права доступа к файлу: "sudo chmod -R o+rw путь-к-файлу-у-которого-меняем-права"
21. Загрузка процессора: sudo top -c -b | head -25
    %CPU : Процент процессора, используемого процессом.    
    https://itsecforu.ru/2019/12/12/%F0%9F%90%A7-%D0%BA%D0%B0%D0%BA-%D0%BD%D0%B0%D0%B9%D1%82%D0%B8-%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D1%8B-%D1%81-%D0%BD%D0%B0%D0%B8%D0%B1%D0%BE%D0%BB%D0%B5%D0%B5-%D0%B2%D1%8B%D1%81%D0%BE%D0%BA/
22. Как узнать температуру процессора, ядер ?
    https://losst.pro/izmerenie-temperatury-v-linux
    После установки, можно выполнить команду:  sudo sensors
    Графическая утилита: sudo psensor
23. Установка графического интерфейса для отключения OpenGL:
    sudo apt-get install compizconfig-settings-manager 
    Запуск: sudo ccsm
24. Список прав у файла:: ls -l /path/to/file
25. Изменение прав на файл: https://help.ubuntu.com/community/FilePermissions
    sudo chmod {options} filename
    sudo chmod g+w /path/to/file
    Пример:
    sudo chmod -R g+w /home/stalker/Документы/LearnJava-Prj/MultiThreading/target/classes/ru/lainer/course/eight10/deadlock
26. Изменение владельца и группы: sudo chown -R username:group directory
    R - это все файлы и саму папку 
    Пример:
    sudo chown -R stalker:stalker /home/stalker/Документы/LearnJava-Prj/MultiThreading/target/classes/ru/lainer/course/eight10/deadlock
27. Как найти pid запущенной программы ?
    ps aux | grep -i "telegram"
             или
    pidof Telegram         
28. Как завершить процесс по pid ?
    kill pid
29. Список wi-fi сетей и их уровень сигнала: sudo nmcli dev wifi list 
30. Посмотреть логи в режиме реального времени: tail -f 100 /var/log/syslog    
