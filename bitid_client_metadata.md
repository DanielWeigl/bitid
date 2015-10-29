BitID metadata request
=====

## Request URI

If the server wants the client to provide additional metadata with its sign in, it adds an `&s=<Scope>` parameter to the URI, for example:

```
bitid:www.site.com/callback?x=NONCE&s=EBtc
```

The `s` stands for scope and defines which fields the client should ask for.

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
| Profile picture url | P | String, URL |
| Telephone | Tel | String |
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

This means the client should request email, username, a profile picture url and his current bitcoin receiving address. 
The client program can store the users preferred values locally, but should easily enable the user to change or completely clear any of the requested fields.

## Bitcoin Address

If the server requests a bitcoin address (other coins could be added on demand) and the client has wallet capabilities, it should inform the user that the server requests his receiving bitcoin address. It also might provide the option to select an account, if needed.
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
 - calculate the Sha256 hash of the UTF8-NFC representation of this string
 
Instead of signing only the `bitid:`-URI, append the hash separated by a ";" to it and sign it as usual.
 
     Bitcoin Signed Message:
     bitid:www.site.com/callback?x=NONCE;9EDDF573CB509F1F62DF633E25C052AC1B2A0FF9241E70223C77C73E834C0045

## Use cases

### Exchange
	
  - The User pairs his wallet while he is logged in at an exchange. (Normal BitID, no address requested)
  - If the user wants to withdraw funds, the server presents him a BitId-Uri with `&s=Btc`
  - The users scans the request with a BitId-enabled wallet
  - The wallet shows the user the BitId request and informs him that his address will also be sent 
      - (optionally the wallet ask the user to which account he wants the funds to)
  - The wallet signs the BitID challenge and also adds its current receiving address to it
  - The exchange can validate if it is the correct user and also has an individual payout address for each withdraw
  - If everything is valid, it responds positive to the BitId-request and shows the address on the website (in a way, that it can not modified - neither by the user or a trojan on the users machine)
  - The user enters the amount he wants and clicks "withdraw"
   
Pros: 

   - no copy/pasting, typing or mailing of the current receive address
   - no chance that a trojan swaps the receiving address
   - the paired BitId acts as a second factor security check
   

### Voucher / Bitcoin ATM
	
  - A Bitcoin-voucher provider wants to make it easy to redeem his vouchers
  - The Provider puts a QR-Code with the BitID request on it, with an URI that also encodes the redeem 
    - either in the URL or via the nonce
  - A customer scans it with a BitId-enabled wallet
  - The wallet shows the user the BitId request and informs him that his address will also be sent 
      - (optionally the wallet ask the user to which account he wants the funds to)
  - The wallet signs the BitID challenge and also adds its current receiving address to it
  - The Service checks the signature and the redeem code, but ignores the BitID-address as this is neither a known nor will it be a recurring user 
  - If everything is valid, it responds positive to the BitId-request and sends the funds associated with the redeem code to the given address 

  (might also work for Bitcoin ATMs)
  
  Pros: 
  
  - no copy/pasting, typing or mailing of the current receive address
  - no typing of the vouchers website or redeem code 
  - if broadly accepted: the BTM does not need a camera to scan QR-Codes, only the client needs a camera
