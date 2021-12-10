# Guía de instalación Gentoo lo más sencilla posible
Esta guía proporciona los pasos para una instalación gentoo, lo más sencilla y genérica posible. La arquitectura será 64bits y para un arranque EFI.
Se utilizará el demonio OpenRC y no Systemd debido a que en Gentoo se usa por default, y no por preferencia personal. Así mismo el perfil de escritorio será el "default" y no gnome o kde, para poder decidir más adelante el escritorio de su preferencia.

Para esta guía se infiere que el disco duro para la instalación será /dev/sdb.

Por último, estoy conciente que uno de los puntos fuertes de Gentoo es la compilación manual y personalizado de un kernel, pero en esta guía pretende que el usuario se familiarize con la instalación general de gentoo, y posterior a ello, el usuario pueda realizar una instalación personalizada y más profunda.

## **0. Conexión a Wi-Fi**

Un método sencillo para conectarnos al Wi-Fi es con net-setup. Pero debe tener en cuenta que necesita de tres datos importantes:

1. Nombre del dispositivo de red inalámbrica (INTERFAZ)
2. El nombre de nuestra red Wi-Fi (SSID)
3. La contraseña de red (PASSWOWRD)

Los datos 2 y 3 son proporcionados por su proveedor de internet. En caso de tenerlos a la mano. Sólo falta saber el nombre de nuestra interfaz de red con el siguiente comando:

`iwconfig`

Una vez que cuente con el nombre del dispositivo de red, proceda a ejecutar net-setup añadiendo el nombre de la interfaz:

`net-setup INTERFAZ`

**NOTA:** No se olvide de cambiar la palabra INTERFAZ por el nombre del dispositivo de red que obtuvo con iwconfig.

Si no es posible conectarse con este método, puede también hacerlo por medio de USB compartiendo internet desde su dispositivo Android. De esta manera no necesita realizar configuraciones adicionales.

## **1. Preparando el disco**

`cfdisk /dev/sda`

**NOTA:** Si el disco no cuenta con tabla de particiones seleccione GPT (GUID Partition Table).

```
dev/sda1; Tipo: Efi system; Tamaño: 150M
dev/sda2; Tipo: Linux file system; Tamaño: 15G
dev/sda3; Tipo: Linux file system; Tamaño: Resto del disco
```

### Formateo de particiones

`mkfs.fat -F 32 /dev/sda1`

`mkfs.ext4 /dev/sda2`

`mkfs.ext4 /dev/sda3`

### Montaje de partición swap y gentoo

`mount /dev/sda2 /mnt/gentoo`

`mkdir -p /mnt/gentoo/home`

`mount /dev/sda3 /mnt/gentoo/home`

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
MAKEOPTS="-j4"

GRUB_PLATFORMS="efi-64"
VIDEO_CARDS="intel i965 iris"

L10N="es-MX es"
USE="elogind networkmanager -systemd -qt5"
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

`eselect profile list`

`eselect profile set X`

**NOTA:** Sustituya la X por el número del perfil deseado.

## **5. Zona Horaria**

`echo "America/Mexico_City" > /etc/timezone`

`emerge --config sys-libs/timezone-data`

`echo "es_MX.UTF-8 UTF-8" > /etc/locale.gen`

`locale-gen`

`eselect locale list`

`eselect locale set 4`

**NOTA:** De igual manera, cambie el valor "4" por el deseado.

## **6. Configurar el Nucleo Linux**

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
