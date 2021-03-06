# Padcheck: A TLS CBC Padding Oracle Scanner

This tool tests how a server responds to various CBC padding errors.

The tool makes a series of connections where the TLS record containing an HTTP request is malformed. Servers should respond uniformly to all malformed records. If the server responds differently to certain types of errors, an attacker may be able to construct a padding oracle for use in an adaptive chosen ciphertext attack.

There are currently five malformed record test cases: 
1. Invalid MAC with Valid Padding (0-length pad)
2. Missing MAC with Incomplete Padding (255-length pad)
3. Valid MAC with Inconsistent Padding (SSLv3 style padding)
4. Missing MAC with Valid Padding (Entire record is padding)
5. Invalid MAC with Valid Padding (0-length record)

## Usage

|              |        |                                                                                        |
| ------------ | ------ | -------------------------------------------------------------------------------------- |
| -h           |        | Show help                                                                              |
| -hosts       | string | Filename containing hosts to query                                                     |
| -iterations  | int    | Number of iterations required to confirm oracle (default 3)                            |
| -keylog      | string | Path to a file NSS key log export (needed to decrypt pcap files) (default "/dev/null") |
| -v           | int    | Specify verboseness level (default: 1, max: 5) (default 1)                             |
| -workerCount | int    | Desired number of workers for testing lists (default 32)                               |

The basic usage is to run ```padcheck hostname```
A list of hosts can also be read from a file ```padcheck -hosts hostnames.txt```

Vulnerable hosts are indicated in the tool output with a line similar to:

*Hostname (ip:443)* is VULNERABLE with a *Observable MAC Validity (Zombie POODLE)* oracle when using cipher *0xc027* with TLS *0x0303*. The fingerprint is *6867b5*

The fingerprint produced by this tool is a hash of the server responses. These values are subject to change with changes to the tool or with environmental variation which may influence the error message text. The fingerprint value should therefore be primarily used for correlating similar vulnerabilities within a specific environment.


## Building (manual)

This tool requires first building a patched Go toolchain. See [Installing Go from source](https://golang.org/doc/install/source) for compiler requirements.

Run `./build.sh` to pull the appropriate version of the Go source, patch it, build it, then use it to build the `padcheck` binary.

## Docker build (recommended)

Building with Docker is easier and cross-platform.

Run `docker build . -t padcheck` to build the patched Go toolchain and the `padcheck` tool in a container.

Run with: `docker run --rm -it padcheck [args]`

If you want to use a hosts file or keylog file, you will need to mount them in the container:

```sh
docker run --rm -it \
    -v /path/to/hosts:/tmp/hosts \
    -v /path/to/keylog:/tmp/keylog \
    padcheck -hosts /tmp/hosts -keylog /tmp/keylog
```

## Credits

The original idea for this padding check tool was a very simple tool for checking for POODLE issues in TLS servers, by Adam Langley (`agl` AT `imperialviolet` DOT `org`). See:

- https://www.imperialviolet.org/2014/12/08/poodleagain.html
- https://www.imperialviolet.org/binary/poodle-tls-go.patch
- https://www.imperialviolet.org/binary/scanpadding.go

## Additional Resources

More information about scanning for TLS CBC padding oracles on the Internet can be found in this repo: https://github.com/RUB-NDS/TLS-Padding-Oracles


## License

Original tool copyright 2014 Adam Langley, released under a BSD license.

Copyright 2019 Tripwire, Inc. All rights reserved.
Released under a [BSD 2-Clause License](./LICENSE).
