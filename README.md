# Instructions to install wine32 on KDE Neon

The KDE Neon maintainers forgot to build libdecor-0 and libsdl2 for i386, resulting in inability to install wine32, so we need to build them ourselves.

- Install docker
```bash
sudo apt install docker.io
```
- Pull the latest i386 ubuntu image
```bash
docker pull i386/ubuntu:bionic
```
- Create docker with i386 ubuntu
```bash
docker run -it --name i386_neon i386/ubuntu:bionic
```
- Update to latest packages
```bash
apt update
apt upgrade -y
```
- Since i386/ubuntu:bionic is the last i386 release, we need to first update it to focal
```bash
sed -i 's/bionic/focal/g' /etc/apt/sources.list
apt update
apt upgrade -y
```
- And then to jammy
```bash
sed -i 's/focal/jammy/g' /etc/apt/sources.list
apt update
apt upgrade -y
```
- Resolve conflicts
```bash
apt remove libapt-pkg5.0 -y
apt install gcc-10-base libsemanage-common -y
apt upgrade -y 
apt autoremove -y
```
- Now let's turn our ubuntu into a kde neon
```bash
apt install wget gnupg -y
wget -qO - 'https://archive.neon.kde.org/public.key' | apt-key add -
echo "deb https://archive.neon.kde.org/user/ jammy main" | tee /etc/apt/sources.list.d/neon.list
echo "deb-src https://archive.neon.kde.org/user/ jammy main" | tee -a /etc/apt/sources.list.d/neon.list
apt update
```
- Install the build dependencies
```bash
apt install build-essential devscripts -y
apt build-dep libdecor-0 -y
```
- Download the source for libdecor-0
```bash
mkdir build
cd build
apt source libdecor-0
```
- Build the libdecor-0 package
```bash
cd libdecor-0*/
dpkg-buildpackage -us -uc
cd ..
```
- Install the resulting package
```bash
apt install /build/libdecor-0-0_*.deb /build/libdecor-0-dev_*.deb -y
```
- Download the source for libsdl2
```bash
apt build-dep libsdl2 -y
apt source libsdl2
```
- Build the libsdl2 package
```bash
cd libsdl2*/
dpkg-buildpackage -us -uc
cd ..
```
- Copy the resulting packages out of the container
```bash
mkdir debs
cp *.deb debs/
exit
docker cp i386_neon:/build/debs .
```
- Install the packages
```bash
# depending on your desktop environment
sudo dpkg -i debs/libdecor-0-0_* debs/libdecor-0-plugin-1-cairo_* debs/libsdl2-2.0-0_*.deb
# or
sudo dpkg -i debs/libdecor-0-0_* debs/libdecor-0-plugin-1-gtk_* debs/libsdl2-2.0-0_*.deb
```
- Install wine32
```bash
sudo apt install wine32
```
- Remove the docker container and image
```bash
docker rm i386_neon
docker rmi i386/ubuntu:bionic
```
Done!
Or y'know, just download the debs from the releases page of this repo.
