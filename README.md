# egpu-switcher


Несложный скрипт для пользователей ноутбучных адаптеров для внешних видеокарт. Призван упростить выбор видеокарты по умолчанию без ковыряния конфигов руками.
---

- [Описание](#description)
- [Скриншот](#screenshot)
- [Требования](#requirements)
- [Установка](#installation)
  - [Ubuntu (apt)](#ubuntu-apt)
  - [Other](#other)
- [Команды](#commands)
- [Проблемы](#troubleshooting)
- [Background Information](#background-information)

---

## Description
Цель этого скрипта - упростить настройку адаптера eGPU в вашем дистрибутиве Linux. Установка проходит в интерактивном режиме с понятными пояснениями.

**Достижение функционала как в Windows пока не представляется возможным.<br> Скорее всего после применения скрипта необходимо перезагрузить компьютер для полного функционала eGPU.**.

## Screenshot
![Screenshot of setup](https://raw.githubusercontent.com/hertg/egpu-switcher/master/images/screenshot_setup.png)

## Requirements
1. Система работает на X-Server.
1. Установлена **pciutils 3.3.x или выше** installed (проверить командой `lspci --version`).
1. Минимум **Bash 4.x или выше**.
1. Настроенный физически и подключеный физически к ноутбуку адаптер eGPU.
1. Полностью установленный драйвер на подключенную через eGPU видеокарту.

---

## Installation

### Ubuntu (apt)
Установка и настройка:
```bash
$ sudo add-apt-repository ppa:hertg/egpu-switcher
$ sudo apt update
$ sudo apt install egpu-switcher
$ sudo egpu-switcher setup
```

Удаление:
```bash
$ apt remove egpu-switcher
```

### Other
Установка и настройка:
```bash
$ git clone git@github.com:hertg/egpu-switcher.git
$ cd egpu-switcher
$ make install
$ sudo egpu-switcher setup
```

Удаление: 
> **Внимание!**: **Не используйте данный метод на версиях ниже `0.10.2`!**\
> Была критическая ошибка в Makefile, которая удаляет папку `/usr/bin`. Лучше вручную удалить папку `egpu-switcher` в каталогах `/usr/bin/` и `/usr/share/`.
```bash
$ sudo egpu-switcher cleanup
$ make uninstall
```

---

## Commands
<pre>
<b>egpu-switcher setup</b> [--override] [--noprompt]
    This will generate the "xorg.conf.egpu" and "xorg.conf.internal" files and
    symlink the "xorg.conf" file to one of them.
    
    It will also create the systemd service, that runs the "switch" command on each bootup.
    
    This will NOT delete any already existing files. If an "xorg.conf" file already exists, 
    it will be backed up to "xorg.conf.backup.{datetime}". This can later be reverted by
    executing the "cleanup" command.

    <b>--override</b>
        If an AMD GPU or open-source NVIDIA drivers are used, the "switch" command 
        will prevent from switching to the eGPU if there are no displays directly attached to it. 
        This flag will make sure to switch to the EGPU even if there are no displays attached.

    <b>--noprompt</b>
        Prevent the setup from prompting for user interaction if there is
        no existing configuration file found. (Is currently only used by the "postinst" script)
</pre>

<pre>
<b>egpu-switcher switch auto|egpu|internal</b> [--override]
    Switches to the specified GPU. if the <u>auto</u> parameter is used, the script will check if 
    the eGPU is attached and switch accordingly. 
    
    The computer (or display-manager) needs to be restarted for this to take effect.

    <b>--override</b>
        If an AMD GPU or open-source NVIDIA drivers are used, the "switch" command 
        will prevent from switching to the eGPU if there are no displays directly attached to it. 
        This flag will make sure to switch to the EGPU even if there are no displays attached.
</pre>

<pre>
<b>egpu-switcher cleanup</b> [--hard]
    Remove all files egpu-switcher has created previously and restore the backup
    of previous "xorg.conf" files.

    <b>--hard</b>
        Remove configuration files too.
</pre>

<pre>
<b>egpu-switcher config</b>
    Prompts the user to specify their external/internal GPU and saves their answer
    to the configuration file.
</pre>

<pre>
<b>egpu-switcher remove</b>
    Allows the user to remove their eGPU without a complete reboot.
    This method will still restart the display-manager, and therefore terminate all its child-processes.
</pre>

---

## Troubleshooting
If you run into problems, please have a look at [TROUBLESHOOT.md](https://github.com/hertg/egpu-switcher/blob/master/TROUBLESHOOT.md) before reporting any issues.

---

## Background information
> A backup of your current `xorg.conf` will be created, nothing gets deleted. If the script doesn't work for you, you can revert the changes by executing `egpu-switcher cleanup` or just completely uninstall the script with `apt remove egpu-switcher`. This will remove all files it has created and also restore your previous `xorg.conf` file.

This script will create two configuration files in your X-Server folder `/etc/X11`.
The file `xorg.conf.egpu` holds the settings for your EGPU and the file `xorg.conf.internal` holds the settings for your internal graphics.

Then a symlink `xorg.conf` will be generated which points to the corresponding config file, depending on wheter your egpu is connected or not.

Additionally a custom `systemd` service with the following content will be created.

*/etc/systemd/system/egpu.service*
```bash
[Unit]
Description=EGPU Service
Before=display-manager.service
After=bolt.service

[Service]
Type=oneshot
ExecStart=/usr/bin/egpu-switcher switch auto

[Install]
WantedBy=graphical.target
```

This will enable the automatic detection wheter your egpu is connected or not on startup.
