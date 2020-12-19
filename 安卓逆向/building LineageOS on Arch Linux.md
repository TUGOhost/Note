These instructions are adapted from the official LineageOS [build for athene](https://wiki.lineageos.org/devices/athene/build).
You will need about 100GB of free disk space, or more if using `ccache`.

Install the AOSP [repo](https://source.android.com/setup/develop/repo) command and additional build tools.
`$ sudo pacman -S repo$ sudo pacman -S schedtool ninja`

Clone the [LineageOS/android](https://github.com/LineageOS/android) cm-14.1 repository.
`$ mkdir -p $HOME/workspaces/forks/LineageOS && cd $_$ repo init -u https://github.com/LineageOS/android.git -b cm-14.1$ repo sync [-j 4] -c`

If you do not have an existing lineage build with working proprietary drivers, then add the [TheMuppets/proprietary_vendor_motorola](https://github.com/TheMuppets/proprietary_vendor_motorola) repository to the `.repo/local_manifests/roomservice.xml` file.
`$ nano .repo/local_manifests/roomservice.xml<project name="TheMuppets/proprietary_vendor_motorola" path="vendor/motorola" remote="github" revision="cm-14.1" /><!--project name="TheMuppets/proprietary_vendor_qcom_binaries" path="vendor/qcom/binaries" remote="github" revision="cm-14.1" /-->`

Configure Python2.
`$ mkdir _bin$ ln -sf /usr/bin/python2 _bin/python$ ln -sf /usr/bin/python2-config _bin/python-config$ export PATH=$HOME/workspaces/forks/LineageOS/_bin:$PATH`

Configure the build environment.
`$ source build/envsetup.sh$ breakfast athene`

If you do have an existing lineage build with working proprietary drivers, then extract them from the file.
`$ mkdir _system_athene$ unzip /path/to/lineage-*-athene.zip system/* -d _system_athene$ cd device/motorola/athene && ./extract-files.sh ../../../_system_athene`

Configure system locale, system libraries, and Java settings.
`$ export LC_ALL=C`

```
# ln -sf $HOME/workspaces/forks/LineageOS/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot/usr/lib/libncurses.so.5 /usr/lib/libncurses.so.5# ln -sf $HOME/workspaces/forks/LineageOS/prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot/usr/lib/libtinfo.so.5 /usr/lib/libtinfo.so.5
# archlinux-java set java-8-openjdk$ export ANDROID_JACK_VM_ARGS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4G"
```

Build.
`$ croot$ brunch athene`

Install `out/target/product/athene/lineage-14.1-date-UNOFFICIAL-athene.zip` to your phone.
Backup `out/target/product/athene/recovery.img`.

Optionally, clean up.
`$ make clean [clobber]$ rm -rf _bin _system_athene# rm /usr/lib/libncurses.so.5 /usr/lib/libtinfo.so.5`

Troubleshooting.

Revert the LineageOS repository.
`$ repo forall -vc "git reset --hard"$ repo sync -c`





https://forum.xda-developers.com/t/guide-linux-manjaro-arch-how-to-set-up-android-build-environment.3300596/