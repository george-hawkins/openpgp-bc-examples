PGP encryption and decryption with Bouncy Castle
================================================

We want to generate a key pair suitable for use with PGP encryption and then use this key pair to encrypt and decrypt a file.

The Bouncy Castle bc-java repository contains [examples](https://github.com/bcgit/bc-java/tree/master/pg/src/main/java/org/bouncycastle/openpgp/examples) that demonstrate this (and many other things).

This repository contains those examples separated out from the rest of the Bouncy Castle code base to create a minimum standalone setup that can be built with Maven.

See below for a walkthru of the steps taken to separate out the Bouncy Castle examples and create the pom found in this repository.

But first let's build and run the examples as found here.

### Building and running

Build the examples using Maven and dump the classpath used by Maven to `classpath.txt` so we can use it to run the examples:
```bash
$ mvn clean package
$ mvn dependency:build-classpath -Dmdep.outputFile=classpath.txt
```

### Running the examples

Our sample file for encryption and decryption is [`magna-carta-1215.txt`](magna-carta-1215.txt).

First let's create a key pair for ourselves (that's secured with the pass phrase 'SuperSecret'):
```bash
$ java -classpath target/openpgp-bc-examples-0.1-SNAPSHOT.jar:$(< classpath.txt) org.bouncycastle.openpgp.examples.RSAKeyPairGenerator my-identifier SuperSecret
```

This will generate the file `pub.bgp` for the public part of the pair and the file `secret.bgp` (protected by the pass phrase) that contains the secret part.

Now let's encrypt the sample file - the result is found in `magna-carta-1215.txt.asc`:
```bash
$ java -classpath target/openpgp-bc-examples-0.1-SNAPSHOT.jar:$(< classpath.txt) com.example.KeyBasedFileProcessor -e -ai magna-carta-1215.txt pub.bpg 
$ less magna-carta-1215.txt.asc 
```

Now let's backup our original, decrypt our encrypted file and compare the original with the regenerated plaintext:
```bash
$ mv magna-carta-1215.txt magna-carta-1215.txt.orig
$ java -classpath target/openpgp-bc-examples-0.1-SNAPSHOT.jar:$(< classpath.txt) com.example.KeyBasedFileProcessor -d magna-carta-1215.txt.asc secret.bpg SuperSecret
$ diff magna-carta-1215.txt magna-carta-1215.txt.orig 
```

**Important:** here we've encrypted and decrypted our sample file with our own key pair. Usually you have a file that you need to send to someone - first you encrypt it with _their_ public key, send the encrypted file to them and then they decrypt it with _their_ secret key.

For more details on these examples and the options they take see the classes [`RSAKeyPairGenerator`](src/main/java/com/example/RSAKeyPairGenerator.java) and [`KeyBasedFileProcessor`](src/main/java/com/example/KeyBasedFileProcessor.java).

How this repository was created
-------------------------------

As noted above the examples here were copied from the Bouncy Castle bc-java repository, the package name used for the examples classes was then changed and a pom was constucted to pull in the necessary Bouncy Castle dependencies. These steps are walked thru below.

### Separating out the examples

First let's clone the Bouncy Castle bc-java repository and pull out the examples we're interested in into our own project called openpgp-bc-examples:

```bash
$ git clone git@github.com:bcgit/bc-java.git
$ mkdir openpgp-bc-examples
$ cd openpgp-bc-examples
$ mkdir -p src/main/java/com/example
$ DEST=$PWD/!$
$ cd ../bc-java/pg/src/main/java/org/bouncycastle/openpgp/examples
$ cp KeyBasedFileProcessor.java RSAKeyPairGenerator.java PGPExampleUtil.java $DEST
$ cd -
```

### Renaming the package

Now while still in the root diretory of our openpgp-bc-examples let's rename the package of our example classes to match where we put them:

```bash
$ sed -i 's/package org.bouncycastle.openpgp.examples/package com.example/' src/main/java/com/example/*
```

This isn't an optional step - we cannot use the Bouncy Castle package `org.bouncycastle.openpgp.examples` as the Bouncy Castle jars, that we'll depend on, are signed such that using their package names would result in an error like this:
```
Exception in thread "main" java.lang.SecurityException: class "org.bouncycastle.openpgp.examples.PGPExampleUtil"'s signer information does not match signer information of other classes in the same package
```

### Creating the pom

The details of the latest Bouncy castle releases can be found here: https://www.bouncycastle.org/latest_releases.html

From this you can determine that you need:

* bcprov-jdk15on - the Bouncy Castle Provider package for JDK 1.5 on.
* bcpg-jdk15on - the Bouncy Castle OpenPGP/BCPG package for JDK 1.5 on.

Then you can find the latest Maven Central versions here: http://search.maven.org/

With this information the standard [`pom.xml`](pom.xml) file found here was built.
