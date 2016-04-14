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
that are considered "valid"; for software, that would be the hash of the binary
files. For a piece of software to be considered valid, it's hash must be
submitted to the network, and naturally propagates throughout the whole network
so all nodes eventually agree that it is valid.

Users of the system can then ask a minimum number of nodes to verify if they
have "witnessed" a particular hash, and if enough of them have, the user can
choose to continue using the binary.

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

## Signature Data

The data that will be signed by verifiers will be a json file of this format:

```json
{
  "src": "https://github.com/WhisperSystems/Signal-Android.git",
  "version": "3.15.2",
  "srcVersion": "a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6",
  "label": "Play Store APK",
  "verifications": [
    {
      "type": "gpg",
      "data": "-----BEGIN PGP SIGNATURE-----\nComment: ..."
    },
    {
      "type": "sha256",
      "data": "097a35284640d7fad85ff00b3ac100bcc207556176080071723c4bed37889057"
    },
  ],
  "signature":{
    "type":"saltpack",
    "data":"BEGIN SALTPACK SIGNED MESSAGE. kXR7VktZdyH7rvq v5wcIkHbs7XwHpb nPtLcF6vE5yY63t aHF62jiEC1zHGqD inx5YqK0nf5W9Lp TvUmM2zBwxgd3Nw kvzZ96W7ZfDdTVg F5Y99c2l5EsCy1I xVNl0nY1TP25vsX 2cRXXPUrM2UKtWq UK2HG2ifBSOED4w
    ...
    0hffDmUk71TlfVx XZCF3voC2ysgl3g YdLz4rDRzMJgd2m 01HIbfdsoZpAMty O27WtUNRLV1iyC9 tK5ApCyekI4nWcf 2OvTHnC8ma7bloW XAG. END SALTPACK SIGNED MESSAGE."
  }
}
```

**Field Breakdown:**

* `"src"` - a URL to the repository of the source code, in this example, a https
  git repo to the GitHub repository of Signal.
* `"version"` - the version string of the version being signed.
* `"srcVersion"` - a unique reference to version of the source code in the VCS
  system for this project (for git, this would be the full commit hash).
* `"label"` - a text string identifying what kind of binary data produced from
  the source is being verified, for example:
  * A source repository may have different build targets (linux, windows,
    OSX...), and each of the files for each of the different platforms will
    require a separate verification, and should be labelled appropriately.
  * The output of a build may be multiple different files, each of which need to
    be distributed separately, and therefore signed separately. A good label for
    each of these may be the filenames.
  * A project may be used across many different linux distributions and
    package repositories, and each of these distributions will require
    distributing the packages in a different manner (`.deb`, `.tar.gz`, ...), so
    therefore each will require separate signatures and appropriate labels.
* `"verifications"` - a list of verifications, either just hashes of the files,
  or a gpg signature of the file.

* `"signature"` - a single signatures that signs a message constructed from the
  preceding fields. Specifically form a canonical version of the precedeing
  fields. Encode these fields as a string. Removal white space and sign it. The
  order of the fields is 1. "src" 2."version" 3. "srcVersion" 4."label" 5.
  "verifications". Implementers should cache the canonical json to allow valid
  signatures to remain after scheme updates.


  There will be a minimum requirement of at least a particular hash algorithm
  (probably blake2) which will serve as **canonical** hash of a file, and the
  way in which we correlate verifications of the same file over different
  algorithms. The API server will only accept verifications which meet this
  requirement.

  (We could potentially change which hash this is at a later date, which would
  change the way in which we canonicalize verifications).

**A note on GPG:**

Usage of GPG as a "hash" will allow for simpler systems to be implemented using
the API service as a GPG signature lookup server, treating GPG in whatever
manner they see fit, without having to implement the complete signing and
verification protocol of this service, and understand this JSON format.

i.e: this opens up the possibility to being able to just make a `http(s)`
request for "give me the GPG signatures for the file with this hash", and then
using the GPG signatures directly.

### Signing

*(yet to be finalised)*

The signature could either just be a GPG signature, or it could be a keybase
signature, or it could potentially be either / both. This needs to be
discussed... either way, it probably needs to be something that can be tied to
a user identity, so a raw NaCl signature, for example, probably won't do.

Why two fields for signatures?

The support for signing in the verification field provides a place for gitian
style signatures or potentially cothority signatures. This field may simply
contain a digest of the bytes. The second signature attests the full dataset the
user is validating

## API

The API will mostly be auth-less (including calls to upload signatures for a
binary), due to the nature of this project. We may later require auth for some
parts of the API (for example updating metadata for a project, we may want to
give auth to project owners, verifying for example via GitHub).

### Get list of projects

```
GET /api/projects
```

Order by something significant (unless otherwise specified), e.g. number of
signatures.

```json
[
  {
    "id": 1234,
    "name": "Signal Android",
    "url": "https://play.google.com/store/apps/...",
    "icon": "https://.../icon.png",
    "src": [
      "https://github.com/WhisperSystems/Signal-Android.git"
    ]
  },
  ...
]
```

### Get a specific project's meta

```
GET /api/projects/1234
```

```json
{
  "id": 1234,
  "name": "Signal Android",
  "url": "https://play.google.com/store/apps/...",
  "icon": "https://.../icon.png",
  "src": [
    "https://github.com/WhisperSystems/Signal-Android.git"
  ]
}
```

### Get a list of versions for a project

```
GET /api/projects/1234/versions?count=10
```

Order by date (that's the date of the commit in the repo).

```json
[
  {
    "date": 1460412723,
    "srcVersion": "a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6",
    "binaries": {
      "097a352...": {
        "projects": {
          "1234": 121,
          "333": 1
        },
        "srcVersions": {
          "a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6": 121,
          "778b1e2fd4656cb6ac086e6a307ff350c4f2ef0c": 1
        },
        "versions": {
          "3.15.2": 101,
          "v3.15.2": 20,
          "v3.15.1": 1
        },
        "labels": {
          "Play Store APK": 95,
          "Play Store": 27
        },
        "verificationTypes": {
          "blake2": 122,
          "gpg": 56,
          "sha256": 100
        }
      },
      "d4656cb...": {

      },
      ...
    }

  },
  ...
]
```

### Get details of binaries for a specific srcVersion (git hash)

```
GET /api/projects/1234/a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6
```

```json
{
  "date": 1460412723,
  "srcVersion": "a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6",
  "binaries": {
    "097a352...": {
      "projects": {
        "1234": 121,
        "333": 1
      },
      "srcVersions": {
        "a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6": 121,
        "778b1e2fd4656cb6ac086e6a307ff350c4f2ef0c": 1
      },
      "versions": {
        "3.15.2": 101,
        "v3.15.1": 1
        "v3.15.2": 20,
      },
      "labels": {
        "Play Store APK": 95,
        "Play Store": 27
      },
      "verificationTypes": {
        "blake2": 122,
        "gpg": 56,
        "sha256": 100
      }
    },
    "d4656cb...": {

    },
    ...
  }

}
```

### Get details for a specific file hash

```
GET /api/hash/097a35284640d7fad85ff00b3ac100bcc207556176080071723c4bed37889057
```

```json
{
  "hash": "097a35284640d7fad85ff00b3ac100bcc207556176080071723c4bed37889057",
  "projects": {
    "1234": 121,
    "333": 1
  },
  "srcVersions": {
    "a307ff350c4f2ef0c778b1e2fd4656cb6ac086e6": 121,
    "778b1e2fd4656cb6ac086e6a307ff350c4f2ef0c": 1
  },
  "versions": {
    "3.15.2": 101,
    "v3.15.2": 20,
    "v3.15.1": 1
  },
  "labels": {
    "Play Store APK": 95,
    "Play Store": 27
  },
  "verificationTypes": {
    "blake2": 122,
    "gpg": 56,
    "sha256": 100
  }
}
```

### Get verifications for a specific file hash

```
GET /api/hash/097a352...057/verifications?count=10
```

```json
[
  {
    "data": "-- escaped / encoded signed json data --",
    "signature": "-- escaped / encoded signature --"
  },
  ...
]
```

### Get verifications for a specific file hash that include a specific type

```
GET /api/hash/097a352...057/verifications?count=10&type=gpg
```

```json
[
  {
    "data": "-- escaped / encoded signed json data --",
    "signature": "-- escaped / encoded signature --"
  },
  ...
]
```

### Get gpg signatures for a specific hash

```
GET /api/hash/097a352...057/gpg?count=10
```

```json
[
  "-----BEGIN PGP SIGNATURE-----\nComment: ...",
  "-----BEGIN PGP SIGNATURE-----\nComment: ...",
  ...
]
```

### TODO

API calls for uploading a signature

## Data Model

TODO
