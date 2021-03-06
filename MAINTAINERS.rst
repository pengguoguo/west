Notes for maintainers cutting a release.

Pre-release test plan
---------------------

1. Make tox happy on the following first-party platforms:

   - Windows 10
   - the latest macOS
   - the latest Ubuntu LTS

2. Make tox happy on other popular Linux distributions as resources allow.
   Doing this in a container is fine.

   - Arch
   - the latest Ubuntu release (if different than the latest LTS)
   - Debian stable (if its Python 3 is still supported)
   - Debian testing

3. Build alpha N (N=1 to start, then N=2 if you need more commits, etc.) and
   upload to pypi.

   Make sure to check if west.manifest.SCHEMA_VERSION also needs an update
   before uploading. ::

     git clean -ffdx
     WEST_VERSION=X.YaN python3 setup.py sdist bdist_wheel
     twine upload -u zephyr-project dist/*

4. Install the alpha on test platforms. ::

     pip3 install west==X.YaN

5. Create and update a default (Zephyr) workspace on all of the platforms from
   1., using the installed alpha::

     west init zephyrproject
     cd zephyrproject
     west update

6. Do the following Zephyr specific testing in the Zephyr workspace on all of
   the platforms from 1. Skip QEMU tests on non-Linux platforms, and make sure
   ZEPHYR_BASE is unset in the calling environment. ::

     west build -b qemu_x86 -s zephyr/samples/hello_world -d build-qemu-x86
     west build -d build-qemu-x86 -t run

     west build -b qemu_cortex_m3 -s zephyr/samples/hello_world -d build-qemu-m3
     west build -d build-qemu-m3 -t run

     # This example uses a Nordic board. Do this for as many boards
     # as you have access to / volunteers for.
     west build -b nrf52_pca10040 -s zephyr/samples/hello_world -d build-nrf52
     west flash -d build-nrf52
     west debug -d build-nrf52
     west debugserver -d build-nrf52
     west attach -d build-nrf52

   (It's still a pass if ``west build`` requires ``--pristine``.)

7. Assuming that all went well (if it didn't, go fix it and repeat),
   get the version bump committed, either to master or the release branch as
   appropriate. See below for release branch information.

   For the first release (e.g. vX.Y.0), master should point to the release
   branch commit. Thereafter, the release branch is allowed to fork from master
   to just take bugfixes etc.

Building and uploading the release wheels
-----------------------------------------

You need the zephyr-project PyPI credentials for the 'twine upload' command. ::

  git clean -ffdx
  python3 setup.py sdist bdist_wheel
  twine upload -u zephyr-project dist/*

The 'git clean' step is important. We've anecdotally observed broken wheels
being generated from dirty repositories.

Tagging the release
-------------------

Create and push a GPG signed tag. ::

  git tag -a -s vX.Y.Z -m 'West vX.Y.Z

  Signed-off-by: Your Name <your.name@example.com>'

  git push origin vX.Y.Z

Cut a release branch
--------------------

If you've cut a new minor version (vX.Y.0), also cut a release branch,
vX.Y-branch. Subsequent fixes for versions vX.Y.Z should go to that branch
after being backported from master (or the other way around in case of an
urgent hotfix).
