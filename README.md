# DualBoot

#Tuve el mismo problema después de instalar Arch. Los siguientes comandos me #funcionaron perfectamente:

sudo apt-get install os-prober

Primero instale os-prober para detectar ventanas y luego actualice grub:

sudo grub-mkconfig -o /boot/grub/grub.cfg

muy facil...

Tambien existen otras maneras:

mount -t ntfs-3g -o ro /dev/sda4 /media/windows

-------/////////////---------------

Si el método os-prober anterior no funciona, intente agregar una entrada de menú grub personalizada. 
Documentación:
https://wiki.archlinux.org/index.php/GRUB_(Espa%C3%B1ol)#Archivo_grub.cfg_personalizado
https://help.ubuntu.com/community/Grub2/CustomMenus

1) Los primeros dos pasos son para encontrar su <UUID>.

2) Ejecute lsblk y busque el nombre de la fila con /boot/efi
3) Ejemplo de salida (aquí la respuesta es sda2):

4) lsblk

NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0   477G  0 disk 
├─sda1        8:1    0   450M  0 part 
├─sda2        8:2    0   100M  0 part /boot/efi
├─sda3        8:3    0    16M  0 part 
├─sda4        8:4    0    47G  0 part /windows
├─sda5        8:5    0 425,6G  0 part /
└─sda6        8:6    0   3,7G  0 part [SWAP]
mmcblk0     179:0    0  14,9G  0 disk 
└─mmcblk0p1 179:1    0  14,9G  0 part

5) Ejecute sudo blkid /dev/sdaX donde sdaX es la respuesta del paso anterior (sda2 en mi caso).

6) Ejemplo de salida (aquí la respuesta es 58E4-427D):

7) /dev/sda2: UUID="58E4-427D" TYPE="vfat" PARTLABEL="EFI system partition" PARTUUID="b81727be-ba90-5f8c-ab98-d3ec67778b7d"

8) Agregue lo siguiente al final del archivo /etc/grub.d/40_custom:

9) menuentry "Windows 7" {  
        insmod ntfs  
        set root='(hd0,1)'  
        search --no-floppy --fs-uuid --set <UUID>
        chainloader +1  
    }
    
9.1)menuentry "Windows 10" {
        insmod part_gpt
        insmod fat
        search --no-floppy --fs-uuid --set <boot_efi_uuid>
        chainloader /EFI/Microsoft/Boot/bootmgfw.efi
    }

10) Ejecute sudo update-grub y reinicie.
