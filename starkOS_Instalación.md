## StarkOS GNU/Linux instalación: https://stark-os.codeberg.page/index.html

- Arranca desde un ISO Live, preferiblemente de StarkOS.

- Descarga el último rootfs tarball con SysV, Runit o S6.

- Crea y prepara las particiones:

1 - BIOS system con MBR.

cfdisk

mkswap -L swap /dev/sda1

mkfs.ext4 -L starkOS /dev/sda2

- Crea un directorio para montar la partición creada y monta esta ahí.

mkdir /mnt/starkos

mount /dev/sda2 /mnt/starkos

2 - EFI system con GPT. EFI system requiere una partición adicional vfat para /boot/efi, creala primero.

cfdisk

mkfs.vfat -n boot /dev/sda1

mkswap -L swap /dev/sda2

mkfs.ext4 -L starkOS /dev/sda3

- Crea los directorios para montar /root and /boot/efi.

mkdir /mnt/starkos

mount /dev/sda3 /mnt/starkos

mkdir -pv /mnt/starkos/boot/efi

mount /dev/sda1 /mnt/starkos/boot/efi

swapon /dev/sda2

- Extrae la imagen de starkOS en el punto de montaje elegido.

tar xvJpf starkos-rootfs-version-x86_64.tar.xz -C /mnt/starkos

- Entrar desde chroot.

- Montar y preparar los directorios dentro de la imagen de starkOS desconprimida, para un correcto chroot.

mount -v --bind /dev /mnt/starkos/dev

mount -vt devpts devpts /mnt/starkos/dev/pts -o gid=5,mode=620

mount -vt proc proc /mnt/starkos/proc

mount -vt sysfs sysfs /mnt/starkos/sys

mount -vt tmpfs tmpfs /mnt/starkos/run

mkdir -pv /mnt/starkos/$(readlink /mnt/starkos/dev/shm)

cp -L /etc/resolv.conf /mnt/starkos/etc/

chroot /mnt/starkos /bin/bash

- Configuración del sistema: hostname, timezone, clock, font, keymap y daemon:

vim /etc/rc.conf

- Configurar el directorio /etc/fstab, ver ejemplo.

vim /etc/fstab

PARTUUID=C306-F008 /boot/efi vfat defaults 0 2

PARTUUID=d28dd521-a874-4939-aacb-1f740102bbac none swap pri=1 0 0

PARTUUID=2cfac6a6-9670-45fe-a6ab-11735228ff4f / ext4 defaults 1 1

tmpfs /tmp tmpfs rw,nosuid,noatime,nodev,mode=1777,size=2G 0 0

- Configurar zona horaria.

ln -sf /usr/share/zoneinfo/Region/City /etc/localetime

- Configurar reloj del sistema.

hwclock --systohc

- Configurar locales.

vim /etc/locales

- Descomentar el deseado y ejecutar comando "genlocales".

genlocales

- Configurar idioma local en Español export LANG=es_ES.UTF-8 dentro de /etc/locale.conf

vim /etc/locale.conf

- Crear nombre del sistema y configurar red local.

vim /etc/host

vim /etc/hosts

- Configurar un root password.

passwd

- Establecer permisos para la cuenta de root.

chown root:root /

chmod 755 /

- Añadir un usuario.

useradd -m -G users,wheel,audio,video -s /bin/bash "usuario"

- Crear un password para el nuevo asuario.

passwd "usuario"

- Sincronizar repositorios.

portsync -r

stark sync

- Actualizar el sistema.

stark sysup

- Instalación del Kernel.

stark install linux

- Nota: “remplaza 'linux' por 'linux-lts' si prefieres la versión lts”.

- Configurar un gestor de arranque, por ejemplo: grub.

1 - BIOS:

grub-install /dev/sdX

grub-mkconfig -o /boot/grub/grub.cfg

Nota: “remplaza 'X' con tu partición de arranque”.

2 - EFI: (comprobar antes si tienes efivars).                                

mountpoint /sys/firmware/efi/efivars  

- Montar el directorio efivars.

mount -v -t efivarfs none /sys/firmware/efi/efivars

df -Th

- Instala el paquete 'grub-efi'.

stark install grub-efi

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=starkos

grub-mkconfig -o /boot/grub/grub.cfg

- Salir del entorno de chroot.

exit

- Desmontar directorios de /mnt/starkos antes de salir de la instalación.

umount -v /mnt/starkos/dev/pts

umount -v /mnt/starkos/dev

umount -v /mnt/starkos/run

umount -v /mnt/starkos/proc

umount -v /mnt/starkos/sys

umount -R /mnt/starkos

- Ahora puedes reiniciar la máquina, starkOS GNU/Linux debería de ser un sistema arrancable.

reboot
