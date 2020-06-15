# How to make brightness hotkey work on AMD Renoir laptops

## Instructions for Manjaro

1. Clone desired Manjaro kernel (here v5.6)

```bash
$ git clone https://gitlab.manjaro.org/packages/core/linux56
```

2. Add brightness patch to PKGBUILD

[Patch](https://aur.archlinux.org/cgit/aur.git/plain/brightness.patch?h=linux-renoir-backlight), can be found in the repository.

Put the patch file in `linux56` folder.
Open `PKGBUILD`, and add `'brightness.patch'`on a new line at line 71, right before the ")".

```diff
# '0012-bootsplash.patch'
! '0013-bootsplash.patch'
+ 'brightness.patch')
# sha256sums=(
```

At line 226, add a new line and paste this:
```
msg "Add brightness patch"
patch -Np1 -i "${srcdir}/brightness.patch"
```

```diff
# patch -Np1 -i "${srcdir}/vfs-ino.patch"
+
+ msg "Add brightness patch"
+ patch -Np1 -i "${srcdir}/brightness.patch"
+
# if [ "${CARCH}" = "x86_64" ]; then
```

**Be careful to indentation!**

Save.

3. Retrieve SHA256 sums

```bash
$ makepkg -g
```

Makepkg will download build sources, like Linux kernel.

Copy lines from the output, from `sha256sums=(` to the very last `)`.
Paste them into PKGBUILD, replacing the previous `sha256sums` array (lines 73-115)

4. Apply patch

```bash
$ makepkg -o
```

5. Prepare modules

```bash
$ cd src/linux-5.6/
$ make modules_prepare
```

6. Compile GPU module

```bash
$ make -j$(nproc) M=drivers/gpu/ modules
```

7. Install GPU module into temporary folder (there will be SSL errors, ignore them)

``` bash
$ mkdir /tmp/staging/
$ make M=drivers/gpu/ INSTALL_MOD_PATH=/tmp/staging modules_install
```

8. Replace original driver

```bash
$ modprobe amdgpu
$ modprobe drm
# cp -r /tmp/staging/lib/modules/5.6.18-1-MANJARO/extra/* /lib/modules/5.6.18-1-MANJARO/kernel/drivers/gpu/
```

:warning:
**Your path may not be `5.6.18-1-MANJARO` as it depends on the Manjaro kernel version you chose**
:warning:

9. Reboot






