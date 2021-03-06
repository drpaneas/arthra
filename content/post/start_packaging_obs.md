+++
date = "2015-09-17"
title = "How to RPM package your script using OBS"
description = "A simple guide for beginners getting familiar with obs"
keywords = ["panos georgiadis", "rpm packaging", "obs", "beginner", "how to"]
menu = "How-To"
+++


The purpose of this tutorial is to teach you the very basics of OpenBuild
service and RPM packaging using a dummy hello world script. Suppose that you
have a script that you want to package it and share it with your friends. This
is how to do it:


### Write the Script

Make a folder called `building` or whatever you prefer and inside of it, create
your bash script `hello.sh`:

```bash
pgeorgiadis@panos:~> cd building/
pgeorgiadis@panos:~/building> vi hello.sh
```

`hello.sh` source code:
```bash
#!/bin/bash
echo "Hello World"
```

### Create the spec file

After writing the script, then it's time to create the spec file. This is the
file that contains *all* the necessary information about building the package.
The trick here is that there's an already made template for the spec file; all
you have to do is just create the `hello.spec` file.

The default template is like that: 

```spec
Name:           hello
Version:
Release:
License:
Summary:
Url:
Group:
Source:
Patch:
BuildRequires:
PreReq:
Provides:
BuildRoot:      %{_tmppath}/%{name}-%{version}-build

%description

%prep
%setup -q

%build
%configure
make %{?_smp_mflags}

%install
make install DESTDIR=%{buildroot} %{?_smp_mflags}

%post

%postun

%files
%defattr(-,root,root)
%doc ChangeLog README COPYING
```

and modify it accordingly into this:

```spec
Name:           hello
Version:        1.1
Release:        0
License:        GPL-2.0
Summary:        TODO
Url:            TODO
Group:          TODO
Source:         hello.sh
#Patch:
#BuildRequires:
%if 0%{?suse_version}
PreReq:         nfs-client
%endif
#Provides:
BuildRoot:      %{_tmppath}/%{name}-%{version}-build

%description
TODO

%prep
#setup -q

%build
#configure
#make %{?_smp_mflags}

%install
#make install DESTDIR=%{buildroot} %{?_smp_mflags}
install -D -m 755 %{S:0} %{buildroot}%{_bindir}/hello.sh

%files
%defattr(-,root,root)
%{_bindir}/hello.sh
```

If you are interested to know what's all these macros, feel free to google
theand make sure that they are supported by your system. For example what's
`bindir` ?

```bash
pgeorgiadis@panos:~> rpm --eval "%_bindir"
/usr/bin
```


### Create project on OBS

Go to [Open Build Service](https://build.opensuse.org/) and log in. Then click
on your `Home Project` (if you don't have one, then create it). Over there, you
can create as many packages as you want. In our case, we are going to create
the dummy `hello` package, so we click on `Create package` and populate the
required fields respectively:

1. name : `hello`
2. title: `hello`
3. description: `dummy package`

Next, we leave the tickbox `Disable build results publishing.` deactivated and
finally click on `Save changes` button.

#### Connect our remote project with our local files

Normally, we should have created the project first on OBS, then check-it out
locally (even empty) and then add files in it. Now, we are going to do it
backwards, but this is not going to be a problem.

In order to be able to interactive with OBS via command line we have to provide
the required authetication (username, password) and API that we are going to
use. Before that, let's edit the `.bashrc` file and create an alias:

```bash
vim ~/.bashrc
```

create an alias to ioscrc
```bash
test -s ~/.alias && . ~/.alias || true
alias obs='osc --apiurl=https://api.opensuse.org'
```

So, instead of having to type the whole `osc --apiurl=https://api.opensuse.org`
command, you can just type `obs`, which is faster and easy to remember. Last
but not least, don't forget to do `source ~/.bashrc` in order for the changes
to take immediate effect.

Next, we proceed by defining our login username & password, used for the OBS.
We do that, by editing the `~/.oscrc` file.

```bash
[https://api.opensuse.org]
user=<username>
pass=<password>
```

Finally, you should be able to checkout (download) your project from OBS into
our local hard disk. To do that, you need to know the URL of your project. In
my case, that is
`https://build.opensuse.org/package/show/home:pgeorgiadis/hello`.

```bash
pgeorgiadis@panos:~/building> obs co home:pgeorgiadis/hello
A    home:pgeorgiadis
A    home:pgeorgiadis/hello
At revision None.
```

Now, let's put them all together, like that:
`home:pgeorgiadis/hello/<myfiles>`

```bash
pgeorgiadis@panos:~/building> ls
hello.sh  hello.spec  home:pgeorgiadis
pgeorgiadis@panos:~/building> mv hello.s* home:pgeorgiadis/
pgeorgiadis@panos:~/building> cd home:pgeorgiadis/
pgeorgiadis@panos:~/building/home:pgeorgiadis> mv hello.s* hello
pgeorgiadis@panos:~/building/home:pgeorgiadis> ls
hello
pgeorgiadis@panos:~/building/home:pgeorgiadis> cd hello/
```

### Build the RPM Package

First let's see what Repositories I have enabled at remote OBS. This can be
seen from the web-interface

1. click to `Home Project`
2. Go to tab `Repositories`

or this can also be done in console:

```
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> obs repos
openSUSE_13.2  i586
openSUSE_13.2  x86_64
SLE_12         x86_64
```

If I would like to build it locally in my machine, and if I would like to build
it only for openSUSE_13.2 then I would type:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc build openSUSE_13.2
```

Short while the build service is going to ask you to put your faith on it.
Please do by selecting `1 - always`:

```bash
The build root needs packages from project 'openSUSE:13.2'.
Note that malicious packages can compromise the build result or even your
system.
Would you like to ...
0 - quit (default)
1 - always trust packages from 'openSUSE:13.2'
2 - trust packages just this time
? 1
```

This actually means that is going to write a new line into your `.oscrc` file
related to openSUSE:13.2 and trust.

```bash
adding 'openSUSE:13.2' to ~/.oscrc: ['https://api.opensuse.org']['trusted_prj']
100.0% cache miss. 0/98 dependencies cached.
```

Just in case you get this error:

```bash
packagecachedir is not writable for you?
[Errno 13] Permission denied: '/var/tmp/osbuild-packagecache/openSUSE:13.2'
```

this means that you have previously used OBS as a different user, and you don't
have the permission to write into `/var/tmp/osbuild-packagecache/` directory.
In my case I used OBS before as `slenkins` user, and I left some leftovers...

```bash
pgeorgiadis@panos:/var/tmp/osbuild-packagecache> ls -l
total 0
drwxr-xr-x 1 slenkins users 26 Sep 15 15:49 Devel:SLEnkins:testsuites
drwxr-xr-x 1 slenkins users 30 Sep 15 15:49 SUSE:SLE-12:GA
```

so it's completely normal to see that message in that case. All I had to do is
either remove it or rename it (in case I don't want to delete it).

```bash
pgeorgiadis@panos:/var/tmp/osbuild-packagecache> cd ..
pgeorgiadis@panos:/var/tmp> sudo mv osbuild-packagecache/ osbuild-packagecache2
```

and now try again to build the package locally. It should work:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc build openSUSE_13.2
Building hello.spec for openSUSE_13.2/x86_64
Getting buildinfo from server and store to
/suse/pgeorgiadis/building/home:pgeorgiadis/hello/.osc/_buildinfo-openSUSE_13.2-x86_64.xml
Getting buildconfig from server and store to
/suse/pgeorgiadis/building/home:pgeorgiadis/hello/.osc/_buildconfig-openSUSE_13.2-x86_64
Updating cache of required packages
100.0% cache miss. 0/98 dependencies cached.

(openSUSE:13.2) aaa_base-13.2+git20140911.61c1681-1. 100%
|==========================| 121 kB    00:00     
(openSUSE:13.2) attr-2.4.47-4.1.2.x86_64.rpm         100%
|==========================|  50 kB    00:00     
(openSUSE:13.2) bash-4.2-75.3.1.x86_64.rpm           100%
|==========================| 344 kB    00:00     
(openSUSE:13.2) coreutils-8.23-2.3.1.x86_64.rpm      100%
|==========================| 1.2 MB    00:10     
(openSUSE:13.2) diffutils-3.3-6.1.2.x86_64.rpm       100%
|==========================| 199 kB    00:00     
(openSUSE:13.2) filesystem-13.2-4.3.1.x86_64.rpm     100%
|==========================|  69 kB    00:00     
(openSUSE:13.2) fillup-1.42-271.1.2.x86_64.rpm       100%
|==========================|  20 kB    00:00     
(openSUSE:13.2) glibc-2.19-16.2.5.x86_64.rpm         100%
|==========================| 1.6 MB    00:07     
(openSUSE:13.2) grep-2.20-2.1.2.x86_64.rpm           100%
|==========================| 338 kB    00:00     
(openSUSE:13.2) libbz2-1-1.0.6-29.2.7.x86_64.rpm     100%
|==========================|  34 kB    00:00     
(openSUSE:13.2) libgcc_s1-4.8.3+r212056-2.2.4.x86_64 100%
|==========================|  49 kB    00:00     
(openSUSE:13.2) m4-1.4.17-2.1.2.x86_64.rpm           100%
|==========================| 237 kB    00:00     
(openSUSE:13.2) libncurses5-5.9-52.2.3.x86_64.rpm    100%
|==========================| 353 kB    00:04     
(openSUSE:13.2) pam-1.1.8-11.1.2.x86_64.rpm          100%
|==========================| 400 kB    00:01     
(openSUSE:13.2) permissions-2014.08.26.1452-1.2.x86_ 100%
|==========================|  31 kB    00:00     
(openSUSE:13.2) libreadline6-6.2-75.3.1.x86_64.rpm   100%
|==========================| 130 kB    00:00     
(openSUSE:13.2) rpm-4.11.3-1.2.x86_64.rpm            100%
|==========================| 1.6 MB    00:03     
(openSUSE:13.2) sed-4.2.2-5.1.2.x86_64.rpm           100%
|==========================|  99 kB    00:00     
(openSUSE:13.2) tar-1.28-2.2.2.x86_64.rpm            100%
|==========================| 510 kB    00:00     
(openSUSE:13.2) libz1-1.2.8-5.1.2.x86_64.rpm         100%
|==========================|  51 kB    00:00     
(openSUSE:13.2) libselinux1-2.3-2.2.5.x86_64.rpm     100%
|==========================|  64 kB    00:00     
(openSUSE:13.2) liblzma5-5.0.7-1.1.x86_64.rpm        100%
|==========================|  96 kB    00:00     
(openSUSE:13.2) libcap2-2.22-13.1.8.x86_64.rpm       100%
|==========================|  12 kB    00:00     
(openSUSE:13.2) libacl1-2.2.52-5.1.2.x86_64.rpm      100%
|==========================|  19 kB    00:00     
(openSUSE:13.2) libattr1-2.4.47-4.1.2.x86_64.rpm     100%
|==========================|  21 kB    00:00     
(openSUSE:13.2) libpopt0-1.16-27.1.2.x86_64.rpm      100%
|==========================|  49 kB    00:00     
(openSUSE:13.2) libelf1-0.158-4.2.6.x86_64.rpm       100%
|==========================|  46 kB    00:00     
(openSUSE:13.2) liblua5_1-5.1.5-10.2.2.x86_64.rpm    100%
|==========================|  78 kB    00:00     
(openSUSE:13.2) libpcre1-8.35-3.2.3.x86_64.rpm       100%
|==========================| 244 kB    00:00     
(openSUSE:13.2) util-linux-2.25.1-2.4.x86_64.rpm     100%
|==========================| 938 kB    00:01     
(openSUSE:13.2) libmount1-2.25.1-2.4.x86_64.rpm      100%
|==========================| 147 kB    00:00     
(openSUSE:13.2) perl-base-5.20.1-1.3.x86_64.rpm      100%
|==========================| 1.1 MB    00:01     
(openSUSE:13.2) libdb-4_8-4.8.30-29.1.2.x86_64.rpm   100%
|==========================| 668 kB    00:00     
(openSUSE:13.2) libsepol1-2.3-2.1.8.x86_64.rpm       100%
|==========================| 106 kB    00:00     
(openSUSE:13.2) libblkid1-2.25.1-2.4.x86_64.rpm      100%
|==========================| 139 kB    00:00     
(openSUSE:13.2) libuuid1-2.25.1-2.4.x86_64.rpm       100%
|==========================|  52 kB    00:00     
(openSUSE:13.2) net-tools-1.60-765.1.2.x86_64.rpm    100%
|==========================| 222 kB    00:00     
(openSUSE:13.2) libsmartcols1-2.25.1-2.4.x86_64.rpm  100%
|==========================|  96 kB    00:00     
(openSUSE:13.2) kernel-obs-build-3.16.6-2.1.x86_64.r 100%
|==========================|  20 MB    00:05     
(openSUSE:13.2) rpm-build-4.11.3-1.2.x86_64.rpm      100%
|==========================|  33 kB    00:00     
(openSUSE:13.2) glibc-devel-2.19-16.2.5.x86_64.rpm   100%
|==========================| 637 kB    00:01     
(openSUSE:13.2) xz-5.0.7-1.1.x86_64.rpm              100%
|==========================| 123 kB    00:00     
(openSUSE:13.2) cpio-2.11-29.2.3.x86_64.rpm          100%
|==========================|  83 kB    00:00     
(openSUSE:13.2) gcc-4.8-7.1.2.x86_64.rpm             100%
|==========================| 5.7 kB    00:00     
(openSUSE:13.2) file-5.19-3.1.2.x86_64.rpm           100%
|==========================|  45 kB    00:00     
(openSUSE:13.2) findutils-4.5.14-2.1.2.x86_64.rpm    100%
|==========================| 280 kB    00:00     
(openSUSE:13.2) bzip2-1.0.6-29.2.7.x86_64.rpm        100%
|==========================|  36 kB    00:00     
(openSUSE:13.2) make-4.0-2.2.3.x86_64.rpm            100%
|==========================| 381 kB    00:00     
(openSUSE:13.2) gawk-4.1.1-3.1.2.x86_64.rpm          100%
|==========================| 1.0 MB    00:02     
(openSUSE:13.2) binutils-2.24-6.1.7.x86_64.rpm       100%
|==========================| 4.3 MB    00:01     
(openSUSE:13.2) gzip-1.6-8.1.8.x86_64.rpm            100%
|==========================| 113 kB    00:00     
(openSUSE:13.2) glibc-locale-2.19-16.2.5.x86_64.rpm  100%
|==========================| 6.0 MB    00:00     
(openSUSE:13.2) which-2.20-4.1.2.x86_64.rpm          100%
|==========================|  31 kB    00:00     
(openSUSE:13.2) patch-2.7.1-7.1.2.x86_64.rpm         100%
|==========================|  88 kB    00:00     
(openSUSE:13.2) systemd-rpm-macros-2-8.1.2.noarch.rp 100%
|==========================| 4.7 kB    00:00     
(openSUSE:13.2) libgmp10-5.1.3-3.1.2.x86_64.rpm      100%
|==========================| 237 kB    00:00     
(openSUSE:13.2) linux-glibc-devel-3.16-2.1.7.noarch. 100%
|==========================| 945 kB    00:01     
(openSUSE:13.2) libcap-ng0-0.7.4-3.1.2.x86_64.rpm    100%
|==========================|  22 kB    00:00     
(openSUSE:13.2) libaudit1-2.4-1.3.x86_64.rpm         100%
|==========================|  56 kB    00:00     
(openSUSE:13.2) libutempter0-1.1.6-5.2.3.x86_64.rpm  100%
|==========================|  19 kB    00:00     
(openSUSE:13.2) insserv-compat-0.1-12.2.2.noarch.rpm 100%
|==========================| 9.0 kB    00:00     
(openSUSE:13.2) gcc48-4.8.3+r212056-2.2.4.x86_64.rpm 100%
|==========================| 8.5 MB    00:13     
(openSUSE:13.2) cpp-4.8-7.1.2.x86_64.rpm             100%
|==========================| 4.8 kB    00:00     
(openSUSE:13.2) libmagic1-5.19-3.1.2.x86_64.rpm      100%
|==========================|  65 kB    00:00     
(openSUSE:13.2) update-alternatives-1.16.10-8.1.2.x8 100%
|==========================|  35 kB    00:00     
(openSUSE:13.2) libsemanage1-2.3-2.1.8.x86_64.rpm    100%
|==========================|  72 kB    00:00     
(openSUSE:13.2) info-4.13a-38.1.2.x86_64.rpm         100%
|==========================| 134 kB    00:00     
(openSUSE:13.2) terminfo-base-5.9-52.2.3.x86_64.rpm  100%
|==========================| 197 kB    00:00     
(openSUSE:13.2) libcrack2-2.9.0-4.1.3.x86_64.rpm     100%
|==========================|  18 kB    00:00     
(openSUSE:13.2) libmpfr4-3.1.2-5.1.2.x86_64.rpm      100%
|==========================| 151 kB    00:00     
(openSUSE:13.2) libcloog-isl4-0.18.1-2.1.2.x86_64.rp 100%
|==========================|  58 kB    00:00     
(openSUSE:13.2) libisl10-0.12.2-2.1.2.x86_64.rpm     100%
|==========================| 414 kB    00:02     
(openSUSE:13.2) libmpc3-1.0.2-2.1.2.x86_64.rpm       100%
|==========================|  42 kB    00:00     
(openSUSE:13.2) cpp48-4.8.3+r212056-2.2.4.x86_64.rpm 100%
|==========================| 4.9 MB    00:01     
(openSUSE:13.2) libasan0-4.8.3+r212056-2.2.4.x86_64. 100%
|==========================|  74 kB    00:00     
(openSUSE:13.2) file-magic-5.19-3.1.2.x86_64.rpm     100%
|==========================| 317 kB    00:00     
(openSUSE:13.2) libustr-1_0-1-1.0.4-33.1.2.x86_64.rp 100%
|==========================|  80 kB    00:00     
(openSUSE:13.2) libzio1-1.00-10.1.2.x86_64.rpm       100%
|==========================|  13 kB    00:00     
(openSUSE:13.2) cracklib-2.9.0-4.1.3.x86_64.rpm      100%
|==========================|  49 kB    00:00     
(openSUSE:13.2) gettext-tools-mini-0.19.2-2.1.3.x86_ 100%
|==========================| 1.8 MB    00:00     
(openSUSE:13.2) gettext-runtime-mini-0.19.2-2.1.3.x8 100%
|==========================| 722 kB    00:01     
(openSUSE:13.2) libstdc++6-4.8.3+r212056-2.2.4.x86_6 100%
|==========================| 247 kB    00:00     
(openSUSE:13.2) libatomic1-4.8.3+r212056-2.2.4.x86_6 100%
|==========================|  21 kB    00:00     
(openSUSE:13.2) libgomp1-4.8.3+r212056-2.2.4.x86_64. 100%
|==========================|  36 kB    00:00     
(openSUSE:13.2) libitm1-4.8.3+r212056-2.2.4.x86_64.r 100%
|==========================|  41 kB    00:00     
(openSUSE:13.2) libtsan0-4.8.3+r212056-2.2.4.x86_64. 100%
|==========================| 107 kB    00:08     
(openSUSE:13.2) pam-modules-12.1-24.1.2.x86_64.rpm   100%
|==========================| 100 kB    00:00     
(openSUSE:13.2) perl-5.20.1-1.3.x86_64.rpm           100%
|==========================| 6.1 MB    00:02     
(openSUSE:13.2) build-mkbaselibs-20140424-2.1.3.noar 100%
|==========================|  26 kB    00:00     
(openSUSE:13.2) brp-check-suse-13.2+git20140727.d76b 100%
|==========================|  73 kB    00:00     
(openSUSE:13.2) post-build-checks-13.2+git20140318.f 100%
|==========================|  44 kB    00:00     
(openSUSE:13.2) rpmlint-Factory-1.0-88.1.2.noarch.rp 100%
|==========================|  13 kB    00:00     
(openSUSE:13.2) build-compare-2014.07.15-4.1.5.noarc 100%
|==========================|  25 kB    00:00     
(openSUSE:13.2) brp-extract-appdata-2012.02.13-22.1. 100%
|==========================| 8.0 kB    00:00     
(openSUSE:13.2) libgdbm4-1.11-3.1.2.x86_64.rpm       100%
|==========================|  69 kB    00:00     
(openSUSE:13.2) aaa_base-malloccheck-13.2+git2014091 100%
|==========================|  35 kB    00:00     
(openSUSE:13.2) rpmlint-mini-1.5-8.1.9.x86_64.rpm    100%
|==========================| 1.7 MB    00:02     
(openSUSE:13.2) libncurses6-5.9-52.2.3.x86_64.rpm    100%
|==========================| 362 kB    00:02     
 Verifying integrity of cached packages
using keys from openSUSE:13.2
Writing build configuration
Running build
root's password:
logging output to /var/tmp/build-root/openSUSE_13.2-x86_64/.build.log...
[    0s] Memory limit set to 30209708KB
[    0s] Using BUILD_ROOT=/var/tmp/build-root/openSUSE_13.2-x86_64
[    0s] Using BUILD_ARCH=x86_64:i686:i586:i486:i386
[    0s] 
[    0s] 
[    0s] panos started "build hello.spec" at Thu Sep 17 12:33:24 UTC 2015.
[    0s] 
[    0s] 
[    0s] processing recipe
/suse/pgeorgiadis/building/home:pgeorgiadis/hello/hello.spec ...
[    0s] running changelog2spec --target rpm --file
/suse/pgeorgiadis/building/home:pgeorgiadis/hello/hello.spec
[    0s] init_buildsystem --configdir /usr/lib/build/configs --cachedir
/var/cache/build --rpmlist /tmp/rpmlist.gqa5_i
/suse/pgeorgiadis/building/home:pgeorgiadis/hello/hello.spec ...
[    1s] [1/29] preinstalling filesystem...
[    1s] [2/29] preinstalling glibc...
[    1s] [3/29] preinstalling fillup...
[    1s] [4/29] preinstalling libattr1...
[    1s] [5/29] preinstalling libbz2-1...
[    1s] [6/29] preinstalling libcap2...
[    1s] [7/29] preinstalling libelf1...
[    1s] [8/29] preinstalling libgcc_s1...
[    1s] [9/29] preinstalling liblua5_1...
[    1s] [10/29] preinstalling liblzma5...
[    1s] [11/29] preinstalling libpcre1...
[    1s] [12/29] preinstalling libpopt0...
[    1s] [13/29] preinstalling libz1...
[    1s] [14/29] preinstalling attr...
[    1s] [15/29] preinstalling libacl1...
[    1s] [16/29] preinstalling libncurses5...
[    1s] [17/29] preinstalling libselinux1...
[    1s] [18/29] preinstalling libreadline6...
[    1s] [19/29] preinstalling bash...
[    1s] [20/29] preinstalling diffutils...
[    1s] [21/29] preinstalling m4...
[    1s] [22/29] preinstalling sed...
[    1s] [23/29] preinstalling tar...
[    1s] [24/29] preinstalling grep...
[    1s] [25/29] preinstalling coreutils...
[    1s] [26/29] preinstalling permissions...
[    1s] [27/29] preinstalling aaa_base...
[    1s] [28/29] preinstalling rpm...
[    2s] [29/29] preinstalling pam...
[    2s] 
[    3s] running aaa_base preinstall script
[    3s] running aaa_base postinstall script
[    3s] Updating /etc/sysconfig/language...
[    3s] Updating /etc/sysconfig/backup...
[    3s] Updating /etc/sysconfig/proxy...
[    3s] Updating /etc/sysconfig/windowmanager...
[    3s] Updating /etc/sysconfig/news...
[    3s] Updating etc/passwd...unchanged
[    3s] Updating etc/group...unchanged
[    3s] Updating etc/shadow...new
[    3s] initializing rpm db...
[    4s] reordering...cycle: libcrack2 -> cracklib
[    4s]   breaking dependency libcrack2 -> cracklib
[    4s] done
[    4s] [1/98] cumulate file-magic-5.19-3.1.2
[    4s] [2/98] cumulate filesystem-13.2-4.3.1
[    4s] [3/98] cumulate insserv-compat-0.1-12.2.2
[    4s] [4/98] cumulate kernel-obs-build-3.16.6-2.1
[    4s] [5/98] cumulate terminfo-base-5.9-52.2.3
[    4s] [6/98] cumulate glibc-2.19-16.2.5
[    4s] [7/98] cumulate fillup-1.42-271.1.2
[    4s] [8/98] cumulate libatomic1-4.8.3+r212056-2.2.4
[    4s] [9/98] cumulate libattr1-2.4.47-4.1.2
[    4s] [10/98] cumulate libaudit1-2.4-1.3
[    4s] [11/98] cumulate libbz2-1-1.0.6-29.2.7
[    4s] [12/98] cumulate libcap-ng0-0.7.4-3.1.2
[    4s] [13/98] cumulate libcap2-2.22-13.1.8
[    4s] [14/98] cumulate libelf1-0.158-4.2.6
[    5s] [15/98] cumulate libgcc_s1-4.8.3+r212056-2.2.4
[    5s] [16/98] cumulate libgdbm4-1.11-3.1.2
[    5s] [17/98] cumulate libgmp10-5.1.3-3.1.2
[    5s] [18/98] cumulate libgomp1-4.8.3+r212056-2.2.4
[    5s] [19/98] cumulate libitm1-4.8.3+r212056-2.2.4
[    5s] [20/98] cumulate liblua5_1-5.1.5-10.2.2
[    5s] [21/98] cumulate liblzma5-5.0.7-1.1
[    5s] [22/98] cumulate libpcre1-8.35-3.2.3
[    5s] [23/98] cumulate libpopt0-1.16-27.1.2
[    5s] [24/98] cumulate libsepol1-2.3-2.1.8
[    5s] [25/98] cumulate libsmartcols1-2.25.1-2.4
[    5s] [26/98] cumulate libustr-1_0-1-1.0.4-33.1.2
[    5s] [27/98] cumulate libuuid1-2.25.1-2.4
[    5s] [28/98] cumulate libz1-1.2.8-5.1.2
[    5s] [29/98] cumulate net-tools-1.60-765.1.2
[    5s] [30/98] cumulate patch-2.7.1-7.1.2
[    5s] [31/98] cumulate perl-base-5.20.1-1.3
[    5s] [32/98] cumulate update-alternatives-1.16.10-8.1.2
[    5s] [33/98] cumulate brp-extract-appdata-2012.02.13-22.1.2
[    6s] [34/98] cumulate build-mkbaselibs-20140424-2.1.3
[    6s] [35/98] cumulate attr-2.4.47-4.1.2
[    6s] [36/98] cumulate libacl1-2.2.52-5.1.2
[    6s] [37/98] cumulate libasan0-4.8.3+r212056-2.2.4
[    6s] [38/98] cumulate libblkid1-2.25.1-2.4
[    6s] [39/98] cumulate libisl10-0.12.2-2.1.2
[    6s] [40/98] cumulate libmpfr4-3.1.2-5.1.2
[    6s] [41/98] cumulate libselinux1-2.3-2.2.5
[    6s] [42/98] cumulate libstdc++6-4.8.3+r212056-2.2.4
[    6s] [43/98] cumulate libtsan0-4.8.3+r212056-2.2.4
[    6s] [44/98] cumulate libmagic1-5.19-3.1.2
[    6s] [45/98] cumulate libzio1-1.00-10.1.2
[    6s] [46/98] cumulate file-5.19-3.1.2
[    6s] [47/98] cumulate libcloog-isl4-0.18.1-2.1.2
[    6s] [48/98] cumulate libdb-4_8-4.8.30-29.1.2
[    6s] [49/98] cumulate libmount1-2.25.1-2.4
[    6s] [50/98] cumulate libmpc3-1.0.2-2.1.2
[    6s] [51/98] cumulate libncurses5-5.9-52.2.3
[    6s] [52/98] cumulate libncurses6-5.9-52.2.3
[    7s] [53/98] cumulate libsemanage1-2.3-2.1.8
[    7s] [54/98] cumulate libreadline6-6.2-75.3.1
[    7s] [55/98] cumulate perl-5.20.1-1.3
[    7s] [56/98] cumulate cpp48-4.8.3+r212056-2.2.4
[    7s] [57/98] cumulate brp-check-suse-13.2+git20140727.d76b99d-3.2.1
[    7s] [58/98] cumulate bash-4.2-75.3.1
[    7s] [59/98] cumulate build-compare-2014.07.15-4.1.5
[    7s] [60/98] cumulate cpp-4.8-7.1.2
[    7s] [61/98] cumulate bzip2-1.0.6-29.2.7
[    7s] [62/98] cumulate xz-5.0.7-1.1
[    7s] [63/98] cumulate info-4.13a-38.1.2
[    7s] [64/98] cumulate cpio-2.11-29.2.3
[    7s] [65/98] cumulate diffutils-3.3-6.1.2
[    7s] [66/98] cumulate gzip-1.6-8.1.8
[    7s] [67/98] cumulate m4-1.4.17-2.1.2
[    7s] [68/98] cumulate make-4.0-2.2.3
[    7s] [69/98] cumulate sed-4.2.2-5.1.2
[    7s] [70/98] cumulate tar-1.28-2.2.2
[    7s] [71/98] cumulate which-2.20-4.1.2
[    8s] [72/98] cumulate findutils-4.5.14-2.1.2
[    8s] [73/98] cumulate gawk-4.1.1-3.1.2
[    8s] [74/98] cumulate gettext-runtime-mini-0.19.2-2.1.3
[    8s] [75/98] cumulate grep-2.20-2.1.2
[    8s] [76/98] cumulate binutils-2.24-6.1.7
[    8s] [77/98] cumulate coreutils-8.23-2.3.1
[    8s] [78/98] cumulate linux-glibc-devel-3.16-2.1.7
[    8s] [79/98] cumulate systemd-rpm-macros-2-8.1.2
[    8s] [80/98] cumulate glibc-locale-2.19-16.2.5
[    8s] [81/98] cumulate gettext-tools-mini-0.19.2-2.1.3
[    8s] [82/98] cumulate permissions-2014.08.26.1452-1.2
[    8s] [83/98] cumulate rpm-4.11.3-1.2
[    8s] [84/98] cumulate glibc-devel-2.19-16.2.5
[    8s] [85/98] cumulate libutempter0-1.1.6-5.2.3
[    8s] [86/98] cumulate rpmlint-mini-1.5-8.1.9
[    8s] [87/98] cumulate rpmlint-Factory-1.0-88.1.2
[    8s] [88/98] cumulate gcc48-4.8.3+r212056-2.2.4
[    8s] [89/98] cumulate gcc-4.8-7.1.2
[    8s] [90/98] cumulate libcrack2-2.9.0-4.1.3
[    9s] [91/98] cumulate cracklib-2.9.0-4.1.3
[    9s] [92/98] cumulate pam-1.1.8-11.1.2
[    9s] [93/98] cumulate pam-modules-12.1-24.1.2
[    9s] [94/98] cumulate util-linux-2.25.1-2.4
[    9s] [95/98] cumulate aaa_base-13.2+git20140911.61c1681-1.3
[    9s] [96/98] cumulate rpm-build-4.11.3-1.2
[    9s] [97/98] cumulate aaa_base-malloccheck-13.2+git20140911.61c1681-1.3
[    9s] [98/98] cumulate post-build-checks-13.2+git20140318.f24deaf-2.1.2
[    9s] now installing cumulated packages
[    9s] Preparing...
########################################
[    9s] Updating / installing...
[    9s] terminfo-base-5.9-52.2.3
########################################
[    9s] filesystem-13.2-4.3.1
########################################
[    9s] glibc-2.19-16.2.5
########################################
[    9s] libz1-1.2.8-5.1.2
########################################
[    9s] perl-base-5.20.1-1.3
########################################
[    9s] libgcc_s1-4.8.3+r212056-2.2.4
########################################
[    9s] libgmp10-5.1.3-3.1.2
########################################
[    9s] libbz2-1-1.0.6-29.2.7
########################################
[    9s] libstdc++6-4.8.3+r212056-2.2.4
########################################
[    9s] libncurses5-5.9-52.2.3
########################################
[    9s] fillup-1.42-271.1.2
########################################
[    9s] libisl10-0.12.2-2.1.2
########################################
[    9s] libmpfr4-3.1.2-5.1.2
########################################
[    9s] libattr1-2.4.47-4.1.2
########################################
[    9s] libaudit1-2.4-1.3
########################################
[    9s] libcap2-2.22-13.1.8
########################################
[    9s] liblzma5-5.0.7-1.1
########################################
[    9s] libacl1-2.2.52-5.1.2
########################################
[    9s] libmpc3-1.0.2-2.1.2
########################################
[    9s] libcloog-isl4-0.18.1-2.1.2
########################################
[   10s] cpp48-4.8.3+r212056-2.2.4
########################################
[   10s] libgomp1-4.8.3+r212056-2.2.4
########################################
[   10s] libpcre1-8.35-3.2.3
########################################
[   10s] libselinux1-2.3-2.2.5
########################################
[   10s] libpopt0-1.16-27.1.2
########################################
[   10s] libuuid1-2.25.1-2.4
########################################
[   10s] libblkid1-2.25.1-2.4
########################################
[   10s] update-alternatives-1.16.10-8.1.2
########################################
[   10s] libmount1-2.25.1-2.4
########################################
[   10s] libzio1-1.00-10.1.2
########################################
[   10s] libreadline6-6.2-75.3.1
########################################
[   10s] bash-4.2-75.3.1
########################################
[   10s] info-4.13a-38.1.2
########################################
[   10s] coreutils-8.23-2.3.1
########################################
[   11s] diffutils-3.3-6.1.2
########################################
[   11s] grep-2.20-2.1.2
########################################
[   11s] permissions-2014.08.26.1452-1.2
########################################
[   11s] Updating /etc/sysconfig/security...
[   11s] Warning: running kernel does not support fscaps
[   11s] Checking permissions and ownerships - using the permissions files
[   11s]    /etc/permissions
[   11s]    /etc/permissions.easy
[   11s]    /etc/permissions.local
[   11s] setting /dev/full to root:root 0666. (wrong permissions 0622)
[   11s] setting /sbin/unix_chkpwd to root:shadow 4755. (wrong owner/group
root:root)
[   11s] sed-4.2.2-5.1.2
########################################
[   11s] cpio-2.11-29.2.3
########################################
[   11s] findutils-4.5.14-2.1.2
########################################
[   11s] gawk-4.1.1-3.1.2
########################################
[   11s] update-alternatives: using /usr/bin/gawk to provide /bin/awk (awk) in
auto mode
[   12s] binutils-2.24-6.1.7
########################################
[   12s] update-alternatives: using /usr/bin/ld.bfd to provide /usr/bin/ld (ld)
in auto mode
[   13s] libutempter0-1.1.6-5.2.3
########################################
[   13s] Warning: running kernel does not support fscaps
[   13s] linux-glibc-devel-3.16-2.1.7
########################################
[   13s] glibc-devel-2.19-16.2.5
########################################
[   13s] systemd-rpm-macros-2-8.1.2
########################################
[   14s] glibc-locale-2.19-16.2.5
########################################
[   14s] gzip-1.6-8.1.8
########################################
[   14s] make-4.0-2.2.3
########################################
[   14s] tar-1.28-2.2.2
########################################
[   14s] which-2.20-4.1.2
########################################
[   14s] gettext-runtime-mini-0.19.2-2.1.3
########################################
[   14s] gettext-tools-mini-0.19.2-2.1.3
########################################
[   15s] cpp-4.8-7.1.2
########################################
[   15s] bzip2-1.0.6-29.2.7
########################################
[   15s] xz-5.0.7-1.1
########################################
[   15s] libcrack2-2.9.0-4.1.3
########################################
[   15s] cracklib-2.9.0-4.1.3
########################################
[   15s] pam-1.1.8-11.1.2
########################################
[   15s] Warning: running kernel does not support fscaps
[   15s] libdb-4_8-4.8.30-29.1.2
########################################
[   15s] libncurses6-5.9-52.2.3
########################################
[   15s] libasan0-4.8.3+r212056-2.2.4
########################################
[   15s] libtsan0-4.8.3+r212056-2.2.4
########################################
[   15s] libatomic1-4.8.3+r212056-2.2.4
########################################
[   15s] libcap-ng0-0.7.4-3.1.2
########################################
[   15s] libelf1-0.158-4.2.6
########################################
[   15s] libgdbm4-1.11-3.1.2
########################################
[   16s] perl-5.20.1-1.3
########################################
[   16s] libitm1-4.8.3+r212056-2.2.4
########################################
[   16s] gcc48-4.8.3+r212056-2.2.4
########################################
[   16s] gcc-4.8-7.1.2
########################################
[   16s] liblua5_1-5.1.5-10.2.2
########################################
[   16s] libsepol1-2.3-2.1.8
########################################
[   16s] libsmartcols1-2.25.1-2.4
########################################
[   16s] libustr-1_0-1-1.0.4-33.1.2
########################################
[   16s] libsemanage1-2.3-2.1.8
########################################
[   16s] net-tools-1.60-765.1.2
########################################
[   16s] patch-2.7.1-7.1.2
########################################
[   16s] insserv-compat-0.1-12.2.2
########################################
[   16s] util-linux-2.25.1-2.4
########################################
[   16s] Warning: running kernel does not support fscaps
[   16s] setting /usr/bin/wall to root:tty 2755. (wrong permissions 0755)
[   16s] setting /usr/bin/write to root:tty 2755. (wrong permissions 0755)
[   16s] Warning: running kernel does not support fscaps
[   16s] aaa_base-13.2+git20140911.61c1681-1.3
########################################
[   17s] Updating /etc/sysconfig/language...
[   17s] Updating /etc/sysconfig/backup...
[   17s] Updating /etc/sysconfig/proxy...
[   17s] Updating /etc/sysconfig/windowmanager...
[   17s] Updating /etc/sysconfig/news...
[   17s] Updating etc/passwd...unchanged
[   17s] Updating etc/group...unchanged
[   17s] Updating etc/shadow...unchanged
[   17s]
aaa_base-malloccheck-13.2+git20140911.########################################
[   17s] file-magic-5.19-3.1.2
########################################
[   17s] libmagic1-5.19-3.1.2
########################################
[   17s] rpm-4.11.3-1.2
########################################
[   17s] Updating /etc/sysconfig/services...
[   18s] rpmlint-mini-1.5-8.1.9
########################################
[   18s] file-5.19-3.1.2
########################################
[   18s] rpm-build-4.11.3-1.2
########################################
[   18s] rpmlint-Factory-1.0-88.1.2
########################################
[   18s]
post-build-checks-13.2+git20140318.f24########################################
[   18s]
brp-check-suse-13.2+git20140727.d76b99########################################
[   18s] pam-modules-12.1-24.1.2
########################################
[   18s] Warning: running kernel does not support fscaps
[   18s] m4-1.4.17-2.1.2
########################################
[   18s] build-compare-2014.07.15-4.1.5
########################################
[   18s] attr-2.4.47-4.1.2
########################################
[   18s] brp-extract-appdata-2012.02.13-22.1.2
########################################
[   18s] build-mkbaselibs-20140424-2.1.3
########################################
[   19s] kernel-obs-build-3.16.6-2.1
########################################
[   19s] removing nis flags from
/var/tmp/build-root/openSUSE_13.2-x86_64/etc/nsswitch.conf...
[   19s] now finalizing build dir...
[   19s] -----------------------------------------------------------------
[   19s] ----- building hello.spec (user abuild)
[   19s] -----------------------------------------------------------------
[   19s] -----------------------------------------------------------------
[   19s] + exec rpmbuild -ba --define '_srcdefattr (-,root,root)' --nosignature
/home/abuild/rpmbuild/SOURCES/hello.spec
[   19s] Executing(%prep): /bin/sh -e /var/tmp/rpm-tmp.lltMuT
[   19s] + umask 022
[   19s] + cd /home/abuild/rpmbuild/BUILD
[   19s] + exit 0
[   19s] Executing(%build): /bin/sh -e /var/tmp/rpm-tmp.4VTIQ2
[   19s] + umask 022
[   19s] + cd /home/abuild/rpmbuild/BUILD
[   19s] + /usr/bin/rm -rf /home/abuild/rpmbuild/BUILDROOT/hello-0-0.x86_64
[   19s] ++ dirname /home/abuild/rpmbuild/BUILDROOT/hello-0-0.x86_64
[   19s] + /usr/bin/mkdir -p /home/abuild/rpmbuild/BUILDROOT
[   19s] + /usr/bin/mkdir /home/abuild/rpmbuild/BUILDROOT/hello-0-0.x86_64
[   19s] + exit 0
[   19s] Executing(%install): /bin/sh -e /var/tmp/rpm-tmp.sCRndc
[   19s] + umask 022
[   19s] + cd /home/abuild/rpmbuild/BUILD
[   19s] + install -D -m 755 /home/abuild/rpmbuild/SOURCES/hello.sh
/home/abuild/rpmbuild/BUILDROOT/hello-0-0.x86_64/usr/bin/hello.sh
[   19s] + /usr/lib/rpm/brp-compress
[   19s] + /usr/lib/rpm/brp-suse
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-05-permissions
[   19s] setting / to root:root 0755. (wrong owner/group abuild:abuild)
[   19s] setting /usr/ to root:root 0755. (wrong owner/group abuild:abuild)
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-15-strip-debug
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-25-symlink
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-30-desktop
[   19s] WARNING: '/usr/lib/rpm/brp-desktop.data/suse-screensavers.menu' does
not exist
[   19s] WARNING: '/usr/lib/rpm/brp-desktop.data/preferences-gnome.menu' does
not exist
[   19s] WARNING: '/usr/lib/rpm/brp-desktop.data/applications-kmenuedit.menu'
does not exist
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-35-rpath
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-40-rootfs
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-45-tcl
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-50-check-python
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-55-boot-scripts
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-60-hook
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-65-lib64-linux
[   19s] calling /usr/lib/rpm/brp-suse.d/brp-72-extract-appdata
[   19s] Processing files: hello-0-0.x86_64
[   19s] Provides: hello = 0-0 hello(x86-64) = 0-0
[   19s] Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
[   19s] Requires: /bin/bash
[   19s] Checking for unpackaged file(s): /usr/lib/rpm/check-files
/home/abuild/rpmbuild/BUILDROOT/hello-0-0.x86_64
[   19s] Wrote: /home/abuild/rpmbuild/SRPMS/hello-0-0.src.rpm
[   19s] Wrote: /home/abuild/rpmbuild/RPMS/x86_64/hello-0-0.x86_64.rpm
[   19s] Executing(%clean): /bin/sh -e /var/tmp/rpm-tmp.LQOxLH
[   19s] + umask 022
[   19s] + cd /home/abuild/rpmbuild/BUILD
[   19s] + /usr/bin/rm -rf /home/abuild/rpmbuild/BUILDROOT/hello-0-0.x86_64
[   19s] + exit 0
[   19s] ... checking for files with abuild user/group
[   19s] ... running 00-check-install-rpms
[   19s] ... installing all built rpms
[   19s] Preparing packages...
[   19s] hello-0-0.x86_64
[   20s] ... running 01-check-debuginfo
[   20s] ... testing for empty debuginfo packages
[   20s] ... running 02-check-gcc-output
[   20s] ... testing for serious compiler warnings
[   20s]     (using /usr/lib/build/checks-data/check_gcc_output)
[   20s]     (using /var/tmp/build-root/openSUSE_13.2-x86_64/.build.log)
[   20s] ... running 03-check-binary-kernel-log
[   20s] ... running 04-check-filelist
[   20s] ... checking filelist
[   20s] ... running 05-check-invalid-requires
[   20s] ... running 06-check-installtest
[   20s] ... testing for pre/postinstall scripts that are not idempotent
[   20s] ... running 08-check-permissions
[   20s] ... testing for modified permissions
[   20s] ... running 09-check-packaged-twice
[   20s] ... running 10-check-lanana
[   20s] ... running 12-check-libtool-deps
[   20s] ... testing devel dependencies required by libtool .la files
[   20s]     (can be skipped by "skip-check-libtool-deps" anywhere in spec)
[   20s] ... running 13-check-invalid-provides
[   20s] ... running 14-check-gconf-scriptlets
[   20s] ... testing GConf scriptlet presence
[   20s] ... running 72-translate-appdata
[   20s] ... running 99-check-remove-rpms
[   20s] ... removing all built rpms
[   20s]     (order: reverse hello)
[   20s] 
[   20s] RPMLINT report:
[   20s] ===============
[   20s] hello.x86_64: W: no-manual-page-for-binary hello.sh
[   20s] Each executable in standard binary directories should have a man page.
[   20s] 
[   20s] hello.x86_64: W: no-changelogname-tag
[   20s] hello.src: W: no-changelogname-tag
[   20s] There is no changelog. Please insert a '%changelog' section heading in
your
[   20s] spec file and prepare your changes file using e.g. the 'osc vc'
command.
[   20s] 
[   20s] hello.x86_64: W: no-binary
[   20s] The package should be of the noarch architecture because it doesn't
contain
[   20s] any binaries.
[   20s] 
[   20s] hello.src:43: W: macro-in-comment %{buildroot}
[   20s] There is a unescaped macro after a shell style comment in the
specfile. Macros
[   20s] are expanded everywhere, so check if it can cause a problem in this
case and
[   20s] escape the macro with another leading % if appropriate.
[   20s] 
[   20s] hello.x86_64: W: invalid-url URL TODO
[   20s] hello.src: W: invalid-url URL TODO
[   20s] The value should be a valid, public HTTP, HTTPS, or FTP URL.
[   20s] 
[   20s] hello.x86_64: W: description-shorter-than-summary
[   20s] hello.src: W: description-shorter-than-summary
[   20s] The package description should be longer than the summary. be a bit
more
[   20s] verbose, please.
[   20s] 
[   20s] 2 packages and 0 specfiles checked; 0 errors, 9 warnings.
[   20s] 
[   20s] 
[   20s] panos finished "build hello.spec" at Thu Sep 17 12:33:44 UTC 2015.
[   20s] 

/var/tmp/build-root/openSUSE_13.2-x86_64/home/abuild/rpmbuild/SRPMS/hello-0-0.src.rpm

/var/tmp/build-root/openSUSE_13.2-x86_64/home/abuild/rpmbuild/RPMS/x86_64/hello-0-0.x86_64.rpm
```

### Build it remotely

In order to build it remotely, on the OBS side of things you first have to add
the files in order to be staged.

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc addremove
A    hello.sh
A    hello.spec
```

then write the changelog for version control:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc vc
```

and type what's all about. For example:

```changes
-------------------------------------------------------------------
Thu Sep 17 12:34:27 UTC 2015 - pgeorgiadis@novell.com

- Initial Package
```

after that, a new file `hello.changes` will be automatically created. Let's add
it also:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc addremove
A    hello.changes
```

And now we are good to go and commit the changes to OBS:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc ci
Sending    hello.sh
Sending    hello.spec
Sending    hello.changes
Transmitting file data ...
Committed revision 1.
```

Now, go to the web-interface, go to your project `hello` (which can be found at
your `Home Project` page) and check the `Build Result` status on the right side
of the page. Also, feel free to hit refresh of the results (not the whole
page). Once it's done, you should see `succeeded`.

### How to share it with your friends:

If you want to know the repository click on the distribution you have just build for (e.g. [openSUSE_13.2-x86_64](https://build.opensuse.org/package/binaries/home:pgeorgiadis/hello?repository=openSUSE_13.2)
) and then click on [Go to download repository](http://download.opensuse.org/repositories/home:/pgeorgiadis/openSUSE_13.2/).

However, the link you are going to share with your friend is not that. You can
find that link at your project's page, saying [Download
package](https://software.opensuse.org/download.html?project=home%3Apgeorgiadis&package=hello)

This page will contain automatically the information for your friends to
install your package. For example, for OpenSUSE they are going to be provided
with a `One-click installer` or with the manual instructions:

``bash
zypper addrepo
http://download.opensuse.org/repositories/home:pgeorgiadis/openSUSE_13.2/home:pgeorgiadis.repo
zypper refresh
zypper install hello
```

### Update your script

In case you want to make some new changes, for example:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> vi hello.sh 
```

source code:
```bash
#!/bin/bash
VERSION=1.0
echo "Hello World"
echo "Good night"
```

I added a new variable with the name of the version and another print message.
Then let's edit also the `.spec` file:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> vi hello.spec
```

and change the version also there:
```spec
Version:        1.0
```

finally, write the new changelog:
```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> obs vc
```

for example:
```bash
Thu Sep 17 12:46:32 UTC 2015 - pgeorgiadis@novell.com

- Update to 1.0
  Be polite and say good night
```

and commit:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc ci
Sending    hello.changes
Sending    hello.sh
Sending    hello.spec
Transmitting file data ...
Committed revision 2.
```

The packages are going to be also rebuild automatically on the OBS.

### Build for Fedora and CentOS

If you want to build for more **RPM** distributions like Fedora, CentOS, RHEL
etc and go to OBS web-interface > Home Project and then click on the **tab**
[Repositories](https://build.opensuse.org/project/repositories/home:pgeorgiadis).Once
you go there, click [Add repositories] and select the distributions you are
going to build with. Let's select only the RPM once since there are more stuff
needed to build a debian package.

Now, everytime I commit my changes on OBS, my package is going to be build also
for OpenSUSE, CentOS and Fedora.

### Dependencies

Let's pretend that my script uses NFS server. Thus I need NFS server to be
present on the machine. Then I am going to change the spec file accordingly:

```spec
PreReq:         nfs-client
```

The `PreReq` means that my package is going to require nfs-client package to be
present both for installation and runtime. Also, if sometime infuture someone
tries to remove `nfs-client` then my package `hello` is also going to be
removed.

If there's different name package for OpenSUSE, I can do it like that:
```spec
%if 0%{?suse_version}
PreReq:         yast2-nfs-client
%else
PreReq:         nfs-client
%endif
```

This means that if th  system is OpenSUSE then it's going to install
`yast2-nfs-client`. If the system is something else (e.g. CentOS or Fedora in
our case) it's going to install `nfs-client` package.

The difference between `PreReq` and `Requires` is that:
`PreReq`: Needed for Installation and Runtime
`Requires`: Needed only for Runtime

Also, note that there's no need to change the version of my script since I
didn't make any changes. However, I am going to document what have changes in
that revision:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc vc
```

for example:
```
-------------------------------------------------------------------
Thu Sep 17 13:14:54 UTC 2015 - pgeorgiadis@novell.com

- Added pre_req for nfs_server for any suse
```

and commit:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc ci
Sending    hello.changes
Sending    hello.spec
Transmitting file data ..
Committed revision 3.
```

### A friend sends you a request

Let's pretend that a friend of yours or someone else sends you a request. First
of all, how can he do that:

He creates a folder to download your project:

```bash
ktsamis@kostas:~> cd Documents/hello/
```

he branches it:

```bash
ktsamis@kostas:~/Documents/hello> osc -H -A https://api.opensuse.org/ bco home:pgeorgiadis/hello
```

after that, he has branches my project into his home project:

```bash
ktsamis@kostas:~/Documents/hello> cd
home:ktsamis:branches:home:pgeorgiadis/hello/
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>ls
total 20
drwxr-xr-x 3 ktsamis suse 4096 Sep 17 15:40 ./
drwxr-xr-x 4 ktsamis suse 4096 Sep 17 15:40 ../
-rw-r--r-- 1 ktsamis suse  483 Sep 17 15:18 hello.changes
-rw-r--r-- 1 ktsamis suse   62 Sep 17 14:48 hello.sh
-rw-r--r-- 1 ktsamis suse 1289 Sep 17 15:18 hello.spec
drwxr-xr-x 2 ktsamis suse 4096 Sep 17 15:40 .osc/
```

then he edits my script, for example:
```bash
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>vi hello.sh
```

source code:
```
#!/bin/bash
VERSION=1.1
echo "Hello World"
echo "Good night"
echo "Version is: $VERSION"
```

So, he add a new `echo` and since he edits my script he also bumps the version
from `1.0` to `1.1`.

then he runs the script to make sure he didn't make any mistake:
```bash
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>chmod +x hello.sh
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>./hello.sh
Hello World
Good night
Version is: 1.1
```

So, as long as it's ok, then he proceeds with changing the version number also
from the spec file:

```bash
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>vi hello.spec
```

```bash
Version:        1.1
```

and last but not least, write the changelog:

```bash
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>osc vc
```

```changes
-------------------------------------------------------------------
Thu Sep 17 13:44:32 UTC 2015 - ktsamis@suse.com

- Update to Version 1.1. Now it shows you the version
```

then do a diff to see what changed and commit to his branch
```bash
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>osc diff 
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>osc ci 
Sending    hello.changes
Sending    hello.sh
Sending    hello.spec
Transmitting file data ...
Committed revision 2.
```

Finally, he sends me a merge request in order to apply his changes into mine
original project.

```bash
ktsamis@kostas:~/Documents/hello/home:ktsamis:branches:home:pgeorgiadis/hello>osc sr 
created request id 331791
```

### Merging the request from your friend

I can see that I have on task to do via (my mail) or going to the
web-interface. There I can see everything, like the  changelog, what changed
(diff) and so on.

If I prefer the command line, I can see it like that:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc request show -d
331791
Request: #331791

  submit:       home:ktsamis:branches:home:pgeorgiadis/hello@2(cleanup) ->
home:pgeorgiadis


Message:
- Update to Version 1.1. Now it shows you the version

State:   new        2015-09-17T13:46:04 ktsamis
Comment: <no comment>

History: 2015-09-17T13:46:04 ktsamis      Request created
```

and in order to approve it:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> opensc up
U    hello.changes
U    hello.sh
U    hello.spec
At revision 4.
```

This merges his code into mine, and the package is rebuilding again online on
the OBS. Also, I have the locally the latest version:

```bash
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> ls
hello.changes  hello.sh  hello.spec
pgeorgiadis@panos:~/building/home:pgeorgiadis/hello> cat hello.sh
#!/bin/bash
VERSION=1.1
echo "Hello World"
echo "Good night"
echo "Version is: $VERSION"
```

