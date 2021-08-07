README-uli.md
=============

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

Create XXX
----------

```
$ python3 setup.py --command-packages=stdeb.command sdist_dsc --with-python3=true
$ python3 setup.py --command-packages=stdeb.command sdist_dsc
running sdist_dsc
running egg_info
creating src/backgroundremover.egg-info
...
Writing backgroundremover-0.1.7/setup.cfg
creating dist
Creating tar archive
removing 'backgroundremover-0.1.7' (and everything under it)
dpkg-query: Kein Paket gefunden, das auf python-all passt
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

```
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

stdeb
-----

- Download source packages from [launchpad](https://launchpad.net/ubuntu/impish/+source/stdeb)
   - [DSC](https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stdeb/0.10.0-1/stdeb_0.10.0-1.dsc)
   - [ORIG.TAR.GZ](https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stdeb/0.10.0-1/stdeb_0.10.0.orig.tar.gz)
   - [DEBIAN.TAR.XZ](https://launchpad.net/ubuntu/+archive/primary/+sourcefiles/stdeb/0.10.0-1/stdeb_0.10.0-1.debian.tar.xz)
- Extract: `dpkg-source -x stdeb_0.10.0-1.dsc`
- Build: `( cd stdeb-0.10.0; dpkg-buildpackage )`
- Install: `sudo dpkg -i python3-stdeb_0.10.0-1_all.deb`

Links
-----

- [Packaging Python programs - DEB (and RPM) packages](https://www.dlab.ninja/2015/11/packaging-python-programs-debian-and.html)

History
-------

- 2021-08-06: Initial version
