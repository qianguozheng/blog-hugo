+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2018-05-24T17:04:44+08:00"
menu = "main"
title = "go ethereum crypto private public key"

+++

### 1. 生成Ethereum地址：


```
key, _ := crypto.GenerateKey()

address := crypto.PubkeyToAddress(key.PublicKey).Hex()

privateKey := hex.EncodeToString(key.D.Bytes())
```
### 2. 根据私钥16进制恢复地址
```
key,_ := crypto.HexToECDSA(testPrivHex)
msg := crypto.Keccak256([]byte("foo"))

sig, _ := crypto.Sign(msg, key)  #利用私钥对msg进行签名，同样可以通过签名的msg来恢复公钥，从而恢复地址

recoveredPub, _ := crypto.Ecrecover(msg, sig)
pubKey := crypto.ToECDSAPub(recoveredPub)
recoveredAddr := crypto.PubkeyToAddress(*pubKey)   #common.Address格式的地址
```
* 2.1由此可见， 可以通过利用私钥签名，来推断出私钥对应的公钥，进而推断出地址。

* 2.2 如下方式，也可以获取到公钥和地址
```
recoveredPub2,  _:= crypto.SigToPub(msg, sig)

recoveredAddr2 := crypto.PubkeyToAddress(*recoveredPub2)
```
** Ecrecover 和 SigToPub都可以获得公钥！！ **

