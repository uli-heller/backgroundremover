README-uli.md
=============

Ubuntu-21.10
------------

I'll start creating packages for Ubuntu-21.10,
because most of the dependencies are packages for this
already.

### Preparations

On your desktop:

- Create a LXC container: `lxc launch images:ubuntu/21.10 ubuntu-2110`
- Stop the LXC container: `lxc stop ubuntu-2110`
- Copy the LXC container: `lxc copy ubuntu-2110backgroundremover-2110`
- Start the LXC container copy: `lxc start backgroundremover-2110`
- Determine the IP address: `lxc info backgroundremover-2110|grep inet:` -> "inet:  10.2.2.2/24 (global)"
- Copy the backgroundremover source folder into the LXC container copy
    - Either by doing something like this: `tar cf - ./backgroundremover|lxc exec backgroundremover-2110 --user 1000 -- bash -c "cd /home/ubuntu; tar xvf -"`
    - Or by: `lxc file push ...`
    - Or by executing `git clone ...` within the LXC container copy
- Start a shell within the LXC container copy: `lxc exec backgroundremover-2110 bash`

Within the LXC container copy:

- Install python development packages: `apt install python3-stdeb dh-python`

### Create Debian Source Packages

```
ubuntu@backgroundremover-2110:~/backgroundremover$ python3 setup.py --command-packages=stdeb.command sdist_dsc
running sdist_dsc
running egg_info
writing src/backgroundremover.egg-info/PKG-INFO
writing dependency_links to src/backgroundremover.egg-info/dependency_links.txt
...
dpkg-buildpackage: info: full upload (original source is included)
dpkg-source: warning: extracting unsigned source package (backgroundremover_0.1.9-1.dsc)
dpkg-source: info: extracting backgroundremover in backgroundremover-0.1.9
dpkg-source: info: unpacking backgroundremover_0.1.9.orig.tar.gz
dpkg-source: info: unpacking backgroundremover_0.1.9-1.debian.tar.xz
```

Preparations
------------

We need the packages "python3-stdeb" and "dh-python".
For Ubuntu-20.04, the version 0.8.5 of "python3-stdeb"
is available within the standard repos. Unfortunately,
using this version produces errors like
"dpkg-query: no packages found matching python-all".
So make sure to create/install version 0.10 of python3-stdeb.
For details, see section [stdeb](#stdeb)!

```
$ sudo apt install python3-stdeb
$ sudo apt install dh-python
```

Create Debian Source Packages
-----------------------------

```
$ python3 setup.py --command-packages=stdeb.command sdist_dsc
running sdist_dsc
running egg_info
creating src/backgroundremover.egg-info
...
dpkg-buildpackage: info: full upload (original source is included)
dpkg-source: warning: extracting unsigned source package (backgroundremover_0.1.7-1.dsc)
dpkg-source: info: extracting backgroundremover in backgroundremover-0.1.7
dpkg-source: info: unpacking backgroundremover_0.1.7.orig.tar.gz
dpkg-source: info: unpacking backgroundremover_0.1.7-1.debian.tar.xz
```

Produces these files:

- backgroundremover-0.1.7.tar.gz
- deb_dist/backgroundremover_0.1.7-1.debian.tar.xz
- deb_dist/backgroundremover_0.1.7-1.dsc
- deb_dist/backgroundremover_0.1.7-1_source.buildinfo
- deb_dist/backgroundremover_0.1.7-1_source.changes
- deb_dist/backgroundremover_0.1.7.orig.tar.gz
- dist/backgroundremover-0.1.7.tar.gz

Create Debian Package
---------------------

You basically need the files

- deb_dist/backgroundremover_0.1.7-1.debian.tar.xz
- deb_dist/backgroundremover_0.1.7-1.dsc
- deb_dist/backgroundremover_0.1.7-1_source.buildinfo
- deb_dist/backgroundremover_0.1.7-1_source.changes
- deb_dist/backgroundremover_0.1.7.orig.tar.gz

You create the debian package by doing this:

```
cd deb_dist
dpkg-source -x backgroundremover_0.1.7-1.dsc
cd backgroundremover-0.1.7
dpkg-buildpackage
cd ..
# name of the debian package: python3-backgroundremover_0.1.7-1_all.deb
```

Release To [Github](https://github.com/uli-heller/backgroundremover/releases)
-----------------

```
git tag u0.1.7
git push --tags
# upload these files
# - backgroundremover_0.1.7-1.debian.tar.xz
# - backgroundremover_0.1.7-1.dsc
# - backgroundremover_0.1.7-1_amd64.buildinfo
# - backgroundremover_0.1.7-1_amd64.changes
# - backgroundremover_0.1.7-1_source.buildinfo
# - backgroundremover_0.1.7-1_source.changes
# - backgroundremover_0.1.7.orig.tar.gz
# - python3-backgroundremover_0.1.7-1_all.deb
```

stdeb
-----

- Download source packages from [launchpad](https://launchpad.net/ubuntu/impish/+source/stdeb)
   - [DSC](https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stdeb/0.10.0-1/stdeb_0.10.0-1.dsc)
   - [ORIG.TAR.GZ](https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stdeb/0.10.0-1/stdeb_0.10.0.orig.tar.gz)
   - [DEBIAN.TAR.XZ](https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stdeb/0.10.0-1/stdeb_0.10.0-1.debian.tar.xz)
- Extract: `dpkg-source -x stdeb_0.10.0-1.dsc`
- Build: `( cd stdeb-0.10.0; dpkg-buildpackage )`
- Install: `sudo dpkg -i python3-stdeb_0.10.0-1_all.deb`

Problems
--------

### dpkg-query: no packages found matching python-all

When trying to create the debian source packages, you get lots of
output including a stack trace and the error message "dpkg-query: no packages found matching python-all".

This is probably caused by an old version of stdeb (package name "python3-stdeb").

Fix: Install a newer version!

```
$ python3 setup.py --command-packages=stdeb.command sdist_dsc
running sdist_dsc
running egg_info
creating src/backgroundremover.egg-info
...
Writing backgroundremover-0.1.7/setup.cfg
creating dist
Creating tar archive
removing 'backgroundremover-0.1.7' (and everything under it)
dpkg-query: no packages found matching python-all
ERROR running: /usr/bin/dpkg-query --show --showformat=${Version} python-all
Traceback (most recent call last):
  File "setup.py", line 12, in <module>
    setup(
  File "/home/uli/.local/lib/python3.8/site-packages/setuptools/__init__.py", line 161, in setup
    return distutils.core.setup(**attrs)
  File "/usr/lib/python3.8/distutils/core.py", line 148, in setup
    dist.run_commands()
  File "/usr/lib/python3.8/distutils/dist.py", line 966, in run_commands
    self.run_command(cmd)
  File "/usr/lib/python3.8/distutils/dist.py", line 985, in run_command
    cmd_obj.run()
  File "/usr/lib/python3/dist-packages/stdeb/command/sdist_dsc.py", line 135, in run
    build_dsc(debinfo,
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 1369, in build_dsc
    python_defaults_version_str = get_version_str('python-all')
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 268, in get_version_str
    stdout = get_cmd_stdout(args)
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 243, in get_cmd_stdout
    raise RuntimeError('returncode %d', returncode)
RuntimeError: ('returncode %d', 1)
```

### Can't locate Debian/Debhelper/Sequence/python3.pm

When trying to create the debian source packages, you get lots of
output including the error message "Can't locate Debian/Debhelper/Sequence/python3.pm".

Fix: Install "dh-python"!

```
$ python3 setup.py --command-packages=stdeb.command sdist_dsc
...
 dpkg-source --before-build .
dpkg-source: info: using options from backgroundremover-0.1.7/debian/source/options: --extend-diff-ignore=\.egg-info$
 fakeroot debian/rules clean
dh clean --with python3 --buildsystem=python_distutils
dh: error: unable to load addon python3: Can't locate Debian/Debhelper/Sequence/python3.pm in @INC (you may need to install the Debian::Debhelper::Sequence::python3 module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/x86_64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base) at (eval 11) line 1.
BEGIN failed--compilation aborted at (eval 11) line 1.

make: *** [debian/rules:7: clean] Error 255
dpkg-buildpackage: error: fakeroot debian/rules clean subprocess returned exit status 2
Traceback (most recent call last):
  File "setup.py", line 12, in <module>
    setup(
  File "/home/uli/.local/lib/python3.8/site-packages/setuptools/__init__.py", line 161, in setup
    return distutils.core.setup(**attrs)
  File "/usr/lib/python3.8/distutils/core.py", line 148, in setup
    dist.run_commands()
  File "/usr/lib/python3.8/distutils/dist.py", line 966, in run_commands
    self.run_command(cmd)
  File "/usr/lib/python3.8/distutils/dist.py", line 985, in run_command
    cmd_obj.run()
  File "/usr/lib/python3/dist-packages/stdeb/command/sdist_dsc.py", line 137, in run
    build_dsc(debinfo,
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 1569, in build_dsc
    dpkg_buildpackage(*args, cwd=fullpath_repackaged_dirname)
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 585, in dpkg_buildpackage
    process_command(args, cwd=cwd)
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 226, in process_command
    check_call(args, cwd=cwd)
  File "/usr/lib/python3/dist-packages/stdeb/util.py", line 59, in check_call
    raise CalledProcessError(retcode)
stdeb.util.CalledProcessError: 2
```

### python3-backgroundremover : Depends: python3-six (= 1.16.0) but 1.14.0-2 is to be installed

When trying to install the debian package,
you get an error messagecomplaining about "python3-six" and other python packages.

```
$ LANG=C sudo apt install  python3-backgroundremover
Reading package lists... Done
Building dependency tree
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 python3-backgroundremover : Depends: python3-six (= 1.16.0) but 1.14.0-2 is to be installed
                             Depends: python3-torch but it is not installable
                             Depends: python3-torchvision but it is not installable
E: Unable to correct problems, you have held broken packages.
```

Links
-----

- [Packaging Python programs - DEB (and RPM) packages](https://www.dlab.ninja/2015/11/packaging-python-programs-debian-and.html)

History
-------

- 2021-08-06: Initial version
