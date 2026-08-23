# OpenPGP identity for `tinytoast.dev`

The OpenPGP identity for `me@tinytoast.dev` is a modern Ed25519 primary key
with a Curve25519 encryption subkey.

```
6A00 EAB5 2C5F 492C 8A98  1926 ED02 396C 918F 8262
```

The key expires on 2029-08-22 unless it is extended or replaced first.

## Where the public key is published

- Identity page: `https://tinytoast.dev/identity/`
- ASCII-armored key: `https://tinytoast.dev/pgp/me-tinytoast-dev.asc`
- Direct WKD: `https://tinytoast.dev/.well-known/openpgpkey/hu/s8y7oh5xrdpu9psba3i5ntk64ohouhga`
- Advanced WKD files are included at `.well-known/openpgpkey/tinytoast.dev/`.
  They become live after the DNS CNAME below is added.

Trust a downloaded key only after comparing its fingerprint with the identity
page through a channel you trust.

## Sign locally

GnuPG and this key are installed in the local user keyring. Its generated
passphrase is held in macOS Keychain, so use file descriptor 3 to give GnuPG
the passphrase without displaying it. To make a detached, ASCII-armored
signature for a file:

```sh
gpg --batch --pinentry-mode loopback --passphrase-fd 3 --armor --detach-sign \
  --local-user 6A00EAB52C5F492C8A981926ED02396C918F8262 FILE \
  3< <(security find-generic-password -a "$USER" -s 'OpenPGP identity key — tinytoast.dev' -w)
```

This produces `FILE.asc`. To clear-sign short text instead:

```sh
printf '%s\n' 'message to sign' | gpg --batch --pinentry-mode loopback --passphrase-fd 3 \
  --armor --clearsign --local-user 6A00EAB52C5F492C8A981926ED02396C918F8262 \
  3< <(security find-generic-password -a "$USER" -s 'OpenPGP identity key — tinytoast.dev' -w)
```

The private key remains in `~/.gnupg` and must never be copied into this
repository or uploaded to a keyserver.

## Verify as someone else

```sh
curl -fsSLO https://tinytoast.dev/pgp/me-tinytoast-dev.asc
gpg --import me-tinytoast-dev.asc
gpg --fingerprint ED02396C918F8262
# Confirm: 6A00 EAB5 2C5F 492C 8A98 1926 ED02 396C 918F 8262
gpg --verify FILE.asc FILE
```

Clients supporting Web Key Directory can usually discover it with:

```sh
gpg --auto-key-locate clear,wkd --locate-keys me@tinytoast.dev
```

## Secure backups and revocation

The original revocation certificate is local-only at:

```
~/.gnupg/openpgp-revocs.d/6A00EAB52C5F492C8A981926ED02396C918F8262.rev
```

Make an encrypted backup on an offline encrypted drive, not a cloud-synced
folder. Use a new empty directory on that drive, then run:

```sh
gpg --armor --export-secret-keys 6A00EAB52C5F492C8A981926ED02396C918F8262 > private-key-backup.asc
cp ~/.gnupg/openpgp-revocs.d/6A00EAB52C5F492C8A981926ED02396C918F8262.rev revocation-certificate.rev
chmod 600 private-key-backup.asc revocation-certificate.rev
```

Keep that drive physically separate. The exported private key remains
passphrase-protected; its passphrase is in the local macOS Keychain under
`OpenPGP identity key — tinytoast.dev`. Ensure that Keychain item is included
in a protected device backup, or record the passphrase in a separate secure
password manager before relying on the exported key.

To revoke after compromise, remove the safety colons from a *copy* of the
generated certificate, import it, then republish the replacement public key at
the same HTTP and WKD locations:

```sh
sed 's/^://' ~/.gnupg/openpgp-revocs.d/6A00EAB52C5F492C8A981926ED02396C918F8262.rev | gpg --import
```

To rotate normally, generate a new key, publish its new public key and
fingerprint, retain the old key during a transition, and refresh it before its
expiry date.

## DNS

For broad WKD support, add this DNS record (DNS-only, not proxied):

| Type | Name | Target | TTL |
| --- | --- | --- | --- |
| CNAME | `openpgpkey` | `tinytoast.dev.` | Auto / 3600 |

This enables:
`https://openpgpkey.tinytoast.dev/.well-known/openpgpkey/tinytoast.dev/hu/s8y7oh5xrdpu9psba3i5ntk64ohouhga`.

The direct WKD location works on the main host without a DNS change. The CNAME
must serve HTTPS successfully for `openpgpkey.tinytoast.dev`; add that custom
hostname to the static host if it is not provisioned automatically. No DNS
record is required for ordinary signature verification. DNSSEC improves
DNS-based discovery if supported by the registrar, but is optional for WKD.

## Security model

An OpenPGP signature proves that the signer controlled this private key when
they signed, and detects later changes to the signed content. Publishing the
fingerprint at this domain provides a useful independent identity anchor when
the HTTPS site and domain control are trustworthy.

It does not prove the human operator's real-world identity, prevent a
compromised device or domain from publishing a replacement key, guarantee a
message was read by a particular person, or provide anonymity. Verify the
fingerprint through another trusted channel for high-value decisions.
