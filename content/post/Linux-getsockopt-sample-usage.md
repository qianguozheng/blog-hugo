+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2017-04-26T16:24:12+08:00"
menu = "main"
title = "Linux getsockopt sample usage"

+++

# Linux getsockopt sample usage

```
	#include <linux/netfilter_ipv4.h> //SO_ORIGINAL_DST
	//Get the target ip and port address
	 int status;
	 socklen_t destlen;
	 struct sockaddr_in destaddr;
	 char destinationName[256];
	 int port;
	 memset(destinationName, 0, 256);
	 
	 status = getsockopt(afd, IPPROTO_IP, SO_ORIGINAL_DST, (struct sockaddr *) &destaddr, &destlen);
	 inet_ntop(AF_INET, (void *)&destaddr.sin_addr, destinationName, sizeof(destinationName));
	 port = ntohs(destaddr.sin_port);
	 
```

    这里的SO_ORIGINAL_DST是用于netfilter REDIRECT target重定向过的数据包中的。
    
    
