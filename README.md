# Guía de instalación Gentoo lo más sencilla posible
Esta guía proporciona los pasos para una instalación gentoo, lo más sencilla y genérica posible. La arquitectura del equipo será de 64bits. Por otro lado el modo de arranque será EFI.

Respecto al sistema de inicio usaremos el preferido de Gentoo que es OpenRC, ya que el handbook proporciona una guía detallada para este init.

A su vez, para esta guía se usará el disco duro "/dev/sda" para la instalación.

Por último, estoy conciente que uno de los puntos fuertes de Gentoo es la compilación manual y personalizado de un kernel, pero en esta guía pretende que el usuario se familiarize con la instalación general de gentoo, y posterior a ello, el usuario pueda realizar la compilación del kernel con gentoo ya funcionando.

## **0. Conexión a Wi-Fi**

Un método sencillo para conectarnos al Wi-Fi es con net-setup. Pero debe tener en cuenta que necesita de tres datos importantes:

1. Nombre del dispositivo de red inalámbrica (INTERFAZ)
2. El nombre de nuestra red Wi-Fi (SSID)
3. La contraseña de red (PASSWOWRD)

Escribimos el comando:

`net-setup`

En esta guía introduciremos datos como: El nombre del SSID al que nos conectaremos, el tipo de contraseña que usa, la contraseña como tal y si queremos usar DHCP automático. Una vez finalizando los datos podemos verificar nuestra conexión con:

`ping -c3 www.gentoo.org`

**NOTA:** Si no es posible conectarse con este método, puede también hacerlo por medio de USB compartiendo internet desde su dispositivo Android. De esta manera no necesita realizar configuraciones adicionales.

## **1. Preparando el disco**

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux file system; Resto del disco
```

### Formateo de particiones y montaje de la partición raíz

`mkfs.fat -F 32 /dev/sda1`

`mkfs.ext4 /dev/sda2`



`mount /dev/sda2 /mnt/gentoo`



## **2. Instalar el Stage comprimido**

`date 071509002021`

**NOTA:** El formato para introducir la fecha es: MMDDhhmmYYYY. En este ejemplo se setea: Junio 06 2021, Hora: 10:00 p.m.

`cd /mnt/gentoo`

`links www.gentoo.org/downloads`

`tar -xpf stage3-amd64-openrc-*.tar.xz --numeric-owner --xattrs-include="*.*"`

**NOTA:** Sustituya el * por el nombre completo del stage.

`nano -w /mnt/gentoo/etc/portage/make.conf`

```
-* archivo make.conf *-
COMMON_CFLAGS="-march=native -O2 -pipe"
MAKEOPTS="-j2"

GRUB_PLATFORMS="efi-64"

L10N="es-MX es"
USE="elogind networkmanager -systemd"
```

**NOTA:** La licencia "EULA" permitirá instalar linux-firmware que tontiene drivers no libres.

## **3. Enjaulamiento**

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

## **4. Configurar Portage y Elegir perfil**

`emerge-webrsync -v`

`emerge --ask --sync --quiet`

`emerge --ask --verbose --update --deep --newuse @world`

**NOTA:** Sustituya la X por el número del perfil deseado.

## **5. Zona Horaria**

`echo "America/Mexico_City" > /etc/timezone`

`emerge --config sys-libs/timezone-data`

`echo "es_MX.UTF-8 UTF-8" > /etc/locale.gen`

`locale-gen`

`eselect locale list`

`eselect locale set 4`

**NOTA:** De igual manera, cambie el valor "4" por el deseado.

## **6. Instalar un kernel binario y el firmware no libre**

`emerge --ask sys-kernel/gentoo-kernel-bin`

`echo "sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE" > /etc/portage/package.license`

`emerge --ask sys-kernel/linux-firmware`

### **7. Archivos de configuración**

`nano -w /etc/fstab`

```
-* archivo fstab *-
/dev/sda1  boot  vfat  noatime  0 0
/dev/sda2  none  swap  sw       0 0
/dev/sda3  /     ext4  noatime  0 1
```

`nano -w /etc/conf.d/hostname`

```
-* archivo hostname *-
localhost="gentoo"
```

`nano -w /etc/hosts`

```
-* archivo hosts *-
127.0.0.1 gentoo.redhogar gentoo localhost
::1 gentoo
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

### **8. Instalacion de GRUB EFI**

`emerge -av sys-boot/grub:2`

`grub-install --target=x86_64-efi --efi-directory=/boot`

`grub-mkconfig -o /boot/grub/grub.cfg`

### **9. Instalar Herramientas**

`emerge -av net-misc/dhcpcd`

`emerge -av gentoolkit`

### **10. Password ROOT y salir del sistema**

`passwd`

`exit`

`cd`

`umount -l /mnt/gentoo/dev{/shm,/pts,} `

`umount -R /mnt/gentoo`

`reboot`

**¡Felicidades, ya has instalado gentoo!**
