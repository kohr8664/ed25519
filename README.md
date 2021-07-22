# ED25519 (Supercharged)

## About

This is a portable implementation of [Ed25519](http://ed25519.cr.yp.to/) based
on the SUPERCOP "ref10" implementation. Additionally there is key exchanging
and scalar addition included to further aid building a PKI using Ed25519. All
code is licensed under the permissive zlib license.

All code is pure ANSI C without any dependencies, except for the random seed
generation which uses standard OS cryptography APIs (`CryptGenRandom` on
Windows, `/dev/urandom` on nix). If you wish to be entirely portable define
`ED25519_NO_SEED`. This disables the `ed25519_create_seed` function, so if your
application requires key generation you must supply your own seeding function
(which is simply a 256 bit (32 byte) cryptographic random number generator).

But, instead of using SHA-512 my fork use tinyBLAKE3.

## Differences in this fork

The main difference from the original [ED25519](https://github.com/orlp/ed25519) ORLP's implementation is that instead of using SHA-512 I use [tinyBLAKE3](https://github.com/kohr8664/tinyBLAKE3).

tinyBLAKE3 is nothing more than BLAKE3 but with some personal modifications in the source tree. You should assume tinyBLAKE3 as the official BLAKE3 C implementation but with a delicious CMake script.

## Usage

If you are using CMake on your project (recommended) just clone (or submodule) this project into your source tree and then use CMake's `add_subdirectory` to automatically add it to your building queue.

If you are not using CMake on your project then you can compile the libraries by simply invoking cmake with:

```bash
mkdir build && cd build
cmake ..
make
```

This will build ED25519 and tinyBLAKE3 in the `build` directory you can use these binaries to link against your project.

Then simple copy the include folder content to their desired location in your source tree :)

ED22519 will require you to define some arrays with fixed size for it's operations. Here's a list of them from the original repository.

```c
unsigned char seed[32];
unsigned char signature[64];
unsigned char public_key[32];
unsigned char private_key[64];
unsigned char scalar[32];
unsigned char shared_secret[32];
```

In the original repository it's said that `unsigned char` is not required but I strongly recommend you to stick to it.

## API

This section describe this library functions, all of this is from the original repository.

```c
int ed25519_create_seed(unsigned char *seed);
```

Creates a 32 byte random seed in `seed` for key generation. `seed` must be a
writable 32 byte buffer. Returns 0 on success, and nonzero on failure.

```c
void ed25519_create_keypair(unsigned char *public_key, unsigned char *private_key,
                            const unsigned char *seed);
```

Creates a new key pair from the given seed. `public_key` must be a writable 32
byte buffer, `private_key` must be a writable 64 byte buffer and `seed` must be
a 32 byte buffer.

```c
void ed25519_sign(unsigned char *signature,
                  const unsigned char *message, size_t message_len,
                  const unsigned char *public_key, const unsigned char *private_key);
```

Creates a signature of the given message with the given key pair. `signature`
must be a writable 64 byte buffer. `message` must have at least `message_len`
bytes to be read. 

```c
int ed25519_verify(const unsigned char *signature,
                   const unsigned char *message, size_t message_len,
                   const unsigned char *public_key);
```

Verifies the signature on the given message using `public_key`. `signature`
must be a readable 64 byte buffer. `message` must have at least `message_len`
bytes to be read. Returns 1 if the signature matches, 0 otherwise.

```c
void ed25519_add_scalar(unsigned char *public_key, unsigned char *private_key,
                        const unsigned char *scalar);
```

Adds `scalar` to the given key pair where scalar is a 32 byte buffer (possibly
generated with `ed25519_create_seed`), generating a new key pair. You can
calculate the public key sum without knowing the private key and vice versa by
passing in `NULL` for the key you don't know. This is useful for enforcing
randomness on a key pair by a third party while only knowing the public key,
among other things.  Warning: the last bit of the scalar is ignored - if
comparing scalars make sure to clear it with `scalar[31] &= 127`.


```c
void ed25519_key_exchange(unsigned char *shared_secret,
                          const unsigned char *public_key, const unsigned char *private_key);
```

Performs a key exchange on the given public key and private key, producing a
shared secret. It is recommended to hash the shared secret before using it.
`shared_secret` must be a 32 byte writable buffer where the shared secret will
be stored.

## License and Credits

I must thank [orpl](https://github.com/orlp) and the original repository contributors for 
awesome work they made.

Without their work, this one wouldn't be possible!

All code is released under the zlib license. See license.txt for details.
