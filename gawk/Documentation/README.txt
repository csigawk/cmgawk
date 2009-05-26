Please note that the "ECM ThirdParty Gawk Tree" power point
presentation documents the previous build design approach used
(and lasted for only a single release of the gawk distribution).
Although the old approach does work, in practice it had several
shortcomings (this list also includes shortcomings with the previous
build structure and lack of an overall Makefile or other mechanism
for driving the tool build in a reproduceable way):

    - The previously proposed perforce baseless merge for handling
      upgrades doesn't really work well when there have been branches
      off of branches (a situation we deal with all the time).

    - It causes old files from the earlier pristine distribution to
      be carried forward incorrectly into the upgraded ECM branch.

    - There was no consistent, easily reproduceable mechanism for
      building the tool.  If one person passed in special "configure"
      options, such as a particular compiler to be used, this information
      was not captured, and thus, a subsequent developer would not likely
      build the next version the same way, possibly hitting issues that
      a previous developer already solved.

    - In an attempt to address the previous shortcoming, one developer
      checked in certain modifications to gawk config build files, which
      are normally generated during the build.  This not only was a
      questionable solution, but because the files were checked into
      perforce, the read-only file permissions during subsequent perforce
      extractions would cause build errors that would require changing
      the permissions each time.

    - Finally, it was too easy to accidentally modify the supposedly
      pristine gawk source distribution, resulting in a situation where
      the base distribution couldn't be trusted as pristine.


For all these reasons, I decided to move to a different approach -- one
that is very similar to the FreeBSD ports system and the Linux RPM system.

Now, we simply check in the pristine source distribution tar file,
exactly as we download it from the third party source.  We also check in
the set of our ECM code changes as standard "patch" files.  That's all
that gets checked into perforce except for the Makefile (and possible
supporting mklib "make" definition files) which drives the build.  Now,
any special "configure" arguments / build options for each build platform
are captured and defined in the Makefile, and so no one needs to guess
how we build the software package each time.

To upgrade distributions, we simply download the new version, check it
in and update the Makefile to use the new version.  Often most of the
patch files won't even need to be modified, though for generality and
safety sake, I have placed the patch files in a separate subdirectory,
tied to the distribution release level.  So, when upgrading, one will
also want to copy the patch files into a new subdirectory, named to
match to the release level.

See the comments at the top of the Makefile for a description of the
build options, but basically, typing "make" will completely build the ECM-
extended gawk executable.  This will extract the base distribution, apply
the ECM patches, run "configure" and then run "make" in a per-platform
build directory (build/<hostname>).  Typing "make test" after that
will run through the large set of gawk tests that are included with
the distribution.  Finally, if the "gawk" binary in ../../bin/<platform>
is opened for edit in your perforce workspace, typing "make install" will
copy a stripped version of the built executable into the right place,
ready to be subsequently checked into perforce.

The most important step to remember when making code-changes is to
generate and check-in up-to-date patch files.  The complete set of
patch files will be auto-generated simply by running "make patches".
Copy these files from the build/patches directory to patches/<distrib>,
and check updates into perforce as appropriate.

