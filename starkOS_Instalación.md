## StarkOS GNU/Linux instalación:

- Arranca desde un ISO Live, preferiblemente de StarkOS.

- Descarga el último rootfs tarball con SysV, Runit o S6.

- Crea y prepara las particiones: BIOS system con MBR.

cfdisk

mkswap /dev/sda1

mkfs.ext4 -L StarkOS /dev/sda2

- Crea un directorio para montar la partición creada y monta esta ahí.

mkdir /mnt/starkos

mount /dev/sda2 /mnt/starkos

- EFI system con GPT. EFI system requiere una partición adicional vfat para /boot/efi, creala primero.

cfdisk

mkfs.vfat /dev/sda1

mkswap /dev/sda2

mkfs.ext4 -L StarkOS /dev/sda3

- Crea los directorios para montar /root and /boot/efi.

mkdir -pv /mnt/starkos/boot/efi

mount /dev/sda3 /mnt/starkos

mount /dev/sda1 /mnt/starkos/boot/efi

swapon /dev/sda2

- Extrae la imagen de starkOS en el punto de montaje elegido.

tar xvJpf starkos-rootfs--x86_64.tar.xz -C /mnt/starkos

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

- Configurar el directorio /etc/fstab.

vim /etc/fstab

- <device> <dir> <type> <options> <dump> <fsck>
PARTUUID=C306-F008 /boot/efi vfat defaults 0 2

PARTUUID=d28dd521-a874-4939-aacb-1f740102bbac none swap pri=1 0 0

PARTUUID=2cfac6a6-9670-45fe-a6ab-11735228ff4f / ext4 defaults 1 1

tmpfs /tmp tmpfs rw,nosuid,noatime,nodev,mode=1777,size=2G 0 0

- Configurar zona horaria.

ln -sf /usr/share/zoneinfo/Region/City /etc/localetime

Configurar reloj del sistema.

hwclock --systohc

Configurar locales.

vim /etc/locales

Descomentar el deseado y ejecutar comando "genlocales".

genlocales

Crear nombre del sistema y configurar red local.

vim /etc/host

vim /etc/hosts

Configurar un root password.

passwd

chown root:root /

chmod 755 /

Añadir un usuario.

useradd -m -G users,wheel,audio,video -s /bin/bash

Crear un password para el nuevo asuario.

passwd

Sincronizar repositorios.

portsync -r

stark sync

Actualizar el sistema.

stark sysup

Instalación del Kernel.

stark install linux

Nota: “remplaza 'linux' por 'linux-lts' si prefieres la versión lts”

Configurar un gestor de arranque, por ejemplo: grub.

## BIOS:

grub-install /dev/sdX

grub-mkconfig -o /boot/grub/grub.cfg

Nota: “remplaza 'X' con tu partición de arranque”

## EFI:

df -Th                              (para comprobar efivars)

mountpoint /sys/firmware/efi/efivars  (comprobar si tienes efivars)

Montar el directorio efivars.

mount -v -t efivarfs efivarfs /sys/firmware/efi/efivars

Instala el paquete 'grub-efi'.

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=starkos

grub-mkconfig -o /boot/grub/grub.cfg

Salir del entorno de chroot.

exit

Desmontar directorios de /mnt antes de salir de la instalación.

umount -v /mnt/starkos/dev/pts

umount -v /mnt/starkos/dev

umount -v /mnt/starkos/run

umount -v /mnt/starkos/proc

umount -v /mnt/starkos/sys

umount -R /mnt/starkos

Ahora puedes reiniciar la máquina, starkOS GNU/Linux deberia de ser un sistema arrancable.

reboot
