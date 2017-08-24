# DHCP
[TOC]
##一. DHCP源码&报文分析
###1. 服务器
服务器处理6种请求，分别是：
**TODO**
####1.1 DHCPDISCOVER
#####1.1.1 find_lease
1. client ip地址为空则分配ip，否则查找报文中的requested address
2. 不知道在干啥
3. 查找client identifier是否已经绑定其他的主机.

```
 Option: (61) Client identifier
     Length: 7
     Hardware type: Ethernet (0x01)
     Client MAC address: HewlettP_62:02:4e (ec:b1:d7:62:02:4e)
```
  通过uid ---> 通过mac地址 ----> 通过host-identifier option

  如果收到的是DHCPREQUEST请求并且在租约中的ip地址和requested address不同，就返回错误

```c
/* If fixed_lease is present but does not match the requested
   IP address, and this is a DHCPREQUEST, then we can't return
   any other lease, so we might as well return now. */
if (packet -> packet_type == DHCPREQUEST && fixed_lease &&
    (fixed_lease -> ip_addr.len != cip.len ||
     memcmp (fixed_lease -> ip_addr.iabuf,
	     cip.iabuf, cip.len))) {
	if (ours)
		*ours = 1;
	strcpy (dhcp_message, "requested address is incorrect");
	goto out;
	}
```

4. 找到匹配的lease之后验证lease的合法性

#####1.1.2 没有找到已有的lease，分配新的 allocate_lease
遍历地址池，首先寻找没有使用过的地址----> 找能用的地址 ----> 重启一个已经废弃的地址 ----> 不分配地址,当有两个状态相同的候选地址的时候选择先过期的.
将租期设置为至少两分钟

```c
when = cur_time + 120;
if (when < lease -> ends)
	when = lease -> ends;
```

#####1.1.3 ack_lease
**TODO**

####1.2 DHCPREQUEST
#####1.2.1 查找REQUESTED_ADDRESS
```c
//#define DHO_DHCP_REQUESTED_ADDRESS 50
oc = lookup_option (&dhcp_universe, packet -> options,
			    DHO_DHCP_REQUESTED_ADDRESS);
```
```
Option: (50) Requested IP Address
    Length: 4
    Requested IP Address: 192.168.2.198
```

如果REQUESTED_ADDRESS不可用就分配0.0.0.0

```c
	if (oc &&
	    evaluate_option_cache (&data, packet, (struct lease *)0,
				   (struct client_state *)0,
				   packet -> options, (struct option_state *)0,
				   &global_scope, oc, MDL)) {
		cip.len = 4;
		memcpy (cip.iabuf, data.data, 4);
		data_string_forget (&data, MDL);
		have_requested_addr = 1;
	} else {
		oc = (struct option_cache *)0;
		cip.len = 4;
		memcpy (cip.iabuf, &packet -> raw -> ciaddr.s_addr, 4);
	}

```

#####1.2.2 查找REQUESTED_ADDRESS对应的lease

```c
	subnet = (struct subnet *)0;
	lease = (struct lease *)0;
	if (find_subnet (&subnet, cip, MDL))
		find_lease (&lease, packet,
			    subnet -> shared_network, &ours, 0, ip_lease, MDL);

	if (lease && lease -> client_hostname) {
		if ((strlen (lease -> client_hostname) <= 64) &&
		    db_printable((unsigned char *)lease->client_hostname))
			s = lease -> client_hostname;
		else
			s = "Hostname Unsuitable for Printing";
	} else
		s = (char *)0;
```
#####1.2.3 检查服务器标志

```c
	oc = lookup_option (&dhcp_universe, packet -> options,
			    DHO_DHCP_SERVER_IDENTIFIER);
	memset (&data, 0, sizeof data);
	if (oc &&
	    evaluate_option_cache (&data, packet, (struct lease *)0,
				   (struct client_state *)0,
				   packet -> options, (struct option_state *)0,
				   &global_scope, oc, MDL)) {
		sip.len = 4;
		memcpy (sip.iabuf, data.data, 4);
		data_string_forget (&data, MDL);
		/* piaddr() should not return more than a 15 byte string.
		 * safe.
		 */
		sprintf (smbuf, " (%s)", piaddr (sip));
		have_server_identifier = 1;
	} else
		smbuf [0] = 0;
```
```
Option: (54) DHCP Server Identifier
    Length: 4
    DHCP Server Identifier: 192.168.2.1
```

####1.3 DHCPRELEASE
释放已经获取的ip地址
#####1.3.1 通过CLIENT_IDENTIFIER标签查找lease
```
Option: (61) Client identifier
    Length: 7
    Hardware type: Ethernet (0x01)
    Client MAC address: HewlettP_62:02:4e (ec:b1:d7:62:02:4e)

```

```c
	oc = lookup_option (&dhcp_universe, packet -> options,
			    DHO_DHCP_CLIENT_IDENTIFIER);
	memset (&data, 0, sizeof data);
	if (oc &&
	    evaluate_option_cache (&data, packet, (struct lease *)0,
				   (struct client_state *)0,
				   packet -> options, (struct option_state *)0,
				   &global_scope, oc, MDL)) {
		find_lease_by_uid (&lease, data.data, data.len, MDL);
		data_string_forget (&data, MDL);

		/* See if we can find a lease that matches the IP address
		   the client is claiming. */
		while (lease) {
			if (lease -> n_uid)
				lease_reference (&next, lease -> n_uid, MDL);
			if (!memcmp (&packet -> raw -> ciaddr,
				     lease -> ip_addr.iabuf, 4)) {
				break;
			}
			lease_dereference (&lease, MDL);
			if (next) {
				lease_reference (&lease, next, MDL);
				lease_dereference (&next, MDL);
			}
		}
		if (next)
			lease_dereference (&next, MDL);
	}
```
#####1.3.2 如果查找失败通过clent ip查找
```c
	if (!lease) {
		cip.len = 4;
		memcpy (cip.iabuf, &packet -> raw -> ciaddr, 4);
		find_lease_by_ip_addr (&lease, cip, MDL);
	}
```
#####1.3.3 检查mac地址时候匹配，仅在地址匹配情况下release

####1.4 DHCPDECLINE
PC收到DHCP服务器的地址后，发送分配地址免费ARP，如果有回应，会发送DHCP DECLINE报文
#####1.4.1 必须指明特定的地址

```c
	/* DHCPDECLINE must specify address. */
	if (!(oc = lookup_option (&dhcp_universe, packet -> options,
				  DHO_DHCP_REQUESTED_ADDRESS)))
		return;
```
#####1.4.2 如果找到对应的lease，标记为unusable
```c
if (lease) {
	#if defined (FAILOVER_PROTOCOL)
		if (lease -> pool && lease -> pool -> failover_peer) {
		    dhcp_failover_state_t *peer =
			    lease -> pool -> failover_peer;
		    if (peer -> service_state == not_responding ||
			peer -> service_state == service_startup) {
			if (!ignorep)
			    log_info ("%s: ignored%s",
				      peer -> name, peer -> nrr);
			goto out;
		    }

		    /* DHCPDECLINE messages are broadcast, so we can safely
		       ignore the DHCPDECLINE if the peer has the lease.
		       XXX Of course, at this point that information has been
		       lost. */
		}
#endif

		abandon_lease (lease, "declined.");
		status = "abandoned";
	    } else {
		status = "not found";
	    }
```

```c
/* Abandon the specified lease (set its timeout to infinity and its
   particulars to zero, and re-hash it as appropriate. */
void abandon_lease (lease, message)
	struct lease *lease;
	const char *message;
```
####1.5 DHCPINFORM
PC单独请求域名、DNS这些参数的时候。

```c
	/* Use the subnet mask from the subnet declaration if no other
	   mask has been provided. */
	i = DHO_SUBNET_MASK;
	if (subnet && !lookup_option (&dhcp_universe, options, i)) {
		oc = (struct option_cache *)0;
		if (option_cache_allocate (&oc, MDL)) {
			if (make_const_data (&oc -> expression,
					     subnet -> netmask.iabuf,
					     subnet -> netmask.len,
					     0, 0, MDL)) {
				option_code_hash_lookup(&oc->option,
							dhcp_universe.code_hash,
							&i, 0, MDL);
				save_option (&dhcp_universe, options, oc);
			}
			option_cache_dereference (&oc, MDL);
		}
	}
```

```c
/* Use the parameter list from the scope if there is one. */
oc = lookup_option (&dhcp_universe, options,
			    DHO_DHCP_PARAMETER_REQUEST_LIST);
```

```
Option: (55) Parameter Request List
    Length: 13
    Parameter Request List Item: (1) Subnet Mask
    Parameter Request List Item: (15) Domain Name
    Parameter Request List Item: (3) Router
    Parameter Request List Item: (6) Domain Name Server
    Parameter Request List Item: (44) NetBIOS over TCP/IP Name Server
    Parameter Request List Item: (46) NetBIOS over TCP/IP Node Type
    Parameter Request List Item: (47) NetBIOS over TCP/IP Scope
    Parameter Request List Item: (31) Perform Router Discover
    Parameter Request List Item: (33) Static Route
    Parameter Request List Item: (121) Classless Static Route
    Parameter Request List Item: (249) Private/Classless Static Route (Microsoft)
    Parameter Request List Item: (43) Vendor-Specific Information
    Parameter Request List Item: (252) Private/Proxy autodiscovery
```

####1.6 DHCPLEASEQUERY
**TODO**

###2. 客户端
dhcpv4_client_assignments
确定src port和dst port

```c
/* If we're faking a relay agent, and we're not using loopback,
   use the server port, not the client port. */
if (mockup_relay && giaddr.s_addr != htonl (INADDR_LOOPBACK)) {
	local_port = htons(67);
} else {
	ent = getservbyname ("dhcpc", "udp");
	if (!ent)
		local_port = htons (68);
	else
		local_port = ent -> s_port;

/* If we're faking a relay agent, and we're not using loopback,
   we're using the server port, not the client port. */
if (mockup_relay && giaddr.s_addr != htonl (INADDR_LOOPBACK)) {
	remote_port = local_port;
} else
	remote_port = htons (ntohs (local_port) - 1);   /* XXX */
}
```
####2.1 discover

#####2.1.1 options
1 Send the server identifier if provided.

```c
	if (sid)
		save_option (&dhcp_universe, *op, sid);
```

2 Send the requested address if provided.

```c
			if (rip) {
		client -> requested_address = *rip;
		i = DHO_DHCP_REQUESTED_ADDRESS;
		if (!(option_code_hash_lookup(&option, dhcp_universe.code_hash,
					      &i, 0, MDL) &&
		      make_const_option_cache(&oc, NULL, rip->iabuf, rip->len,
					      option, MDL)))
			log_error ("can't make requested address cache.");
		else {
			save_option (&dhcp_universe, *op, oc);
			option_cache_dereference (&oc, MDL);
		}
		option_dereference(&option, MDL);
	} else {
		client -> requested_address.len = 0;
	}
```

```
Option: (50) Requested IP Address
    Length: 4
    Requested IP Address: 192.168.2.198

```
3 DHCP_MESSAGE_TYPE一定会包括

```c
	i = DHO_DHCP_MESSAGE_TYPE;
	if (!(option_code_hash_lookup(&option, dhcp_universe.code_hash, &i, 0,
				      MDL) &&
	      make_const_option_cache(&oc, NULL, type, 1, option, MDL)))
		log_error ("can't make message type.");
	else {
		save_option (&dhcp_universe, *op, oc);
		option_cache_dereference (&oc, MDL);
	}
```

```
Option: (53) DHCP Message Type (Discover)
    Length: 1
    DHCP: Discover (1)
```

4 DHCP_PARAMETER_REQUEST_LIST

```c
unsigned code = DHO_DHCP_PARAMETER_REQUEST_LIST;
len = 0;
for (i = 0 ; prl[i] != NULL ; i++)
	if (prl[i]->universe == &dhcp_universe)
		bp->data[len++] = prl[i]->code;

if (!(option_code_hash_lookup(&option,
			      dhcp_universe.code_hash,
			      &code, 0, MDL) &&
      make_const_option_cache(&oc, &bp, NULL, len,
			      option, MDL)))
	log_error ("can't make option cache");
else {
	save_option (&dhcp_universe, *op, oc);
	option_cache_dereference (&oc, MDL);
}
option_dereference(&option, MDL);
```

```
Option: (55) Parameter Request List
    Length: 12
    Parameter Request List Item: (1) Subnet Mask
    Parameter Request List Item: (15) Domain Name
    Parameter Request List Item: (3) Router
    Parameter Request List Item: (6) Domain Name Server
    Parameter Request List Item: (44) NetBIOS over TCP/IP Name Server
    Parameter Request List Item: (46) NetBIOS over TCP/IP Node Type
    Parameter Request List Item: (47) NetBIOS over TCP/IP Scope
    Parameter Request List Item: (31) Perform Router Discover
    Parameter Request List Item: (33) Static Route
    Parameter Request List Item: (121) Classless Static Route
    Parameter Request List Item: (249) Private/Classless Static Route (Microsoft)
    Parameter Request List Item: (43) Vendor-Specific Information

```

#####2.1.2 packet内容构造

```c
	client -> packet.op = BOOTREQUEST;
	client -> packet.htype = client -> interface -> hw_address.hbuf [0];
	client -> packet.hlen = client -> interface -> hw_address.hlen - 1;
	client -> packet.hops = 0;
	client -> packet.xid = random ();
	client -> packet.secs = 0; /* filled in by send_discover. */

	if (can_receive_unicast_unconfigured (client -> interface))
		client -> packet.flags = 0;
	else
		client -> packet.flags = htons (BOOTP_BROADCAST);
```

```
Bootstrap Protocol (Discover)
    Message type: Boot Request (1) //packet.op
    Hardware type: Ethernet (0x01) //packet.htype
    Hardware address length: 6     //packet.hlen
    Hops: 0 						     //packet.hops
    Transaction ID: 0x32fbf6d2     //packet.xid
    Seconds elapsed: 0 			    //packet.secs
    Bootp flags: 0x8000, Broadcast flag (Broadcast) //client -> packet.flags
    	1... .... .... .... = Broadcast flag: Broadcast
    	.000 0000 0000 0000 = Reserved flags: 0x0000
```

```c
memset (&(client -> packet.ciaddr),0, sizeof client -> packet.ciaddr);
memset (&(client -> packet.yiaddr),0, sizeof client -> packet.yiaddr);
memset (&(client -> packet.siaddr),0, sizeof client -> packet.siaddr);
client -> packet.giaddr = giaddr;
if (client -> interface -> hw_address.hlen > 0)
    memcpy (client -> packet.chaddr,&client -> interface -> hw_address.hbuf [1], (unsigned)(client -> interface -> hw_address.hlen - 1));
```

```
Client IP address: 0.0.0.0 //client -> packet.ciaddr
Your (client) IP address: 0.0.0.0 //client -> packet.yiaddr
Next server IP address: 0.0.0.0 //client -> packet.siaddr
Relay agent IP address: 0.0.0.0 //client ->packet.giaddr
Client MAC address: HewlettP_62:02:4e (ec:b1:d7:62:02:4e) //client -> packet.chaddr
```
####2.2 request
#####2.2.1 options
同discover阶段
#####2.2.2 填充其他字段

```c
	client -> packet.op = BOOTREQUEST;
	client -> packet.htype = client -> interface -> hw_address.hbuf [0];
	client -> packet.hlen = client -> interface -> hw_address.hlen - 1;
	client -> packet.hops = 0;
	client -> packet.xid = client -> xid;
	client -> packet.secs = 0; /* Filled in by send_request. */
```

```
Bootstrap Protocol (Request)
    Message type: Boot Request (1) //client -> packet.op
    Hardware type: Ethernet (0x01) //client -> packet.htype
    Hardware address length: 6 //client -> packet.hlen
    Hops: 0 //client -> packet.hops
    Transaction ID: 0x32fbf6d2 //client -> packet.xid
    Seconds elapsed: 0 //client -> packet.secs

```

If we own the address we're requesting, put it in ciaddr;
otherwise set ciaddr to zero.

```c
	if (client -> state == S_BOUND ||client -> state == S_RENEWING ||
	    client -> state == S_REBINDING) {
		memcpy (&client -> packet.ciaddr,lease -> address.iabuf, lease -> address.len);
		client -> packet.flags = 0;
	} else {
		memset (&client -> packet.ciaddr, 0,sizeof client -> packet.ciaddr);
		if (can_receive_unicast_unconfigured (client -> interface))
			client -> packet.flags = 0;
		else
			client -> packet.flags = htons (BOOTP_BROADCAST);
	}
```

```
Client IP address: 0.0.0.0 //client -> packet.ciaddr
Bootp flags: 0x8000, Broadcast flag (Broadcast) //client -> packet.flags
   1... .... .... .... = Broadcast flag: Broadcast
   .000 0000 0000 0000 = Reserved flags: 0x0000
```

```c
	memset (&client -> packet.yiaddr, 0,sizeof client -> packet.yiaddr);
	memset (&client -> packet.siaddr, 0,sizeof client -> packet.siaddr);
	if (client -> state != S_BOUND &&client -> state != S_RENEWING)
		client -> packet.giaddr = giaddr;
	else
		memset (&client -> packet.giaddr, 0,sizeof client -> packet.giaddr);
	if (client -> interface -> hw_address.hlen > 0)
	    memcpy (client -> packet.chaddr,&client -> interface -> hw_address.hbuf [1],
		    (unsigned)(client -> interface -> hw_address.hlen - 1));
```

```
Bootstrap Protocol (Request)
    Your (client) IP address: 0.0.0.0 //client -> packet.yiaddr
    Next server IP address: 0.0.0.0 //client -> packet.siaddr
    Relay agent IP address: 0.0.0.0 //client -> packet.giaddr
    Client MAC address: HewlettP_62:02:4e (ec:b1:d7:62:02:4e) //client -> packet.chaddr
```

####2.3 decline
#####2.3.1 options填充
同上
#####2.3.2 其他选项
同上
但是，ciaddr must always be zero

```c
	memset (&client -> packet.ciaddr, 0,
		sizeof client -> packet.ciaddr);
```
####2.4 release
同上

##二 Dhcp协议
###2.1 典型流程
####2.1.1 dhcpdicover
客户端基于UDP的源端口号68，目的端口号67，广播dhcpdiscover报文
客户机开始，可能以0.0.0.0源地址开始，携带null ciaddr/broadcast;xid;params request
####2.1.2 offer
服务器收到请求后会它从尚未出租的IP地址中挑选一个分配给DHCP客户机，在本地形成一个绑定。
	（可以单播可以广播，取决于DHCP报文中的flag位）收到后发送offer报文
broadcast；siaddr,xid,yiaddr,options
####2.1.3 request
客户端接受收到的第一个offer报文，广播request报文，报文中包含指定的服务器信息
####2.1.4 应答
服务器检查收到的信息，发送ack确认（可以单播可以广播，取决于DHCP报文中的flag位），或者发送nak拒绝
#### 2.1.5 验证配置（可选）
客户端检查冲突（arp/acd)，如果冲突就拒绝

###2.2 重新登录
就不需要再发送DHCP discover发现信息了，而是直接发送包含前一次所分配的IP地址的DHCP request请求信息。

当DHCP服务器收到这一信息后，它会尝试让DHCP客户机继续使用原来的IP地址，并回答一个DHCP ACK确认信息。

如果此IP地址已无法再分配给原来的DHCP客户机使用时（比如此IP地址已分配给其它DHCP客户机使用），则DHCP服务器给DHCP客户机回答一个DHCP NAK否认信息。

当原来的DHCP客户机收到此DHCP NAK否认信息后，它就必须重新发送DHCP discover发现信息来请求新的IP地址。

###2.3 更新租约
当DHCP客户机启动时和IP租约期限过一半时，DHCP客户机都会自动向DHCP服务器发送单播DHCP REQUEST更新其IP租约的信息，当DHCP客户机启动时和IP租约期限过87.5%时，DHCP客户机都会自动向DHCP服务器发送广播DHCP REQUEST更新其IP租约的信息。

收到DHCP ACK就续期，收到DHCP NAK就直接发送DHCP RELESE报文释放IP地址，然后开始重新一轮的DHCP




