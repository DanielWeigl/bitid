BitID metadata request
=====

## Request URI

If the server wants the client to provide additional metadata with its sign in, it adds an `&s=<Scope>` parameter to the URI

e.g.

```
bitid:www.site.com/callback?x=NONCE&s=EBtc
```

The `s` stands for scope and defines which fields the client should ask for

# Scope


| Field | Permission scope | Format 
| --- | --- | --- |
| Email | E | String |
| Username | U | String |
| First name | N | String |
| Last name | L | String |
| Date of birth  | D | Long, Epoch |
| Gender | G | String, [male/female/other/na] |
| Twitter | T | String |
| Facebook | F | String |
| Homepage | H | String |
| Profile picture url | Pic | String, URL |
| Address Line 1 | A1 | String |
| Address Line 2 | A2 | String |
| City | C | String  |
| State / Province | S | String |
| Postal Code | Z | String |
| Country | Y | String |
| Bitcoin Address  | Btc | String |

Various fields can be requested via the `s`-Parameter by concatenating them, for example:

```
bitid://bitid.bitcoin.blue/callback?x=NONCE&s=EUPicBtc
```

This would request email, username and profile picture url and his current Bitcoin receiving address. 

The client program can store the users preferred values locally, but should easily enable the user to change or completely clear any of the requested fields.

## Bitcoin Address

If the server requests an Bitcoin address (other coins could be added on demand) and the client has wallet capabilities, it should the user inform that the server requests his receiving bitcoin address and might provide the option to select an account, if there is more than one available.
The wallet then automatically fills this field with an valid address where the service might later send funds to.

## Request and Signing

All fields that got set by the client get combined in a JSON-Object with the scope-id as key and the value provided by the user and added to the sign-in POST-request

```
{
  'uri': 'bitid:www.site.com/callback?x=NONCE',
  'address': '1J34vj4wowwPYafbeibZGht3zy3qERoUM1',
  'metadata': {
    'E': 'satoshi@gmx.de',
    'U': 'satoshi',
    'D': 1231002905
    'Btc': '1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa'
  },
  'signature': ''
}
```

To sign the metadata, all fields get concatenated in following way:

 - sort all key-value pairs get by it scope-id, ascending
 - write all fields as string 
     - `string` keep as they are
     - `long` gets converted to its decimal representation without any leading zeros
 - and concatenate them following way: `<ScopeId>|<Value>|<ScopeId>|<Value>|....`
 - calculate the Sha256 hash of the UTF8 representation of this string
 
Instead of signing only the `bitid:`-URI append this hash, seperated by an ";" to it and sign it as normal.
 
     Bitcoin Signed Message:
     bitid:www.site.com/callback?x=NONCE;9EDDF573CB509F1F62DF633E25C052AC1B2A0FF9241E70223C77C73E834C0045

