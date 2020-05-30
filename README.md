# Packaging repo for GTSAM

How to package GTSAM and upload to Ubuntu PPA


## Grab the modified source code

    git clone https://github.com/berndpfrommer/gtsam.git gtsam
	cd gtsam
    git checkout 4.0.2_pkg

What version of the tree you should check out?

  - If it's a new version number that you have *never* uploaded to the PPA you plan to upload it to, then checkout the exact version you want to package, for instance a release tag.

  - If you have already uploaded a source package before, i.e. there is an orig.tar.gz sitting in the cloud at Ubuntu, check out exactly that source version

## Make the tar ball

    git archive --format=tar HEAD -o ../gtsam_${gtsam_version}.orig.tar
	gzip ../gtsam_${version}.orig.tar

This will create a tar ball with the original, unchanged sources. You can only upload a tarball like that *once* for any given version, e.g. 4.0.2. All subsequent modifications that you make to the sources, for instance to get the sources to build, must be captured by patches with respect to that tar ball. The ``quilt`` tool is used for such patching. 

If you don't know exactly what is uploaded at this point, download the ``*.orig.tar.gz`` file from launchpad, and put it in place of the above one. The build process will croak if the sources differ, so you should get an alert.


## Grab the debian files from the packaging repo

    cd gtsam
	git clone https://github.com/berndpfrommer/gtsam_packaging debian

## Use quilt to update the source tree if needed

The quilt patch management tool needs some configuration, so here is an alias for it that you need to run in the ``gtsam`` source tree (one up from the debian tree)

    cd gtsam/debian
    alias dquilt="quilt --quiltrc=`pwd`/quiltrc-dpkg"
    cd ..
	dquilt push -a   # apply all patches up til now

You can find useful info on the quilt tool here: https://wiki.debian.org/UsingQuilt
Now *before* you make any more changes to any files in the source tree, start a new patch:

    dquilt new whatever-this-patch-is-called.patch

And add the file(s) you need to edit *before* you edit them, for example

    dquilt add CMakeLists.txt
	(now edit CMakeLists.txt)
	dquilt refresh  # re-read all edited files and record their changes, as often as you like

When you are done with all patches, save them away

    dquilt header -e   # finish the patch you started
    dquilt pop -a      # unapply all patches, revert to original source

## Update the changelog

Since you may end up uploading the same major+minor number software with slight packaging modificiations until you are happy with what is in your ppa, you must maintain your own version number that needs to lexicographically increase every time you upload a new version. Traditionally this version number has been just 1, 2... etc, but it is a good idea to use it to indicate where this package comes from, i.e. your personal ppa. Let's define a shell variable for this:

    mod_version=2pfrommer1

You also must set up a variable that captures the actual gtsam version you are releasing. There is no wiggle room here, it has to be three numbers, like so:

    gtsam_version=4.0.2

We also need to specify what Ubuntu distribution we are targeting, i.e:

    distro=bionic

Further required: your name and email. This must go into the DEBEMAIL variable:

    export DEBEMAIL="Bernd Pfrommer <bernd.pfrommer@gmail.com>"

Lastly, some change comments are required, for instance:

    change_comment="Modified CMakeLists.txt to build on newer Ubuntu distros"

Now (inside the gtsam directory) update the ``debian/changelog`` file like so:

    debchange --package gtsam --newversion ${gtsam_version}-${mod_version} --distribution $distro -b --force-distribution $change_comment

Note: The package name ``gtsam`` (specified with the ``--package`` option) must agree with what is in the ``Source`` section of the ``debian/control`` file. This is the name under which later the ``*.orig.tar.gz`` file will be searched for when you build the debian source package.

## Adjusting the debian rules for custom build flags

If you want custom build flags, edit the ``debian/rules`` file accordingly

## Doing a test build

Inside the ``gtsam`` source directory, do a local test build to see that you can build a binary package.

    cd gtsam
    debuild -b -us -uc

If you make any changes to get things to work, like editing CMakeLists.txt, *do not* commit them to the original source tree. 

## Build and sign the source debian package

You need to have a secret gpg key set up to build and sign a source package:

    gpg --list-secret-keys --keyid-format SHORT

The key id is the 8-byte hex number right after "rsaNNN/":

    sec   rsa3072/C50C547F 2020-05-19 [SC]
          921389CD8281C9B5A463C9D18CAF5F59C50C547F
    uid         [ultimate] Bernd Pfrommer (Ubuntu PPA) <bernd.pfrommer@gmail.com>
    ssb   rsa3072/5D91D951 2020-05-19 [E]

So in this case it's ``C50C547F``. You must have that key deposited beforehand with the ubuntu keyserver, and activated on launchpad (see instructions there), so in this case:

    gpg --send-keys --keyserver keyserver.ubuntu.com 8CAF5F59C50C547F

Again in the ``gtsam`` directory, build a source package:

    debuild -k 8CAF5F59C50C547F -S

This should prompt you for passphrase if your private key is protected

## Upload to PPA:

The directory one up from the source directory should now have various packages. Upload this one (adjust ppa setting)

    dput "ppa:bernd-pfrommer/gtsam" gtsam_4.0.2-${mod_version}_source.changes

## Uploading multiple distributions

Now repeat the above steps, starting with the changelog, for other distributions (bionic, focal....)


## Finishing up

When you are all done, commit your changes to the debian repo (not the gtsam source repo), and push.


## Troubleshooting

### Upload problems

If you get

    "Package has already been uploaded to ppa on ppa.launchpad.net"

remove the ``*.ppa.upload`` files

If you get an email back like the one below, download the ``*orig.tar.gz`` from launchpad, and replace your ``*orig.tar.gz`` with it before you do the ``debuild -S``.

    Rejected:
    File gtsam_4.0.2.orig.tar.gz already exists in GTSAM Georgia Tech Smoothing And Mapping, but uploaded version has different contents.


### Build problems

Reading the error messages of failed GTSAM builds can be really hard, because the actual error often is drowned by "error" messages when Cmake probes and doesn't find certain libraries.

In the output, search for "Configuring incomplete, errors occurred!"
