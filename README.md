### Howto build latest Openvpn for mips64

#### Download debian wheezy qcow image and kernel
https://people.debian.org/~aurel32/qemu/mips/debian_wheezy_mips_standard.qcow2  
https://people.debian.org/~aurel32/qemu/mips/vmlinux-3.2.0-4-5kc-malta

#### Install qemu-system-mips64
~~~
sudo apt install qemu-system-mips 
~~~

#### Start debian wheezy vm inside screen
~~~
screen -A -S qemu
qemu-system-mips64 -nographic -M malta -kernel vmlinux-3.2.0-4-5kc-malta -hda debian_wheezy_mips_standard.qcow2 -append "root=/dev/sda1 console=tty0" -net nic -net user,hostfwd=tcp::1810-:22
~~~

#### Access debian wheezy vm
*password is root*
~~~
ssh root@localhost -p 1810
~~~

#### Setup QEMU Build Environment
Compiling requires you to install build-essentials. Debian wheezy has moved to the debian archive so you must update the sources.list Note that you will not be able to authenticate the pacakges from the debian archive due to an expired certificate.
~~~
mv /etc/apt/sources.list /etc/apt/sources_old.list
echo "deb http://archive.debian.org/debian wheezy main" > /etc/apt/sources.list
echo "deb-src http://archive.debian.org/debian wheezy main" >> /etc/apt/sources.list
apt-get update
apt-get install build-essential
~~~

#### Create directories
~~~
mkdir ~/src
mkdir ~/build
cd ~/src
~~~

#### Download latest Openssl source and compile it
~~~
wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz
tar -xvf openssl-1.1.1k.tar.gz
cd openssl-1.1.1k
./Configure gcc -static -no-shared --prefix=/root/build
make && make install
~~~

#### Download latest LZ4 source and compile it
~~~
wget https://github.com/lz4/lz4/archive/v1.9.2.tar.gz
tar -xvf v1.9.2.tar.gz
cd lz4-1.9.2/
make && PREFIX=/root/build make install
~~~

#### Download latest LZO source and compile it
~~~
wget https://www.oberhumer.com/opensource/lzo/download/lzo-2.10.tar.gz
tar -xvf lzo-2.10.tar.gz
cd lzo-2.10
./configure --prefix=/root/build --enable-static --target=mips64-linux-gnueabi --host=mips64-linux-gnueabi
make && make install
~~~

#### Download latest Openvpn source and compile it.
~~~
wget https://swupdate.openvpn.org/community/releases/openvpn-2.5.1.tar.gz
tar -xvf openvpn-2.5.1.tar.gz
cd openvpn-2.5.1
./configure --prefix=/root/build --enable-static --disable-shared --disable-debug --disable-plugins \
  OPENSSL_CFLAGS="-I/root/build/include" OPENSSL_LIBS="-L/root/build/lib -lssl -lcrypto" LZO_CFLAGS="-I/root/build/include" \
  LZO_LIBS="-L/root/build/lib -llzo2" LZ4_CFLAGS="-I/root/build/include" LZ4_LIBS="-L/root/build/lib -llz4" \
  IFCONFIG=/sbin/ifconfig ROUTE=/sbin/route NETSTAT=/bin/netstat IPROUTE=/sbin/ip --enable-iproute2
make LIBS="-all-static"
make install
~~~

openvpn client will be inside ~/build/sbin directory. You can shrink the size of the binary (by around half) if you strip debugging symbols:
~~~
strip /root/build/sbin/openvpn
~~~
