# Packaging repo for GTSAM

How to package GTSAM and upload to Ubuntu PPA


## Grab the modified source code

    git clone https://github.com/berndpfrommer/gtsam.git libgtsam-4.0.2
    git checkout 4.0.2_with_packaging

This will create a directory libgtsam-4.0.2 with the necessary "debian" packaging files in that directory.
Adjust the flags in the debian/rules file to compile GTSAM the way you want it.


## Modify the changelog

Edit the file ``libgtsam-4.0.2/debian/changelog`` to add a new release at the top of the file. You should mangle the release (in this case "trusty") into the version name. Here we have Debian release -1, Ubuntu release 0, for Trusty Thar.

    gtsam (4.0.2-1trusty0) trusty; urgency=medium
      * Initial release

    -- Bernd Pfrommer <bernd.pfrommer@gmail.com>  Mon, 25 May 2020 19:52:51 -0400


## Create the source (upstream) tarball

Inside the libtsam-4.0.2 directory, do this:

    git archive --format=tar HEAD -o ../libgtsam_4.0.2-1.orig.tar
	gzip ../libgtsam_4.0.2-1.orig.tar

## Build the source debian package

Again in the ``libgtsam-4.0.2`` directory, build a source package:

    debuild -k0x8CAF5F59C50C547F -S

Use ``gpg --list-secret-keys`` to find the secret keys you have. The key you use must be deposited beforehand with the ubuntu keyserver:

    gpg --send-keys --keyserver keyserver.ubuntu.com NAME_OF_KEY


## Upload to PPA:

The directory one up from the source directory should now have various packages. Upload this one (adjust ppa setting)

    dput "ppa:bernd-pfrommer/gtsam" gtsam_4.0.2-1trusty0_source.changes

	
## Uploading multiple distributions

Now repeat the above steps, starting with the changelog, for other distributions (bionic, focal....)


## Building binary packages

To build binary packages for local testing, again in the source directory:

For local package build, binary only, no sign sources (-us), no sign changes (-uc)

    debuild -b -us -uc

For source only, no sign sources (-us), no sign changes (-uc)

    debuild -S -us -uc

