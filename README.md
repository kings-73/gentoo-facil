# Guía de instalación Gentoo lo más sencilla posible
Esta guía proporciona los pasos para una instalación gentoo, lo más sencilla y genérica posible. La arquitectura será 64bits y para un arranque EFI.
Se utilizará el demonio OpenRC y no Systemd debido a que en Gentoo se usa por default, y no por preferencia personal. Así mismo el perfil de escritorio será "desktop" y no gnome o kde, para poder decidir más adelante el escritorio de su preferencia.

Para esta guía se infiere que el disco duro para la instalación será /dev/sda.

Por último, estoy conciente que uno de los puntos fuertes de Gentoo es la compilación manual y personalizado de un kernel, pero en esta guía pretende que el usuario se familiarize con la instalación general de gentoo, como si se tratase de una instalación de archlinux. Y posterior a ello, el usuario pueda realizar una instalación personalizada y más profunda.

## **1. Crear Partciones, Sistema de Archivos y Formatear**

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux swap; Tamaño: 2G
dev/sda3; Tipo: Linux file system; Tamaño: Resto del disco
```

### Formateo de particiones

`mkfs.fat -F 32 /dev/sda1`

`mkswap /dev/sda2`

`mkfs.ext4 /dev/sda3`

### Montaje de partición swap y gentoo

`swapon /dev/sda2`

`mount /dev/sda3 /mnt/gentoo`

## **2. Instalar el Stage comprimido y Enjaulamiento**

`date 061222002021`

**NOTA:** El formato para introducir la fecha es: MMddHHmmYYYY. En este ejemplo se setea: Junio 06 2021, Hora: 10:00 p.m.

`cd /mnt/gentoo`

`links https://www.gentoo.org/downloads`

`tar -xpvf stage3-*.tar.xz --numeric-owner --xattrs-include="*.*"`

**NOTA:** Sustituya el * por el nombre completo del stage.

`nano -w /mnt/gentoo/etc/portage/make.conf`

```
-* archivo make.conf *-
COMMON_CFLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j5"
ACCEPT_LICENSE="* -@EULA"
```

**NOTA:** La licencia "EULA" permitirá instalar linux-firmware que tontiene drivers no libres.

`cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

`mount --types proc /proc /mnt/gentoo/proc`

`mount --rbind /sys /mnt/gentoo/sys`

`mount --make-rslave /mnt/gentoo/sys`

`mount --rbind /dev /mnt/gentoo/dev`

`mount --make-rslave /mnt/gentoo/dev`

### **Entrar al nuevo entorno**

`chroot /mnt/gentoo /bin/bash`

`source /etc/profile`

`export PS1="(chroot) ${PS1}"`

### **Montar partición de arranque**

`mount /dev/sda1 /boot`

## **3. Configurar Portage y Elegir perfil**

`emerge-webrsync -v`

`emerge --ask --sync --quiet`

`eselect profile list`

`eselect profile set 5`

**NOTA:** Cambia el valor "5" por el deseado.


## **4. Zona Horaria**

`echo "America/Mexico_City" > /etc/timezone`

`emerge --config sys-libs/timezone-data`

`echo "es_MX.UTF-8 UTF-8" > /etc/locale.gen`

`locale-gen`

`eselect locale list`

`eselect locale set 4`

**NOTA:** De igual manera, cambia el valor "4" por el correspondiente.


## **5. Configurar el Nucleo Linux y Fstab**

`emerge --ask sys-kernel/gentoo-sources`

`ls -l /usr/src/linux`

**NOTA:** el comando "ls -l" muestra el enlace simbólico a linux hecho por portage. En caso de no tenerlo realizarlo manualmente con los siguientes comandos:

-* Apéndice para crear enlace simbólico a linux *-

`cd /usr/src`

`ls`

`ln -s linux-5.10.27-gentoo linux`

`cd`

**NOTA:** Sustituya "linux-5.10.27-gentoo" por el valor que arroje el comando "ls".
-* Fin del Apéndice*-


`emerge --ask sys-kernel/genkernel`

`emerge --ask sys-kernel/linux-firmware`

`genkernel all`

`nano -w /etc/fstab`

```
-* archivo fstab *-
/dev/sda1  boot  vfat  noatime  0 0
/dev/sda2  none  swap  sw       0 0
/dev/sda3  /     ext4  noatime  0 1
```


### **6. Instalacion de GRUB EFI**

`echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf`

**NOTA:** Si desea comprobar que el texto fue escrito en make.conf escriba: `cat /etc/portage/make.conf`


`emerge -av sys-boot/grub:2`

`grub-install --target=x86_64-efi --efi-directory=/boot`

`grub-mkconfig -o /boot/grub/grub.cfg`


### **7. Configurar Red, Reloj y teclado**

`nano -w /etc/conf.d/hostname`

```
-* archivo hostname *-
localhost="gentoo"
```

`nano -w /etc/hosts`

```
-* archivo hosts *-
127.0.0.1 localhost
127.0.1.1 gentoo
```

`nano -w /etc/conf.d/net`

```
-* archivo net *-
config_eth0=”dhcp”
```

`nano -w /etc/conf.d/hwclock`

```
-* archivo hwclock *-
clock="local"
```

`nano -w /etc/conf.d/keymaps`

```
-* archivo keymaps *-
keymap="es"
```


### **8. Instalar Herramientas**

`emerge -av net-misc/dhcpcd`

`emerge -av sys-apps/pciutils`

`emerge -av app-admin/sudo`


`emerge -av net-wireless wpa_supplicant`

**NOTA:** wpa_supplicant para uso de red wifi.


### **9. Password ROOT y salir del sistema**

`passwd`

`exit`

`cd`

`umount -l /mnt/gentoo/dev{/shm,/pts,} `

`umount -R /mnt/gentoo`

`reboot`

**¡Felicidades, ya has instalado gentoo!**
