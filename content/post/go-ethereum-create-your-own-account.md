+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang", "Ethereum"]
date = "2018-05-23T16:57:20+08:00"
menu = "main"
title = "go ethereum create your own account"

+++

最近两天一直搜索哪里有开源的Go实现的Ethereum的钱包，但是发现其实那么多的开源项目并没有
符合我想看的，兜兜转转，还是看了go-ethereum的源码，然后在stackoverflow上看到了下面的
实现，值得一试。

[stackoverflow](https://ethereum.stackexchange.com/questions/39900/create-ethereum-account-using-golang)

```
package main

import "github.com/ethereum/go-ethereum/crypto"
import "encoding/hex"
import "fmt"

func main() {
	
	//Create an account
	key, err := crypto.GenerateKey()
	if err != nil {
		fmt.Println("Error: ", err.Error());
	}
	
	//Get the address
	address := crypto.PubkeyToAddress(key.PublicKey).Hex()
	fmt.Printf("address[%d][%v]\n", len(address), address);
	
	//Get the private key
	privateKey := hex.EncodeToString(key.D.Bytes())
	fmt.Printf("privateKey[%d][%v]\n", len(privateKey), privateKey);
	
}

```
