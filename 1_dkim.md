# Verifying DKIM Signatures to detect MITM

## DKIM Signatures on Autocrypt Headers

In December 2017 the provider posteo.de announced that they will DKIM sign
Autocrypt headers of outgoing mail. This way the recipient can verify
that the header content has not been altered after leaving the senders
provider.

A valid DKIM signature on the mail headers that includes the Autocrypt
header will therefore indicate that the recipients provider has not
altered the key included in the header.

Since some providers to not use DKIM signatures at all. A missing
signature by itself does not indicate a MITM attack. Some providers also
alter incoming mails to attach mail headers or footers in the message body.
Therefore even a broken signature can have a number of causes.

The DKIM header includes a field 'bh' with the hash of the email body
that was used to calculate the full signature. If the DKIM signature is
broken it may still be possible to verify the headers based on the body
hash and the signed headers.

![Alt text](images/dkim.pdf)

### DKIM Signatures on attached public keys

DKIM signatures also work on attachments. The same mechanism may
therefor be used to verify that public keys attached to emails have not
been altered by the recipients provider.

## Device loss and MITM attacks

Autocrypt specifies to happily accept new keys send in Autocrypt headers
even if a different key was send before. This is meant to prevent
unreadable mail but also offers a larger attack surface for MITM
attacks.

The Autocrypt spec explicitely states that it does not provide
protection against active attacks. However combined with DKIM signatures
at least a basic level of protection can be achieved:

A new key distributed in a mail header with a valid DKIM signature
signals that the key was not altered after the mail left the senders
provider. Therefore the following threats remain:

* the senders device was compromised
* the senders email account was compromised
* the transport layer encryption between the sender and their provider
  was broken
* the senders provider is malicious
* the senders provider was compromised

Please note that all other key distribution schemes that rely on the
provider to certify or distribute the users key share these attack
vectors.

### Key updates in suspicious mails

If the recipient has seen an Autocrypt header with a valid DKIM
signature from the sender before and receives a new key in a mail
without a signature or with a broken signature that may indicate a MITM
attack.

### One malicious provider out of two

In order to carry out a successful transparent MITM attack on a
conversation the attacker needs to replace both parties keys and
intercept all mails. While it's easy for either one of the providers to
intercept all emails replacing the keys in the headers and the
signatures in the body will lead to broken DKIM signatures in one
direction.

### Same provider or two malicious providers

If both providers cooperate on the attack or both users use the same
provider it's easy for the attacker to replace the keys and pgp
signatures on the mails before DKIM signing them.


## Open Questions

### Reliability of DKIM signatures

Key update notifications suffer from a high number of false positives.
Most of the time the key holder just lost their device and reset the
key. How likely is it that for a given sender and recipient DKIM
signatures that used to be valid when receiving the emails stop being
valid? How likely is this to cooccure with the introduction of a new
key? Intuitively both events occuring at the same time seems highly
unlikely. However we need some data on this.

### Provider support

What can providers do?

* DKIM sign autocrypt headers in outgoing mails
* preserve DKIM signed headers in incoming mails

Maybe they can indicate both these properties in a way that can be
checked by the recipients client?
