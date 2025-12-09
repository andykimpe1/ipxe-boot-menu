# ipxe-boot-menu
my ipxe boot menu
build example for vmware workstation 17 on ubuntu

```
cd
sudo apt -y install gcc binutils make perl liblzma-dev mtools genisoimage syslinux git dnsmasq wget
sudo rm -rf /tmp/tftp /tftp
mkdir /tmp/tftp
cd /tmp/tftp
wget https://mirrors.edge.kernel.org/pub/linux/utils/boot/syslinux/syslinux-6.03.zip
unzip syslinux-6.03.zip
rm -f syslinux-6.03.zip
apt-get download shim.signed
dpkg -x shim-signed_*.deb shim
rm -f shim-signed_*.deb
apt-get download grub-efi-amd64-signed
dpkg -x grub-efi-amd64-signed_*.deb grub
rm -f grub-efi-amd64-signed_*.deb
sudo rm -rf /tftp
sudo mkdir /tftp
sudo mkdir /tftp/bios
sudo mkdir /tftp/boot
sudo mkdir /tftp/grub
#configure /etc/dnsmasq.conf example for work with vmware worstation 17 nat network vmnet8
cat > dnsmasq.conf <<EOF
#Interface information 
#--use ip addr to see the name of the interface on your system
interface=vmnet8,lo
bind-interfaces
domain=c-nergy.local

#--------------------------
#DHCP Settings
#--------------------------
#-- Set dhcp scope
dhcp-range=192.168.235.100,192.168.235.200,255.255.255.0,24h

#-- Set gateway option
dhcp-option=3,192.168.235.2

#-- Set DNS server option
dhcp-option=6,192.168.235.2

#-- dns Forwarder info
server=8.8.8.8

#----------------------#
# Specify TFTP Options #
#----------------------#

#--location of the pxeboot file
dhcp-boot=/bios/pxelinux.0,pxeserver,192.168.235.1

#--enable tftp service
enable-tftp

#-- Root folder for tftp
tftp-root=/tftp

#--Detect architecture and send the correct bootloader file
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,grub/bootx64.efi
EOF
sudo rm -f /etc/dnsmasq.conf
sudo cp dnsmasq.conf /etc/
sudo systemctl restart dnsmasq
sudo cp /tmp/tftp/bios/com32/elflink/ldlinux/ldlinux.c32  /tftp/bios
sudo cp /tmp/tftp/bios/com32/libutil/libutil.c32  /tftp/bios  
sudo cp /tmp/tftp/bios/com32/menu/menu.c32  /tftp/bios
sudo cp /tmp/tftp/bios/com32/menu/vesamenu.c32  /tftp/bios
sudo cp /tmp/tftp/bios/com32/chain/chain.c32  /tftp/bios
sudo cp /tmp/tftp/bios/com32/lib/libcom32.c32  /tftp/bios
sudo cp /tmp/tftp/bios/core/pxelinux.0  /tftp/bios
sudo cp /tmp/tftp/bios/core/lpxelinux.0  /tftp/bios




sudo cp /tmp/tftp/grub/usr/lib/grub/x86_64-efi-signed/grubnetx64.efi.signed  /tftp/grubx64.efi
sudo cp /tmp/tftp/shim/usr/lib/shim/shimx64.efi  /tftp/grub/bootx64.efi
sudo mkdir /tftp/bios/pxelinux.cfg
cat > default <<EOF
DEFAULT menu.c32
MENU TITLE bios selinux menu
PROMPT 0 
TIMEOUT 600

MENU COLOR TABMSG  37;40  #ffffffff #00000000
MENU COLOR TITLE   37;40  #ffffffff #00000000 
MENU COLOR SEL      7     #ffffffff #00000000
MENU COLOR UNSEL    37;40 #ffffffff #00000000
MENU COLOR BORDER   37;40 #ffffffff #00000000

LABEL Demarer sur le disque dur
    com32 chain.c32
    append hd0

LABEL Demarer ipxe pour installer votre serveur
    kernel ipxe.lkrn
EOF
sudo cp default /tftp/bios/pxelinux.cfg/
cat > grub.cfg <<EOF
if loadfont /grub/font.pf2 ; then
set gfxmode=auto
insmod efi_gop
insmod efi_uga
insmod gfxterm
insmod chain
insmod ntfs
terminal_output gfxterm
fi

set menu_color_normal=white/black
set menu_color_highlight=black/light-gray
set timeout=60

menuentry "demarer sur le disque dur" {
set root=(hd0,1)
chainloader +1
}

menuentry "Demarer ipxe pour installer votre serveur" {
chainloader ipxe.efi
}
EOF
sudo cp grub.cfg /tftp/grub/
git clone https://github.com/ipxe/ipxe.git
cd ipxe/src
sed -i "s|#undef	DOWNLOAD_PROTO_HTTPS|#define	DOWNLOAD_PROTO_HTTPS|" config/general.h
cat > boot.ipxe <<EOF
#!ipxe
dhcp
chain https://raw.githubusercontent.com/andykimpe1/ipxe-boot-menu/refs/heads/main/menu.ipxe
EOF
make clean
make bin/ipxe.lkrn EMBED=boot.ipxe
sudo rm -f /tftp/bios/pxelinux.cfg/ipxe.lkrn && sudo cp bin/ipxe.lkrn /tftp/bios/pxelinux.cfg/
sudo rm -f /tftp/bios/ipxe.lkrn && sudo cp bin/ipxe.lkrn /tftp/bios/
sudo rm -f /tftp/boot/ipxe.lkrn && sudo cp bin/ipxe.lkrn /tftp/boot/
sudo rm -f /tftp/ipxe.lkrn && sudo cp bin/ipxe.lkrn /tftp/
make clean
make bin-x86_64-efi/ipxe.efi EMBED=boot.ipxe
sudo rm -f /tftp/bios/pxelinux.cfg/ipxe.efi && sudo cp bin-x86_64-efi/ipxe.efi /tftp/bios/pxelinux.cfg/
sudo rm -f /tftp/bios/ipxe.efi && sudo cp bin-x86_64-efi/ipxe.efi /tftp/bios/
sudo rm -f /tftp/boot/ipxe.efi && sudo cp bin-x86_64-efi/ipxe.efi /tftp/boot/
sudo rm -f /tftp/ipxe.efi && sudo cp bin-x86_64-efi/ipxe.efi /tftp/
```
