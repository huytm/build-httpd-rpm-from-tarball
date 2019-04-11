# build-httpd-rpm-from-tarball
Build httpd rpm from tarball - Ví dụ httpd-2.4.28

## Requirement 
- Centos 7 

```sh
cat /etc/redhat-release
```
> CentOS Linux release 7.6.1810 (Core) 

#### Tắt firewall và SELinux

Bước này phục vụ trong quá trình test 

```sh
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
reboot
```

## Cài đặt các gói cần thiết

Trong bước này một số gói có thể cài đặt bằng yum, một số gói không thể cài bằng yum mình sẽ cài bằng rpm hoặc từ tarball.

```sh
yum install wget rpm-build autoconf zlib-devel libselinux-devel libuuid-devel pcre-devel openldap-devel lua-devel libxml2-devel openssl-devel apr-devel postgresql-devel mysql-devel sqlite-devel unixODBC-devel nss-devel expat-devel mailcap libtool doxygen gnutls -y
```

**freetds-devel**

```sh
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/nettle-2.7.1-8.el7.x86_64.rpm
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/trousers-0.3.14-2.el7.x86_64.rpm
wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/home:/matthewdva:/build:/EPEL:/el7/RHEL_7/x86_64/freetds-0.95.81-1.el7.x86_64.rpm
wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/home:/matthewdva:/build:/EPEL:/el7/RHEL_7/x86_64/freetds-devel-0.95.81-1.el7.x86_64.rpm

rpm -Uvh nettle-2.7.1-8.el7.x86_64.rpm
rpm -Uvh trousers-0.3.14-2.el7.x86_64.rpm
rpm -Uvh freetds-0.95.81-1.el7.x86_64.rpm
rpm -Uvh freetds-devel-0.95.81-1.el7.x86_64.rpm
```

**distcache-devel**

```sh
wget https://archive.fedoraproject.org/pub/archive/fedora/linux/releases/18/Fedora/source/SRPMS/d/distcache-1.4.5-23.src.rpm
rpmbuild --rebuild distcache-1.4.5-23.src.rpm
cd /root/rpmbuild/RPMS/x86_64/
rpm -Uvh distcache-*
```

**Berkeley DB-5.3.28**

```sh
yum install gcc-c++
wget http://anduin.linuxfromscratch.org/BLFS/bdb/db-5.3.28.tar.gz
tar -xzvf db-5.3.28.tar.gz

cd db-5.3.28
sed -i 's/\(__atomic_compare_exchange\)/\1_db/' src/dbinc/atomic.h
```

```
cd build_unix                        &&
../dist/configure --prefix=/usr      \
                  --enable-compat185 \
                  --enable-dbm       \
                  --disable-static   \
                  --enable-cxx       &&
make
```

```
wget http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-4.8.30-13.el7.x86_64.rpm
wget http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/l/libdb4-devel-4.8.30-13.el7.x86_64.rpm

rpm -Uvh libdb4-4.8.30-13.el7.x86_64.rpm
rpm -Uvh libdb4-devel-4.8.30-13.el7.x86_64.rpm
```

**apr**

```sh
wget http://mirror.downloadvn.com/apache//apr/apr-1.7.0.tar.bz2
wget http://mirror.downloadvn.com/apache//apr/apr-util-1.6.1.tar.bz2
```

```sh
rpmbuild -tb apr-1.7.0.tar.bz2
cd /root/rpmbuild/RPMS/x86_64/
rpm -Uvh apr-*
```

```sh
 cd - 
rpmbuild -tb apr-util-1.6.1.tar.bz2
cd /root/rpmbuild/RPMS/x86_64/
rpm -Uvh apr-util-*
```

## Build httpd rpm 

Tạo một folder chứa các file thực thi

```sh
mkdir httpd-build
cd httpd-build/
wget http://archive.apache.org/dist/httpd/httpd-2.4.28.tar.bz2
```

Build file

```
rpmbuild -tb httpd-2.4.28.tar.bz2
```

Cài đặt http từ rpm file 

```
cd /root/rpmbuild/RPMS/x86_64/
rpm -Uvh httpd-*
```

Start httpd

```
systemctl start httpd
```

