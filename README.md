Build instructions for (Maven) Central artifacts
================================================

As part of [Reproducible Builds efforts for the JVM](https://reproducible-builds.org/docs/jvm/), this repository is an attempt
at writing rebuild instructions for the artifacts available in [(Maven) Central](https://repo.maven.apache.org/maven2/),
equivalent to the packaging instructions that are maintained by every Linux distribution
(for example Debian's [debian/rules](https://www.debian.org/doc/debian-policy/ch-source#s-debianrules) or ArchLinux's PKGBUILD), whatever the build tool or the build platform.

The fact that build platforms used for building artifacts published to Maven Central are not uniform (unlike a Linux distribution), defining the environment requirements of the rebuild instructions will be a tricky part: which OS is necessary? Has it to be very precise (a specific version of a specific distribution of Linux?) or can it be loose (any Unix)? Which version of which build tools have to be installed? With the same question on version precision. The objective is to track only key information that impacts the binary, to have a chance to get the same binary as the binary found on Central, but letting as much flexibility as possible for the rebuilder to create his rebuild platform.

Once rebuild instructions are precise enough for people from different horizons to be able to rebuild typical projects as their binary was published on Central,
we'll see buildinfo more in details: how does it link to build instructions and add extra details?

And in the future, once formats are stabilized, it is expected that every project provides its rebuild instructions (including explicit prerequisites) when publishing to Central. Then the current work is not necessary to be maintained forever: just for old artifacts.

### Interesting cases built with Maven

For each project, an `analyze.sh` script is written to detect the key environment setup used to build the binary published in Central: the result can be found in `result.txt`. The `build` variable in the script is not really used, just stored in the analysis script in preparation of future steps where real rebuild instructions will be written.

- [Plexus Utils](https://codehaus-plexus.github.io/plexus-utils/) = `org.codehaus.plexus:`[`plexus-utils:*`](https://repo.maven.apache.org/maven2/org/codehaus/plexus/plexus-utils/)\
  __Simple jar__ from basic java code that could be built without Maven but with javac and jar commands.\
  No buildable source tarball available anywhere, just scm (when still available, since migrations over time)\
  See [results](https://github.com/jvm-repo-rebuild/reproducible-central/tree/master/org/codehaus/plexus/plexus-utils)\
  __Learning 1__: target bytecode != minimum required JDK version to build != effective JDK used for the reference build on Central (example: for plexus-utils 3.1.1, target = 6, minimum = 7, used = 8)\
  __Learning 2__: bytecode generated by different major JDK version is very different, then to reproduce the binary published in central, it's the effective JDK version that counts, not minimum (even less target)\
  __Learning 3__: some versions were built on Windows, which cause a source of variability in a few text files but not bytecode.
  Windows vs Unix (any) prerequisite will have to be stored in rebuild instructions, or if possible the build script will have
  to be improved to remove the impact of Windows platform on the binary (considering any Unix will be easier for rebuilders to setup)

- [Jansi](http://fusesource.github.io/jansi/) = `org.fusesource.jansi:`[`jansi:*`](https://repo.maven.apache.org/maven2/org/fusesource/jansi/jansi/)\
  Jar built from Java code and uber-jar with dependencies.\
  Notice: __multi-module Maven build__ (root artifactId = `jansi-project`, modules = `jansi` & `example`, groupId = [`org.fusesource.jansi`](https://repo.maven.apache.org/maven2/org/fusesource/jansi/))\
  See [results](https://github.com/jvm-repo-rebuild/reproducible-central/tree/master/org/fusesource/jansi/jansi-project)\
  __Learning__: For future automation, it's more convenient to place rebuild instructions on root artifact and let modules point to the root
  (and ignore the fact that the useul artifact is jansi and the git repository name jansi.git is the id of a module instead of the root artifactId):
  this will permit a Maven plugin to automatically adapt buildinfo format in root vs modules

- [Maven (core)](https://maven.apache.org/ref/) = `org.apache.maven:`[`apache-maven:2.0.10+`](https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/)\
  multi-module (root artifactId = `maven`, modules vary over time), buildable source in repository as `apache-maven-<version>-src.zip` and `.tar.gz` in `org.apache.maven:apache-maven` module\
  See [results](https://github.com/jvm-repo-rebuild/reproducible-central/tree/master/org/apache/maven/maven)\
  __Learning__: added `-Papache-release -Dgpg.skip` to build command to get source and javadoc attached artifacts (`-sources.jar` and `-javadoc.jar`), useful to check reproducibility since downloaded by IDEs

- [Apache Royale](https://royale.apache.org/): `org.apache.royale.` [`compiler`](https://repo.maven.apache.org/maven2/org/apache/royale/compiler) [`typedefs`](https://repo.maven.apache.org/maven2/org/apache/royale/typedefs) and [`framework`](https://repo.maven.apache.org/maven2/org/apache/royale/framework)\
  a few prerequisites to install ("Flash Player projector content debugger", airglobal-20.0.swc): see [build instructions](https://github.com/apache/royale-asjs/wiki/Build-Apache-Royale-with-Maven)\
  __Learning__: prerequisites installation instructions needed, at least as documentation, but in the future in an executable way

- [Jansi native](https://github.com/fusesource/jansi-native) = `org.fusesource.jansi:`[`jansi-native:*`](https://repo.maven.apache.org/maven2/org/fusesource/jansi/jansi-native/)\
  java code plus **additional native (then build-platform dependant with 7 different platforms) code**.\
  Until 1.5, native code published as jansi-native artifactId, since 1.6 published
  as separate `jansi-${platform}` where platform may be `osx`, `linux32`, `linux64`, `windows32`, `windows64`, `freebsd32` or `freebsd64`\
  __Learning 1__: sometimes, not only one very specific OS is required, but sometimes also multiple OSes\
  __Learning 2__: in addition, some native tools are necessary (like a C compiler or some system libraries)

### Interesting cases built with SBT

TBD

### Interesting cases built with (name another build tool)

TBD
