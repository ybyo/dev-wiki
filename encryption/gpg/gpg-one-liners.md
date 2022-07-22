# GPG One-liners

Used
in [Signing commits](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)

## Print GPG Fingerprint

```shell
gpg --list-keys
```

## Remove GPG key by Fingerprint

```shell
gpg --delete-keys --fingerprint <GPG Fingerprint>
```

## Add Git Default GPG Fingerprint

```shell
# Global
git config --global user.signingkey <GPG Fingerprint>
```

## Unset Git Default GPG Fingerprint

```shell
# Global
git config --global --unset-all user.signingkey
```

## Listing public keys

```sh
# Same result with `--list-secret-keys`
gpg --list-keys
```

## Exporting a public key

```sh
# Binary
gpg --output "public-key.gpg" --export "test@example.com"
```

```sh
# ASCII - When the key sent to email or published on a web page
gpg --armor --export "test@example.com"
```

## Exporting a private key

But this approach has security concerns according to [this](https://unix.stackexchange.com/a/482559).

```sh
# However, not recommended for security reasons
gpg --output "private-key.pgp" --armor --export-secret-key "test@example.com"
```

---

**Reference**

* [The GNU Privacy Handbook](https://www.gnupg.org/gph/en/manual/book1.htmll)
* [Stack Exchange](https://unix.stackexchange.com/questions/481939/how-to-export-a-gpg-private-key-and-public-key-to-a-file#)
