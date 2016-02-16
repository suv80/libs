centOS6.6最高版本的gcc也只到4.4.7版本,只好手动升级一下了。

***下载4.8.2源码***

下载依赖(gmp-4.3.2、mpfr-2.4.2、mpc-0.8.1)

```
wget ftp://gcc.gnu.org/pub/gcc/releases/gcc-4.8.2/gcc-4.8.2.tar.bz2
wget http://ftp.gnu.org/pub/gnu/gmp/gmp-4.3.2.tar.bz2
wget http://ftp.gnu.org/pub/gnu/mpfr/mpfr-2.4.2.tar.bz2
wget http://ftp.gnu.org/pub/gnu/mpc/mpc-1.0.1.tar.gz
```

***依次编译安装依赖***

```
cd /usr/gmp
./configure --prefix=/usr/local/gcc/gmp-4.3.2
sudo make
sudo make install
```

```
cd /usr/mpfr
./configure --prefix=/usr/local/gcc/mpfr-2.4.2 --with-gmp=/usr/local/gcc/gmp-4.3.2  
sudo make
sudo make install
```

```
cd /usr/mpc
../configure --prefix=/usr/local/gcc/mpc-0.8.1 --with-mpfr=/usr/local/gcc/mpfr-2.4.2 --with-gmp=/usr/local/gcc/gmp-4.3.2  
sudo make
sudo make install
```

***编译安装gcc4.8.2***

```
cd /usr/gcc-4.8.2
./configure --prefix=/usr/local/gcc --enable-threads=posix --disable-checking --enable-languages=c,c++ --disable-multilib --with-gmp=/usr/local/gcc/gmp-4.3.2 --with-mpfr=/usr/local/gcc/mpfr-2.4.2 --with-mpc=/usr/local/gcc/mpc-0.8.1
sudo make
sudo make install
```

***卸载旧版本***

```
yum remove -y gcc gcc-c++
updatedb
```

***链接新版本***

```
cd /usr/bin  
ln -s /usr/local/gcc/bin/gcc gcc  
ln -s /usr/local/gcc/bin/g++ g++  
```

***检查版本***

```
gcc -v
```

done

转载自：[http://www.centoscn.com/image-text/config/2015/0823/6041.html](http://www.centoscn.com/image-text/config/2015/0823/6041.html)