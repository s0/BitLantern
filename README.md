# Cryptographic Verification of a Reproducible builds

## Scope

### The Problem

Open source software is incredibly valuable to the general population,
particularly in circumstances regarding privacy + security, there are numerous
open source tools in particular that allow people to communicate and use the
web privately and anonymously. However there is a problem in the way in which
OSS Software is typically distributed, which is by the compiled binary files /
software rather than the source code, and there is no easy way in which to
check that any particular binary is directly derived from a given set of source
code, which means that it may have been modified by the entities building +
distributing the software. This is because there is typically some amount of
non-determinism when building from source code (building twice in a row is not
likely to result in the same binaries, and it gets even more tricky when you
talk about building from different computers).

Enter [Reproducible Builds](https://reproducible-builds.org): this project aims
to make building from source code much more deterministic, so that different
people can independently build the software from source code, and produce the
exact same source code. There are [a large number of
projects](https://reproducible-builds.org/who/) that have started work on this,
and this is extremely exciting.

This work is extremely valuable, and very crucial to the health of our future
tech ecosystem, but for this to mean anything, we need to start actually using
the deterministic properties of our builds to protect us from entities
(maliciously or otherwise) modifying binaries that we install on our systems.

### The Solution

This project aims to take advantage of deterministic builds to it's fullest
extent, providing a service where anyone can choose to sign a particular binary
as "verified", and tie that verification to their public ID. Users of the
service can then request a list of signatures for any particular binary
(preferably before using the binary), and see who has verified it as genuinely
from the source code.

**Example Usages:**

* Linux package managers (for example in Debian) could require that a package is
  signed by a minimum number of signers in the keychain.

  An extension of this would be allowing users / sysadmins to configure their
  package manager to require signatures from certain people / identities before
  installing certain packages.

* Android devices could check the list of verifications for a particular app,
  and ensure that an `apk` has not been tampered with.

  An extension to this would be requiring that certain APPs are signed by
  certain people before allowing an update to take place. This can probably be
  done with something like F-Droid more easily than the Play Store.

### But what about [Cothority](https://github.com/dedis/cothority)?

Cothority is cool, but is in essence tackling a very different, and much more
general problem (and has nothing to do with reproducible / deterministic builds,
and in fact conceptually doesn't even require software to be Open Source).

Cothority, conceptually, is a network of nodes that maintain a list of hashes
that are considered "valid", for software, that would be the hash of the binary
files. For a piece of software to be considered valid, it's hash must be
submitted to the network, and naturally propagates throughout the whole network
so all nodes eventually agree that it is valid.

Users of the system can then ask a minimum number of nodes to verify if they
have "witnessed" a particular hash, and if all of them have, the user can choose
to continue using the binary.

A good example of where this could be useful is with iOS software updates, if
Apple were to use a system like this, it would be impossible to create a version
of the software that is targeted for an individual device, and to install it on
that device without announcing the existence of this software to the entire
world.

So although extremely valuable, verifying reproducible builds is strictly more
powerful than Cothority, but is applicable to significantly less use cases.

## Naming

We still need to think of an official name for this project + associated tools.

Possible Options:
* BitLantern / BitTorch / BitLighthouse (some other light source)
* BitCop / BitSherif (less good, people don’t necessarily like cops, especially
  in US).

## Architecture

First and foremost, the entire system / API will be **Open Source**.

This project will be based around an API server that will be at the core of the
project. The website, command line tools and any other apps will all use the
same API (and this will allow for extensive contributions + other 3rd party
apps by external contributors).

The API server should be written in a way that allows for federation / mirrors
of the data, so that people can host their own servers, but synchronize with
the main server / any other mirrors to mirror / distribute the data (making the
system inherently more robust, ala PGP keyservers).

## API

The API will mostly be auth-less (including calls to upload signatures for a
binary), due to the nature of this project. We may later require auth for some
parts of the API (for example updating metadata for a project, we may want to
give auth to project owners, verifying for example via GitHub).

* Get list of projects
* Given a project:
  * Get Meta
  * Get latest versions
* Given a file / sha2 hash:
  * Get the verifications of this file
* Upload a signature

## Data Model

TODO
