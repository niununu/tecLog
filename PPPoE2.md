<center>**pppoe源码分析**</center>

[TOC]

# 1. 发现阶段
发现阶段是无状态的，目的是获得PPPoE终结端（在局端的ADSL设备上）的以太网MAC地址，并建立一个惟一的PPPoE SESSION-ID。

host -----------PADI-----------------> server

host <-----------PADO----------------- server

host -----------PADR-----------------> server

host <-----------PADS----------------- server

## 1.1 PADI，active discovery initiation
1.主机广播发起分组，分组的目的地址为以太网的广播地址 0xffffffffffff

```c
    /* Set destination to Ethernet broadcast address */
    memset(packet.ethHdr.h_dest, 0xFF, ETH_ALEN);
```

```
Ethernet II, Src: Dell_49:3f:5f (24:b6:fd:49:3f:5f), Dst: Broadcast (ff:ff:ff:ff:ff:ff)
    Destination: Broadcast (ff:ff:ff:ff:ff:ff)
    Source: Dell_49:3f:5f (24:b6:fd:49:3f:5f)
    Type: PPPoE Discovery (0x8863)
```

2.CODE（代码）字段值为0×09（PADI Code），SESSION-ID（会话ID）字段值为0x0000。

```c
    packet.ethHdr.h_proto = htons(Eth_PPPOE_Discovery);
    //#define ETH_PPPOE_DISCOVERY 0x8863
    packet.vertype = PPPOE_VER_TYPE(1, 1);
    //#define PPPOE_VER_TYPE(v, t)    (((v) << 4) | (t))
    packet.code = CODE_PADI;
    //#define CODE_PADI           0x09
    packet.session = 0;
```

生成hostUniq

```c
    /* If we're using Host-Uniq, copy it over */
if (conn->hostUniq) {
    PPPoETag hostUniq;
    int len = (int) strlen(conn->hostUniq);
    hostUniq.type = htons(TAG_HOST_UNIQ);
    hostUniq.length = htons(len);
    memcpy(hostUniq.payload, conn->hostUniq, len);
    CHECK_ROOM(cursor, packet.payload, len + TAG_HDR_SIZE);
    memcpy(cursor, &hostUniq, len + TAG_HDR_SIZE);
    cursor += len + TAG_HDR_SIZE;
    plen += len + TAG_HDR_SIZE;
}
```

```
PPP-over-Ethernet Discovery
    0001 .... = Version: 1
    .... 0001 = Type: 1
    Code: Active Discovery Initiation (PADI) (0x09)
    Session ID: 0x0000
    Payload Length: 20
    PPPoE Tags
        Host-Uniq: 010000000000000001000000
```

3.PADI分组必须至少包含一个服务名称类型的标签（Service Name Tag，字段值为0x0101），向接入集中器提出所要求提供的服务。

```c
if (!omit_service_name) {
    plen = TAG_HDR_SIZE + namelen;
    CHECK_ROOM(cursor, packet.payload, plen);

    svc->type = TAG_SERVICE_NAME;
    svc->length = htons(namelen);

    if (conn->serviceName) {
        memcpy(svc->payload, conn->serviceName, strlen(conn->serviceName));
    }
    cursor += namelen + TAG_HDR_SIZE;
}
else{
    plen = 0;
}
```

## 1.2 PADO, active discovery offer
1.接入集中器收到在服务范围内的PADI分组，发送PPPoE有效发现提供包分组，以响应请求。

- 服务器：pppoe-server.c processPADI
可接受的tag：

```c
    PPPoETag acname;
    PPPoETag servname;
    PPPoETag cookie;
```

2.其中CODE字段值为0×07（PADO Code），SESSION-ID字段值仍为0x0000。

```
PPP-over-Ethernet Discovery
    0001 .... = Version: 1
    .... 0001 = Type: 1
    Code: Active Discovery Offer (PADO) (0x07)
    Session ID: 0x0000
    Payload Length: 50
    PPPoE Tags
        Host-Uniq: 010000000000000001000000
        AC-Name: SH-SH-SJCX-BAS-1.MAN.ME60X
```

```c
 /* Generate a cookie */
    cookie.type = htons(TAG_AC_COOKIE);
    cookie.length = htons(COOKIE_LEN);
    genCookie(packet->ethHdr.h_source, myAddr, CookieSeed, cookie.payload);
/**********************************************************************
*%FUNCTION: genCookie
*%ARGUMENTS:
* peerEthAddr -- peer Ethernet address (6 bytes)
* myEthAddr -- my Ethernet address (6 bytes)
* seed -- random cookie seed to make things tasty (16 bytes)
* cookie -- buffer which is filled with server PID and
*           md5 sum of previous items
*%RETURNS:
* Nothing
*%DESCRIPTION:
* Forms the md5 sum of peer MAC address, our MAC address and seed, useful
* in a PPPoE Cookie tag.
***********************************************************************/
```

```c
/* Construct a PADO packet */
//包头
memcpy(pado.ethHdr.h_dest, packet->ethHdr.h_source, ETH_ALEN);
memcpy(pado.ethHdr.h_source, myAddr, ETH_ALEN);
pado.ethHdr.h_proto = htons(Eth_PPPOE_Discovery);
pado.ver = 1;
pado.type = 1;
pado.code = CODE_PADO;
pado.session = 0;
```

3.PADO分组必须包含一个接入集中器名称类型的标签（Access Concentrator Name Tag，字段值为0x0102),以及一个或多个服务名称类型标签，表明可向主机提供的服务种类。

```c
    plen = TAG_HDR_SIZE + acname_len;

    CHECK_ROOM(cursor, pado.payload, acname_len+TAG_HDR_SIZE);
    memcpy(cursor, &acname, acname_len + TAG_HDR_SIZE);
    cursor += acname_len + TAG_HDR_SIZE;

    /* If we asked for an MTU, handle it */
    if (max_ppp_payload > ETH_PPPOE_MTU && ethif->mtu > 0) {
    /* Shrink payload to fit */
    if (max_ppp_payload > ethif->mtu - TOTAL_OVERHEAD) {
        max_ppp_payload = ethif->mtu - TOTAL_OVERHEAD;
    }
    if (max_ppp_payload > ETH_JUMBO_LEN - TOTAL_OVERHEAD) {
        max_ppp_payload = ETH_JUMBO_LEN - TOTAL_OVERHEAD;
    }
    if (max_ppp_payload > ETH_PPPOE_MTU) {
        PPPoETag maxPayload;
        UINT16_t mru = htons(max_ppp_payload);
        maxPayload.type = htons(TAG_PPP_MAX_PAYLOAD);
        maxPayload.length = htons(sizeof(mru));
        memcpy(maxPayload.payload, &mru, sizeof(mru));
        CHECK_ROOM(cursor, pado.payload, sizeof(mru) + TAG_HDR_SIZE);
        memcpy(cursor, &maxPayload, sizeof(mru) + TAG_HDR_SIZE);
        cursor += sizeof(mru) + TAG_HDR_SIZE;
        plen += sizeof(mru) + TAG_HDR_SIZE;
    }
    }
    /* If no service-names specified on command-line, just send default
       zero-length name.  Otherwise, add all service-name tags */
    servname.type = htons(TAG_SERVICE_NAME);
    if (!NumServiceNames) {
    servname.length = 0;
    CHECK_ROOM(cursor, pado.payload, TAG_HDR_SIZE);
    memcpy(cursor, &servname, TAG_HDR_SIZE);
    cursor += TAG_HDR_SIZE;
    plen += TAG_HDR_SIZE;
    } else {
    for (i=0; i<NumServiceNames; i++) {
        int slen = strlen(ServiceNames[i]);
        servname.length = htons(slen);
        CHECK_ROOM(cursor, pado.payload, TAG_HDR_SIZE+slen);
        memcpy(cursor, &servname, TAG_HDR_SIZE);
        memcpy(cursor+TAG_HDR_SIZE, ServiceNames[i], slen);
        cursor += TAG_HDR_SIZE+slen;
        plen += TAG_HDR_SIZE+slen;
    }
    }

    CHECK_ROOM(cursor, pado.payload, TAG_HDR_SIZE + COOKIE_LEN);
    memcpy(cursor, &cookie, TAG_HDR_SIZE + COOKIE_LEN);
    cursor += TAG_HDR_SIZE + COOKIE_LEN;
    plen += TAG_HDR_SIZE + COOKIE_LEN;

    if (relayId.type) {
    CHECK_ROOM(cursor, pado.payload, ntohs(relayId.length) + TAG_HDR_SIZE);
    memcpy(cursor, &relayId, ntohs(relayId.length) + TAG_HDR_SIZE);
    cursor += ntohs(relayId.length) + TAG_HDR_SIZE;
    plen += ntohs(relayId.length) + TAG_HDR_SIZE;
    }
    if (hostUniq.type) {
    CHECK_ROOM(cursor, pado.payload, ntohs(hostUniq.length)+TAG_HDR_SIZE);
    memcpy(cursor, &hostUniq, ntohs(hostUniq.length) + TAG_HDR_SIZE);
    cursor += ntohs(hostUniq.length) + TAG_HDR_SIZE;
    plen += ntohs(hostUniq.length) + TAG_HDR_SIZE;
    }
```

## 1.3 RADR, active discovery request
主机在可能收到的多个PADO分组中选择一个合适的PADO分组，然后向所选择的接入集中器发送PPPoE有效发现请求分组。

1.其中CODE字段为0x19（PADR Code），SESSION_ID字段值仍为0x0000。

```c
    memcpy(packet.ethHdr.h_dest, conn->peerEth, ETH_ALEN);
    memcpy(packet.ethHdr.h_source, conn->myEth, ETH_ALEN);

    packet.ethHdr.h_proto = htons(Eth_PPPOE_Discovery);
    packet.vertype = PPPOE_VER_TYPE(1, 1);
    packet.code = CODE_PADR;
    packet.session = 0;
```

```
PPP-over-Ethernet Discovery
    0001 .... = Version: 1
    .... 0001 = Type: 1
    Code: Active Discovery Request (PADR) (0x19)
    Session ID: 0x0000
    Payload Length: 20
    PPPoE Tags
        Host-Uniq: 010000000000000002000000
```

2.PADR分组必须包含一个服务名称类型标签，确定向接入集线器（或交换机）请求的服务种类。

```c
    svc->type = TAG_SERVICE_NAME;
    svc->length = htons(namelen);
    if (conn->serviceName) {
    memcpy(svc->payload, conn->serviceName, namelen);
    }
    cursor += namelen + TAG_HDR_SIZE;
```

3.当主机在指定的时间内没有接收到PADO，它应该重新发送它的PADI分组，并且加倍等待时间，这个过程会被重复期望的次数。

```c
 do {
        padiAttempts++;
        if (padiAttempts > MAX_PADI_ATTEMPTS) {
            warn("Timeout waiting for PADO packets");
            close(conn->discoverySocket);
            conn->discoverySocket = -1;
            return;
        }
        sendPADI(conn);
        conn->discoveryState = STATE_SENT_PADI;
        waitForPADO(conn, timeout);
    } while (conn->discoveryState == STATE_SENT_PADI);
```

## 1.4 PADS, active discovery session-confirmation
1.产生了唯一的sessionid

2.接入集中器收到PADR分组后准备开始PPP会话，它发送一个PPPoE有效发现会话确认PADS分组。

其中CODE字段值为0×65（PADS Code），SESSION-ID字段值为接入集中器所产生的一个惟一的PPPoE会话标识号码。

```c
    /* Send PADS and Start pppd */
    memcpy(pads.ethHdr.h_dest, packet->ethHdr.h_source, ETH_ALEN);
    memcpy(pads.ethHdr.h_source, myAddr, ETH_ALEN);
    pads.ethHdr.h_proto = htons(Eth_PPPOE_Discovery);
    pads.ver = 1;
    pads.type = 1;
    pads.code = CODE_PADS;
    //#define CODE_PADS           0x65
    pads.session = cliSession->sess;
```

```
PPP-over-Ethernet Discovery
    0001 .... = Version: 1
    .... 0001 = Type: 1
    Code: Active Discovery Session-confirmation (PADS) (0x65)
    Session ID: 0x3f43
    Payload Length: 20
    PPPoE Tags
        Host-Uniq: 010000000000000002000000
```
4.PADS和PADR的Host-Uniq Tag值相同。

## 1.5 PADT
它可以在会话建立后的任何时候发送，来终止PPPoE会话，也就是会话释放。它可以由主机或者接入集中器发送。当对方接收到一个PADT分组，就不再允许使用这个会话来发送PPP业务。PADT分组不需要任何标签，其CODE字段值为0×a7，SESSION-ID字段值为需要终止的PPP会话的会话标识号码。在发送或接收PADT后，即使正常的PPP终止分组也不必发送。PPP对端应该使用PPP协议自身来终止PPPoE会话，但是当PPP不能使用时，可以使用PADT。

tags：

- Host-Uniq
为主机唯一标识，类似于PPP数据报文中的标识域，主要是用来匹配发送和接收端的。因为对于 广播式的网络中会同时存在很多个PPPoE的数据报文。
- AC-Cookie
Ac-Cookie是为了防止拒绝服务攻击（Denial of Service，简称DOS）。访问集中器（AC）能够根据PADR的源地址来重新产生唯一的Tag_Value。使用这种方法，AC可以确保PADI的 源地址是可达的，并对该地址的并行会话数进行限制。

# 2. 会话阶段
LCP协商 --》 认证 --》NCP协商

用户主机与接入集中器根据在发现阶段所协商的PPP会话连接参数进行PPP会话。一旦PPPoE会话开始，PPP数据就可以以任何其他的PPP封装形式发送。所有的以太网帧都是单播的。PPPoE会话的SESSION-ID一定不能改变，并且必须是发现阶段分配的值。

# 3. 关键数据结构
```c
pppoe包结构

/* A PPPoE Packet, including Ethernet headers */
typedef struct PPPoEPacketStruct {
    struct ethhdr ethHdr;   /* Ethernet header */
    unsigned int vertype:8; /* PPPoE Version and Type (must both be 1) */
    unsigned int code:8;    /* PPPoE code */
    unsigned int session:16;    /* PPPoE session */
    unsigned int length:16; /* Payload length */
    unsigned char payload[ETH_JUMBO_LEN]; /* A bit of room to spare */
} PPPoEPacket;
```

```c
/* Header size of a PPPoE packet */
#define PPPOE_OVERHEAD 6  /* type, code, session, length */
#define HDR_SIZE (sizeof(struct ethhdr) + PPPOE_OVERHEAD)
#define MAX_PPPOE_PAYLOAD (ETH_JUMBO_LEN - PPPOE_OVERHEAD)
/* There are other fixed-size buffers preventing
   this from being increased to 16110. The buffer
   sizes would need to be properly de-coupled from
   the default MRU. For now, getting up to 1500 is
   enough. */
//#define ETH_JUMBO_LEN 1508
#define PPP_OVERHEAD 2  /* protocol */
#define MAX_PPPOE_MTU (MAX_PPPOE_PAYLOAD - PPP_OVERHEAD)
#define TOTAL_OVERHEAD (PPPOE_OVERHEAD + PPP_OVERHEAD)
#define ETH_PPPOE_MTU (ETH_DATA_LEN - TOTAL_OVERHEAD)
//#define ETH_DATA_LEN ETHERMTU
```

```c
/* PPPoE Tag */

typedef struct PPPoETagStruct {
    unsigned int type:16;   /* tag type */
    unsigned int length:16; /* Length of payload */
    unsigned char payload[ETH_JUMBO_LEN]; /* A LOT of room to spare */
} PPPoETag;

/* PPPoE Tags */
#define TAG_END_OF_LIST        0x0000
#define TAG_SERVICE_NAME       0x0101
#define TAG_AC_NAME            0x0102
#define TAG_HOST_UNIQ          0x0103
#define TAG_AC_COOKIE          0x0104
#define TAG_VENDOR_SPECIFIC    0x0105
#define TAG_RELAY_SESSION_ID   0x0110
#define TAG_PPP_MAX_PAYLOAD    0x0120
#define TAG_SERVICE_NAME_ERROR 0x0201
#define TAG_AC_SYSTEM_ERROR    0x0202
#define TAG_GENERIC_ERROR      0x0203
```

```c
/* A client session */
typedef struct ClientSessionStruct {
    struct ClientSessionStruct *next; /* In list of free or active sessions */
    PppoeSessionFunctionTable *funcs; /* Function table */
    pid_t pid;          /* PID of child handling session */
    Interface *ethif;       /* Ethernet interface */
    unsigned char myip[IPV4ALEN]; /* Local IP address */
    unsigned char peerip[IPV4ALEN]; /* Desired IP address of peer */
    UINT16_t sess;      /* Session number */
    unsigned char eth[ETH_ALEN]; /* Peer's Ethernet address */
    unsigned int flags;     /* Various flags */
    time_t startTime;       /* When session started */
    char const *serviceName;    /* Service name */
    UINT16_t requested_mtu;     /* Requested PPP_MAX_PAYLOAD  per RFC 4638 */
#ifdef HAVE_LICENSE
    char user[MAX_USERNAME_LEN+1]; /* Authenticated user-name */
    char realm[MAX_USERNAME_LEN+1]; /* Realm */
    unsigned char realpeerip[IPV4ALEN]; /* Actual IP address -- may be assigned
                       by RADIUS server */
    int maxSessionsPerUser; /* Max sessions for this user */
#endif
#ifdef HAVE_L2TP
    l2tp_session *l2tp_ses; /* L2TP session */
    struct sockaddr_in tunnel_endpoint; /* L2TP endpoint */
#endif
} ClientSession;
```

```c
typedef struct PPPoEConnectionStruct {
    int discoveryState;     /* Where we are in discovery */
    int discoverySocket;    /* Raw socket for discovery frames */
    int sessionSocket;      /* Raw socket for session frames */
    unsigned char myEth[ETH_ALEN]; /* My MAC address */
    unsigned char peerEth[ETH_ALEN]; /* Peer's MAC address */
    unsigned char req_peer_mac[ETH_ALEN]; /* required peer MAC address */
    unsigned char req_peer; /* require mac addr to match req_peer_mac */
    UINT16_t session;       /* Session ID */
    char *ifName;       /* Interface name */
    char *serviceName;      /* Desired service name, if any */
    char *acName;       /* Desired AC name, if any */
    int synchronous;        /* Use synchronous PPP */
    int useHostUniq;        /* Use Host-Uniq tag */
    int printACNames;       /* Just print AC names */
    FILE *debugFile;        /* Debug file for dumping packets */
    int numPADOs;       /* Number of PADO packets received */
    PPPoETag cookie;        /* We have to send this if we get it */
    PPPoETag relayId;       /* Ditto */
    int error;          /* Error packet received */
    int debug;          /* Set to log packets sent and received */
    int discoveryTimeout;       /* Timeout for discovery packets */
    int seenMaxPayload;
    int mtu;            /* Stored MTU */
    int mru;            /* Stored MRU */
} PPPoEConnection;
```
