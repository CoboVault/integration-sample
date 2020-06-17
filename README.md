# Cobo Vault integration sample
This repo shows how Cobo Vault can be integrated with a watch-only wallet application. 

Currently, we have two ways to export info of Cobo Vault.

QRCode and microSD-card.

QRCode is used both in All-Coins version and BTC-Only version. 

microSD-card is only used in BTC-Only version.

## QRCode
The data to be transferred is large, 
so we split the data into small fragments can be displayed in single QRCodes 
and combine them by an animated QRCode protocol. 

### All-Coins
This version of QRCode uses a json wrapper to combine the data fragments.

The whole data will be split by:
>split(base64.encode(dataBuffer), capacity)
> 
>or
>
>split(base64.encode(gzip.zip(dataBuffer)), capacity)

One QRCode contains following fields:

```
{
  "total": 5,
  "index": 0,
  "checkSum": "",
  "value": "",
  "compress": true,
  "valueType": "protobuf" 
}
```


>total: identify the length of all fragments
>
>index: identify current fragment index, starts from 0
>
>checksum: a md5 string for whole data
>
>value: current fragment value, it is one part of whole data
>
>compress: whether the process has compressing step;
>
>valueType: would only be 'protobuf'
 

When you collect all the QRCodes, you can extract the `value` and order them by index then concat them into one string, then decode it:

> base64.decode(orderedFragments.join(''))
>
> or
>
> gzip.unzip(base64.decode(orderedFragments.join(''))) 

then you will get a protobuf data which described at [crypto-coin-message](https://github.com/CoboVault/crypto-coin-message-protocol)
just decode it with any protobuf library.

### BTC-only

In this version we follow the [BCR-UR](https://github.com/BlockchainCommons/Research/blob/master/papers/bcr-2020-005-ur.md) format to organize animated QRCodes.

We provide a simple implementation here: [BC-UR](https://github.com/CoboVault/cobo-vault-blockchain-base/tree/master/packages/bc-ur);

You can just collect the animated QRCodes and send the string array to BC-UR's decode method. More details in [BC-UR](https://github.com/CoboVault/cobo-vault-blockchain-base/tree/master/packages/bc-ur);


## microSD card

A wallet json file can be exported into microSD card looks like: 

```json
{
  "ExtPubKey": "",
  "MasterFingerprint": "",
  "CoboVaultFirmwareVersion": "",
  "DerivationPath": ""
}
```
> ExtPubKey: the extended public key of `DeribationPath`
> 
> MasterFingerprint: the master key's fingerprint, big endian
>
> CoboVaultFirmwareVersion: version number in string
>
> DerivationPath: the bip32 derivation path of `ExtPubKey`

All the transactions base on PSBT.

You can write an unsigned PSBT binary file named `xxx.psbt` into microSD card and insert it into Cobo Vault.

Then you can find the file in Cobo Vault and sign it. After that, a `signed-xxx.psbt` would be found in sd card, you can broadcast it in watch-only wallet.  
