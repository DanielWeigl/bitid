<pre>
BIP: TBD
Title: Bitcoin address authentication protocol (BitID)
Author: Eric Larcheveque, @EricLarch
Status: Pre-draft
Type: Process
Created: TBD
</pre>

# Abstract

The following BIP is an open protocol proposal allowing simple and secure 
authentication based on public key cryptography. By authentication we mean 
to prove to a service/application that we control a specific Bitcoin address 
by signing a challenge, and that all related data and settings may securely 
be linked to our session.

# Motivation

Bitcoin related sites and applications shouldn’t have to rely on artificial 
identification methods such as usernames and passwords. Using a wallet for 
authentication purposes has many benefits :

- "one-click" registration and login procedures
- no need to remember or duplicate passwords
- the server only knows and stores the users's Bitcoin public address
- services always know the return address
- optionally, connect to a decentralized identification system in order to populate registration fields (nickname, email ...)

See complete BitID presentation : http://bit.ly/bitid-slides

# Specification

In order to access a restricted area or authenticate oneself against a given service, 
the user is shown the following UX :

![](http://i.imgur.com/CvuXijh.png)

The QR code contains the following data :

```
bitid:www.site.com/callback?x=NONCE
```

- **bitid** is the protocol scheme
- **www.site.com/callback** is the callback URL (https mandatory, cannot have arguments)
- **x** is the NONCE must always be unique, and will be a link to the user's session ID on the site the callback is redirected to.

In order to have a `http` callback, add `&u=1`. This would be recommended for development
purposes only.

The user has to confirm that she wants to authenticate herself on the target website, and has 
to choose which Bitcoin private key will sign the QR code contents.

If the user’s Bitcoin wallet is located on the same computer, a click on the QR code should 
launch the wallet application and supply it with the signature request. If the wallet is located 
on a mobile phone, the user should scan the QR code using the application. In both cases, the 
following dialog options should be shown :

| Step 1 | Step 2 | Step 3 |
|--------|--------|--------|
|![](http://i.imgur.com/6KlZFGe.png)|![](http://i.imgur.com/8ZNMmdp.png)|![](http://i.imgur.com/630hUsu.png)|

After a Bitcoin address is chosen, or created on the fly, the full bitid URI is signed with 
the address’ private key. The signature and public key are then POSTed to the callback url.

**Note :** the signature must comply to the `\x18Bitcoin Signed Message:\n#{message.size.chr}#{message}` format

<pre>
\x18Bitcoin Signed Message:
%bitid:www.site.com/callback?x=NONCE
</pre>

The receiving server verifies the validity of the signature and proceeds to authenticate the user. 
Server-side, only the user's public key is stored. A timeout for the validity of the nonce should 
be implemented by the server in order to prevent replay attacks.

## HD wallet derivation path

For HD wallets, the derivation scheme is based on Trezors [SLIP0013](http://doc.satoshilabs.com/slips/slip-0013.html)

To derive a per service identity, use following scheme:
  
let `a` be the RFC 3986 URI `proto://[user@]host[:port][/path]`
where the BitID response will be send to, but only up to (excluding) the first `?`

Example  
<pre>
  bitid:www.site.com/callback?x=NONCE  -->  https://www.site.com/callback
</pre>

And let `b` be an optional per-service index (32-bit unsigned integer, default=0x00000000)

 1. Concatenate the little endian representation of index with the URI.
 2. Compute the SHA256 hash of the result. `sha256(a | b)`
 3. Let’s take first 128 bits of the hash and split it into four 32-bit numbers `A`, `B`, `C`, `D`.
 4. Set highest bits of numbers `A`, `B`, `C`, `D` to 1 (to enable hardened derivation)
 5. Derive the following [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) Path,
    based on the masterseed `m` with an optional user-provided password `pwd` according to
    [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)

<pre>
(m + pwd)/13’/A’/B’/C’/D’ 
</pre>

The password can be used to have seperated identity accounts based on the same masterseed. If your app does not support more than one identiy or for the default identity the password should be the empty string.

# Test vectors

<pre>
  BITID URL = "bitid:bitcoin.org/login?x=123" 
  MASTERSEED WORDLIST = "abandon, abandon, abandon, abandon, abandon, abandon, abandon, abandon, abandon, abandon, abandon, about"
  
  this results in following:
  a) Without password (PWD="")
    WEBSITE = https://bitcoin.org/login
    A = 0x9D38E2D0, B = 0xA994834C, C = 0xE02DE3F3, D = 0xEEBF0411
    PUBKEY = 0x030a79ba07392dafab29e2bf01917dcb2b1cb235ccad9c7a59639ad0f84c3f619c
    PRODNET ADDRESS = "1LbxwgBqp6VYXfoadiLRVF1jaDxqL4SdRz"
  
  b) With password (PWD="SecondIdentity")
    A = 0x9D38E2D0, B = 0xA994834C, C = 0xE02DE3F3, D = 0xEEBF0411
    PUBKEY = 0x0265da9147121706403032fb22107206b0c510de65a19711eca5781edf67639598
    PRODNET ADDRESS = "11XiTMf6dULM8Uk7QohJMDEvdW6Lqy2gG"
</pre>

# Rationale

Classical password authentication is an insecure process that could be solved with public key cryptography. 
The problem however is that it theoretically offloads a lot of complexity and responsibility on the user. 
Managing private keys securely is complex. However this complexity is already being addressed in the 
Bitcoin ecosystem. So doing public key authentication is practically a free lunch to bitcoiners.

# Backward compatibility

Since not all wallets will provide support for the proposed `bitid` scheme, 
a manual challenge is also possible :

![](http://i.imgur.com/Giz0fGQ.png)

All wallets (including Bitcoin Core) provide manual signing capabilities, therefore any user may use the 
BitID protocol at the expense of the UX.

# Reference implementation

A demonstration of the workflow is available here :  
http://bitid.bitcoin.blue

Source code of the demonstration service :  
https://github.com/bitid/bitid-demo

Ruby implementation :  
https://github.com/bitid/bitid-ruby

Python implementation :  
https://github.com/LaurentMT/pybitid

Javascript implementation :  
https://github.com/porkchop/bitid-js  

PHP implementation :  
https://github.com/conejoninja/bitid-php  

Android Bitcoin wallet fork including support of BitID :  
https://github.com/bitid/bitcoin-wallet

# See also

Reddit thread on the need of such a protocol :
http://www.reddit.com/r/Bitcoin/comments/1nkoju/bitcoin_core_dev_websites_do_not_need_passwords/

SQRL, a similar proposal not limited to Bitcoin :
https://www.grc.com/sqrl/sqrl.htm
