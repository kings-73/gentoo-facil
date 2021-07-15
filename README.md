# Guía de instalación Gentoo lo más sencilla posible
Esta guía proporciona los pasos para una instalación gentoo, lo más sencilla y genérica posible. La arquitectura será 64bits y para un arranque EFI.
Se utilizará el demonio OpenRC y no Systemd debido a que en Gentoo se usa por default, y no por preferencia personal. Así mismo el perfil de escritorio será el "default" y no gnome o kde, para poder decidir más adelante el escritorio de su preferencia.

Para esta guía se infiere que el disco duro para la instalación será /dev/sdb.

Por último, estoy conciente que uno de los puntos fuertes de Gentoo es la compilación manual y personalizado de un kernel, pero en esta guía pretende que el usuario se familiarize con la instalación general de gentoo, y posterior a ello, el usuario pueda realizar una instalación personalizada y más profunda.

## **1. Crear Partciones, Sistema de Archivos y Formatear**

`cfdisk /dev/sdb`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sdb1; Tipo: Efi system; Tamaño: 150M
dev/sdb2; Tipo: Linux swap; Tamaño: 2G
dev/sdb3; Tipo: Linux file system; Tamaño: Resto del disco
```

### Formateo de particiones

`mkfs.fat -F 32 /dev/sdb1`

`mkswap /dev/sdb2`

`mkfs.ext4 /dev/sdb3`

### Montaje de partición swap y gentoo

`swapon /dev/sdb2`

`mount /dev/sdb3 /mnt/gentoo`

## **2. Instalar el Stage comprimido y Enjaulamiento**

`date 071509002021`

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
GRUB_PLATFORMS="efi-64"
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

`mount /dev/sdb1 /boot`

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

**NOTA:** De igual manera, cambie el valor "4" por el deseado.


## **5. Configurar el Nucleo Linux y Fstab**

`emerge --ask sys-kernel/gentoo-sources`

`ls -l /usr/src/linux`

**NOTA:** el comando "ls -l" muestra el enlace simbólico a linux hecho por portage. En caso de no tenerlo realizarlo manualmente con los siguientes comandos:

### **(Opcional) sólo en caso que no aparezca enlace simbólico**

`eselect kernel list`

`eselect kernel set 1`

**NOTA:** Sustituya el valor "1" por el deseado en caso de ser necesario.

### **Compilación del núcleo linux**

`emerge --ask sys-kernel/genkernel`

`emerge --ask sys-kernel/linux-firmware`

`genkernel all` ó `genkernel --menuconfig all`

**NOTA:** genkernel all para compilación completa (nuevos usuarios) ó genkernel --menuconfig all para usuarios experimentados que conozcan su harwware.

`nano -w /etc/fstab`

```
-* archivo fstab *-
/dev/sdb1  boot  vfat  noatime  0 0
/dev/sdb2  none  swap  sw       0 0
/dev/sdb3  /     ext4  noatime  0 1
```


### **6. Instalacion de GRUB EFI**

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
127.0.0.1 gentoo.redhogar gentoo localhost
```

`nano -w /etc/conf.d/net`

```
-* archivo net *-
config_eth0="dhcp"
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

`emerge -av sys-apps/pciutils`


`emerge -av net-misc/dhcpcd`

`emerge -av net-wireless/iw`

`emerge -av net-wireless/wpa_supplicant`

**NOTA:** iw y wpa_supplicant para uso de red wifi.


### **9. Password ROOT y salir del sistema**

`passwd`

`exit`

`cd`

`umount -l /mnt/gentoo/dev{/shm,/pts,} `

`umount -R /mnt/gentoo`

`reboot`

**¡Felicidades, ya has instalado gentoo!**
