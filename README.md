# Quartus II 13.1 Web Edition on Omarchy

How to install Quartus II 13.1 (Web Edition, free) on [Omarchy](https://omarchy.org)
(Arch Linux + Hyprland). Quartus 13.1 is from 2013, and its installer only officially
supports Red Hat/CentOS, so on Arch you have to install its dependencies yourself.

## 1. Enable multilib

Most dependencies are 32-bit (`lib32-*`) packages from the multilib repo.
Check whether it is already enabled:

```bash
grep -A1 '^\[multilib\]' /etc/pacman.conf
```

If that prints nothing, open `/etc/pacman.conf` as root, uncomment these two lines,
then run `sudo pacman -Syu`:

```ini
[multilib]
Include = /etc/pacman.d/mirrorlist
```

## 2. Install the dependencies

Omarchy ships with `yay`, which installs packages from both the official repos and the AUR:

```bash
yay -S --needed lib32-gcc-libs lib32-glibc lib32-libx11 lib32-libxext lib32-libxau lib32-libxdmcp lib32-expat lib32-freetype2 lib32-fontconfig lib32-libxtst lib32-libxi libpng12 lib32-libpng12 ncurses5-compat-libs lib32-libxft lib32-ncurses libudev0-shim
```

| Packages | Why |
|---|---|
| `lib32-gcc-libs` `lib32-glibc` `lib32-libx11` `lib32-libxext` `lib32-libxau` `lib32-libxdmcp` `lib32-expat` `lib32-freetype2` `lib32-fontconfig` | The 32-bit libraries that `setup.sh` checks for |
| `lib32-libxtst` `lib32-libxi` | X11 input libraries |
| `libpng12` `lib32-libpng12` (AUR) | Old libpng that the Quartus GUI links against |
| `ncurses5-compat-libs` (AUR) | Provides `libncurses.so.5` |
| `lib32-libxft` `lib32-ncurses` | **Optional:** only needed for ModelSim-Altera |
| `libudev0-shim` | **Optional:** only needed to program a board over USB-Blaster |

`--needed` skips anything that's already installed.

## 3. Run the installer

Download `Quartus-web-13.1.0.162-linux.tar` from Intel's FPGA download center, then:

```bash
mkdir ~/quartus-installer && tar -xf Quartus-web-13.1.0.162-linux.tar -C ~/quartus-installer
```

```bash
cd ~/quartus-installer && ./setup.sh
```

On Arch, `setup.sh` prints a warning that you need "32-bit compatibility libraries".
That's expected: you installed them in step 2. The default install directory is
`~/altera/13.1`.

The installer picks up the device files (`*.qdz`) and the ModelSim installer from
the `components/` folder next to `setup.sh`, so keep them together.

## 4. Launch it

```bash
~/altera/13.1/quartus/bin/quartus --64bit
```

Quartus 13.1 is an X11 app. On Omarchy it runs through XWayland, which is enabled by
default in Hyprland, so you don't need to set anything up.

## 5. (Optional) USB-Blaster without sudo

To program a board without running Quartus as root, add a udev rule:

```bash
echo 'SUBSYSTEM=="usb", ATTR{idVendor}=="09fb", ATTR{idProduct}=="6001", MODE="0666"' | sudo tee /etc/udev/rules.d/51-usbblaster.rules
```

```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```

Then unplug and replug the USB-Blaster.

## Troubleshooting

If Quartus fails with `error while loading shared libraries: libXXX.so`, find out
which library is missing:

```bash
ldd ~/altera/13.1/quartus/linux64/quartus | grep "not found"
```

Then search for the package that provides it with `pacman -F libXXX.so`
(run `sudo pacman -Fy` once first), or search the AUR.
