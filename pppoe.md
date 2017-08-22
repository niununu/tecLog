<center>**pppoe**</center>
[TOC]

问题：
1. pppoe 和vpn有啥关系
2. pppoe的cookies是啥玩意
3. hostUniq是干啥的
4. lcp协议
5.

# 1. 报文
协商过程分为两个阶段：发现阶段和会话阶段

## 1.1 发现阶段 PPPoED：PPPoE Discovery
host -----------PADI-----------------> server
host <-----------PADO----------------- server
host -----------PADR-----------------> server
host <-----------PADS----------------- server

## 1.1.1 PADI
active discovery initiation
1. 主机广播发起分组，分组的目的地址为以太网的广播地址 0xffffffffffff
2. CODE（代码）字段值为0×09（PADI Code），
    SESSION-ID（会话ID）字段值为0x0000。
3. PADI分组必须至少包含一个服务名称类型的标签（Service Name Tag，字段值为0x0101），向接入集中器提出所要求提供的服务。

- 1. 客户端：discovery.c sendPADI
1. 主机广播发起分组，分组的目的地址为以太网的广播地址 0xffffffffffff
```c
    /* Set destination to Ethernet broadcast address */
    memset(packet.ethHdr.h_dest, 0xFF, ETH_ALEN);
```

2. CODE（代码）字段值为0×09（PADI Code），
    SESSION-ID（会话ID）字段值为0x0000。
```c
    packet.ethHdr.h_proto = htons(Eth_PPPOE_Discovery);
    packet.vertype = PPPOE_VER_TYPE(1, 1);
    //#define PPPOE_VER_TYPE(v, t)    (((v) << 4) | (t))
    packet.code = CODE_PADI;
    //#define CODE_PADI           0x09
    packet.session = 0;
```

3. PADI分组必须至少包含一个服务名称类型的标签（Service Name Tag，字段值为0x0101），向接入集中器提出所要求提供的服务。
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
    } else {
    plen = 0;
    }
```


## 1.1.2 PADO
active discovery offer
1. 接入集中器收到在服务范围内的PADI分组，发送PPPoE有效发现提供包分组，以响应请求。
2. 其中CODE字段值为0×07（PADO Code），
    SESSION-ID字段值仍为0x0000。
3. PADO分组必须包含一个接入集中器名称类型的标签（Access Concentrator Name Tag，字段值为0x0102），
以及一个或多个服务名称类型标签，表明可向主机提供的服务种类。
4. PADO和PADI的Host-Uniq Tag值相同。
Host-Uniq为主机唯一标识

- 2. 服务器：pppoe-server.c processPADI
可接受的tag：
```c
    PPPoETag acname;
    PPPoETag servname;
    PPPoETag cookie;
```

不处理收到的非广播padi包

```c
    /* Ignore PADI's which don't come from a unicast address */
    if (NOT_UNICAST(packet->ethHdr.h_source)) {
    syslog(LOG_ERR, "PADI packet from non-unicast source address");
    return;
    }
```

```c
/* If number of sessions per MAC is limited, check here and don't
   send PADO if already max number of sessions. */
if (MaxSessionsPerMac) {
    if (count_sessions_from_mac(packet->ethHdr.h_source) >= MaxSessionsPerMac){
        syslog(LOG_INFO, "PADI: Client %02x:%02x:%02x:%02x:%02x:%02x attempted to create more than %d session(s)",
           packet->ethHdr.h_source[0],
           packet->ethHdr.h_source[1],
           packet->ethHdr.h_source[2],
           packet->ethHdr.h_source[3],
           packet->ethHdr.h_source[4],
           packet->ethHdr.h_source[5],
           MaxSessionsPerMac);
        return;
    }
}
```

包验证函数
```c
/**********************************************************************
*%FUNCTION: parsePacket
*%ARGUMENTS:
* packet -- the PPPoE discovery packet to parse
* func -- function called for each tag in the packet
* extra -- an opaque data pointer supplied to parsing function
*%RETURNS:
* 0 if everything went well; -1 if there was an error
*%DESCRIPTION:
* Parses a PPPoE discovery packet, calling "func" for each tag in the packet.
* "func" is passed the additional argument "extra".
***********************************************************************/
int
parsePacket(PPPoEPacket *packet, ParseFunc *func, void *extra)
{
    UINT16_t len = ntohs(packet->length);
    unsigned char *curTag;
    UINT16_t tagType, tagLen;

    if (packet->ver != 1) {
    syslog(LOG_ERR, "Invalid PPPoE version (%d)", (int) packet->ver);
    return -1;
    }
    if (packet->type != 1) {
    syslog(LOG_ERR, "Invalid PPPoE type (%d)", (int) packet->type);
    return -1;
    }

    /* Do some sanity checks on packet */
    if (len > ETH_JUMBO_LEN - PPPOE_OVERHEAD) { /* 6-byte overhead for PPPoE header */
    syslog(LOG_ERR, "Invalid PPPoE packet length (%u)", len);
    return -1;
    }

    /* Step through the tags */
    curTag = packet->payload;
    while(curTag - packet->payload < len) {
    /* Alignment is not guaranteed, so do this by hand... */
    tagType = (((UINT16_t) curTag[0]) << 8) +
        (UINT16_t) curTag[1];
    tagLen = (((UINT16_t) curTag[2]) << 8) +
        (UINT16_t) curTag[3];
    if (tagType == TAG_END_OF_LIST) {
        return 0;
    }
    if ((curTag - packet->payload) + tagLen + TAG_HDR_SIZE > len) {
        syslog(LOG_ERR, "Invalid PPPoE tag length (%u)", tagLen);
        return -1;
    }
    func(tagType, tagLen, curTag+TAG_HDR_SIZE, extra);
    curTag = curTag + TAG_HDR_SIZE + tagLen;
    }
    return 0;
}
```

2. 其中CODE字段值为0×07（PADO Code），
    SESSION-ID字段值仍为0x0000。
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
    /*3. PADO分组必须包含一个接入集中器名称类型的标签（Access Concentrator Name Tag，字段值为0x0102），
以及一个或多个服务名称类型标签，表明可向主机提供的服务种类。*/
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



## 1.1.3 PADR
active discovery request
主机在可能收到的多个PADO分组中选择一个合适的PADO分组，然后向所选择的接入集中器发送PPPoE有效发现请求分组。
1. 其中CODE字段为0x19（PADR Code），SESSION_ID字段值仍为0x0000。
```c
    memcpy(packet.ethHdr.h_dest, conn->peerEth, ETH_ALEN);
    memcpy(packet.ethHdr.h_source, conn->myEth, ETH_ALEN);

    packet.ethHdr.h_proto = htons(Eth_PPPOE_Discovery);
    packet.vertype = PPPOE_VER_TYPE(1, 1);
    packet.code = CODE_PADR;
    packet.session = 0;
```

2. PADR分组必须包含一个服务名称类型标签，确定向接入集线器（或交换机）请求的服务种类。
```c
    svc->type = TAG_SERVICE_NAME;
    svc->length = htons(namelen);
    if (conn->serviceName) {
    memcpy(svc->payload, conn->serviceName, namelen);
    }
    cursor += namelen + TAG_HDR_SIZE;
```
3. 当主机在指定的时间内没有接收到PADO，它应该重新发送它的PADI分组，并且加倍等待时间，这个过程会被重复期望的次数。
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

## 1.1.4 PADS
active discovery session-confirmation

1.产生了唯一的sessionid
2.接入集中器收到PADR分组后准备开始PPP会话，它发送一个PPPoE有效发现会话确认PADS分组。
其中CODE字段值为0×65（PADS Code），SESSION-ID字段值为接入集中器所产生的一个惟一的PPPoE会话标识号码。
3. PADS分组也必须包含一个接入集中器名称类型的标签以确认向主机提供的服务。
当主机收到PADS 分组确认后，双方就进入PPP会话阶段。
4. PADS和PADR的Host-Uniq Tag值相同。

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

客户端：

**padt**

## 1.2 会话阶段
LCP协商 --》 认证 --》NCP协商

### 1.2.1 LCP协商
协商是双向的，双方都会发出configre-request请求，并且有可能多次协商才能成功
1、链路配置报文
用来创建和配置一条链路
Configure-Request、Configure-Ack、Configure-Nak、Configure-Reject
2、链路终止报文
用来终止一条链路
Terminate-Request、Terminate-Reply
3、链路维护报文
用来管理和调试链路
Code-Reject、Protocol-Reject、Echo-Request、Echo-Reply、Discard-Request

### 1.2.2 认证
认证方式：pap(两次握手) , chap（三次握手）
pc <--------challenge---------- server
pc --------response------------>server
pc <--------success---------- server

### 1.2.3 NCP协商
NCP的协商和LCP一样，也是双向协商的，即协商双方都需要发送Configure-Request报文，并接受到对端回应的Configure-Ack报文才行。

# 2.代码
```c
/*
     * Initialize magic number generator now so that protocols may
     * use magic numbers in initialization.
     */
    magic_init();

    /*
     * Initialize each protocol.
     */
    for (i = 0; (protp = protocols[i]) != NULL; ++i)
        (*protp->init)(0);

    /*
     * Initialize the default channel.
     */
    tty_init();
```
初始化magic Number -------》初始化协议 -------》初始化channel

pppoe包结构
```c
/* A PPPoE Packet, including Ethernet headers */
typedef struct PPPoEPacketStruct {
    struct ethhdr ethHdr;   /* Ethernet header */
    unsigned int vertype:8; /* PPPoE Version and Type (must both be 1) */
    unsigned int code:8;    /* PPPoE code */
    unsigned int session:16;    /* PPPoE session */
    unsigned int length:16; /* Payload length */
    unsigned char payload[ETH_JUMBO_LEN]; /* A bit of room to spare */
} PPPoEPacket;

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
