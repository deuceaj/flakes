## How to install?
Minimal installation (including wm, no tmpfs as root), go to [here](https://github.com/Ruixi-rebirth/flakes/tree/minimal)

## How to install ?(root on tmpfs)
0. Prepare a 64-bit nixos [minimal iso image](https://channels.nixos.org/nixos-22.11/latest-nixos-minimal-x86_64-linux.iso) and burn it, then enter the live system. Suppose I have divided two partitions `/dev/nvme0n1p1` `/dev/nvme0n1p3`
1. Format the partition 
```bash
  mkfs.fat -F 32 /dev/nvme0n1p1 
  mkfs.ext4 /dev/nvme0n1p3
```
2. Mount 
```bash
  mount -t tmpfs none /mnt 
  mkdir -p /mnt/{boot,nix,etc/nixos}
  mount /dev/nvme0n1p3 /mnt/nix
  mount /dev/nvme0n1p1 /mnt/boot 
  mkdir -p /mnt/nix/persist/etc/nixos
  mount -o bind /mnt/nix/persist/etc/nixos /mnt/etc/nixos
```
3. Generate a basic configuration 
```bash
  nixos-generate-config --root /mnt
```
4. Clone the repository locally 
```bash
nix-shell -p git
git clone  https://github.com/Ruixi-rebirth/flakes.git /mnt/etc/nixos/Flakes 
cd /mnt/etc/nixos/Flakes/
nix develop --extra-experimental-features nix-command --extra-experimental-features flakes 
```
5. Copy `hardware-configuration.nix` from /mnt/etc/nixos to /mnt/etc/nixos/Flakes/hosts/laptop/hardware-configuration.nix 
```bash 
cp /mnt/etc/nixos/hardware-configuration.nix /mnt/etc/nixos/Flakes/hosts/laptop/hardware-configuration.nix
```
6. Modify the overwritten `hardware-configuration.nix` 
```bash
nvim /mnt/etc/nixos/Flakes/hosts/laptop/hardware-configuration.nix
```
```nix
...
#This is just an example
#Please refer to `https://elis.nu/blog/2020/05/nixos-tmpfs-as-root/#step-4-1-configure-disks`

  fileSystems."/" =
    { device = "none";
      fsType = "tmpfs";
      options = [ "defaults" "size=8G" "mode=755"  ];
    };

  fileSystems."/nix" =
    { device = "/dev/disk/by-uuid/49e24551-c0e0-48ed-833d-da8289d79cdd";
      fsType = "ext4";
    };

  fileSystems."/boot" =
    { device = "/dev/disk/by-uuid/3C0D-7D32";
      fsType = "vfat";
    };

  fileSystems."/etc/nixos" =
    { device = "/nix/persist/etc/nixos";
      fsType = "none";
      options = [ "bind" ];
    };
...
```
7. remove '/mnt/etc/nixos/Flakes/.git' 
```bash 
rm -rf .git
```
8. Username modification: edit `/mnt/etc/nixos/Flakes/flake.nix` to modify **user** variable, hostname modification: edit `/mnt/etc/nixos/Flakes/hosts/system.nix` to modify* The **hostName** value in the **networking** property group

9. Use the hash password generated by the `mkpasswd {PASSWORD} -m sha-512` command to replace the value of `users.users.<name>.hashedPassword` in `/mnt/etc/nixos/Flakes/hosts/laptop/wayland/default.nix` ( there is two place needs to be displace )

10. Select Window Manager
> Wayland: uncomment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L17) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L34), comment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L18) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L35) 

Hyprland: uncomment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/default.nix#L12) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/home.nix#L6), comment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/default.nix#L11) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/home.nix#L5)

Sway: uncomment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/default.nix#L11) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/home.nix#L5), comment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/default.nix#L12) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/home.nix#L6)

> Xorg:  uncomment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L18) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L35), comment [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L17) and [this line](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/default.nix#L34)

Bspwm: default 

11. Select a theme 
> Wayland 

[here](https://github.com/Ruixi-rebirth/flakes/blob/main/hosts/laptop/wayland/home.nix#L11-L13) choose the one you want

> Xorg 

nord: default

12. Perform install
```bash
nixos-install --no-root-passwd --flake .#laptop
```
13. Reboot 
```bash
reboot
```
14. Enjoy it!
