# qbitmex 

kdb+/q interface for BitMex API (REST and WebSocket)

# examples

## 0.1 implementation of HMACSHA256  

For Windows, use qx.dll or build DLLs. For Linux, build shared objects.

<https://github.com/ogay/hmac>  

<https://github.com/bitmx/btceQ/blob/master/c/hmac512.cpp>

## 0.2 load the functions
```q
\l qbitmex.q
```

## 0.3 settings: apiHost,apiKey,apiSecret
```q
settings:`apiHost`apiKey`apiSecret!("testnet.bitmex.com";"";"")   //testnet
```

## 1. BitMex REST API (<https://www.bitmex.com/app/restAPI>)

```q

restapi[host;verb;path;data;apiKey;apiSecret]  // Not authenticating when apiKey or apiSecret = ""

r:restapi[settings`apiHost;"GET";"/api/v1/position";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/apiKey?reverse=false";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/chat?count=100&reverse=true";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/announcement";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/announcement/urgent";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/execution";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/execution/tradeHistory";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/funding";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/instrument";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/order";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/orderBook/L2?symbol=XBTUSD";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/quote?symbol=XBTUSD&reverse=true";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/schema";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/schema/websocketHelp";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/stats";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/trade?symbol=XBTUSD&count=5&start=0&startTime=2018-03-01 00:20:00";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/user";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/user/wallet";"";settings`apiKey;settings`apiSecret];r`body
r:restapi[settings`apiHost;"GET";"/api/v1/user/walletSummary";"";settings`apiKey;settings`apiSecret];r`body

```

## 2. WebSocket API (<https://www.bitmex.com/app/wsAPI>)

### wsapi[host;apiKey;apiSecret]: connect to websocket: Not authenticating when apiKey or apiSecret = ""

### wsapi_cmd[wshandle;command]: send command
```q
wshandle: the first element returned from wsapi[...], command: a dict for command args, ex: `op`args!(`subscribe;enlist `$"trade:XBTUSD")
```

### wsapi_sub[wshandle;sub_args]: subscribe
```q
r:wsapi[settings`apiHost; settings`apiKey; settings`apiSecret];  wsapi_sub[first[r];"trade:XBTUSD"]
```


### wsapi_unsub[wshandle;unsub_args] :unsubscribe
```q
wsapi_unsub[first[r];"trade:XBTUSD"] 
```

### wsapi_ping[wshandle]: send ping


### WebSocket API examples:
####  /subscribe trade

```q

trade:([]timestamp:`timestamp$();price:`float$();size:`float$());

.z.ws:{xx::.j.k[x];if[key[xx]~`table`action`data;
	if[xx[`action]~"insert"; `trade insert select ltime`timestamp$"Z"$timestamp,`float$price,`float$size from xx[`data] ]];
	};

wsh:wsapi[settings`apiHost; settings`apiKey; settings`apiSecret];  

wsapi_sub[first[wsh];"trade:XBTUSD"]

wsapi_unsub[first[wsh];"trade:XBTUSD"]

```

#### /Heartbeats: https://www.bitmex.com/app/wsAPI#Heartbeats

```q

wsapi_dt:.z.P;
WS:([]time:`time$();data:());
.z.ws:{wsapi_dt::.z.P; `WS insert (.z.T;x);0N!(.z.T;.z.w)};
wsh:wsapi[settings`apiHost; settings`apiKey; settings`apiSecret];  
wsapi_sub[first[wsh];"trade:XBTUSD"];
.z.ts:{if[00:00:05<.z.P-wsapi_dt; @[wsapi_ping;first wsh;`]];
    if[00:00:05<.z.P-wsapi_dt;
		@[hclose;first[wsh];`];
        wsh::.[wsapi;(settings`apiHost;settings`apiKey;settings`apiSecret);`];
        if[0<first wsh;.[wsapi_sub;(first[wsh];"trade:XBTUSD");`];wsapi_dt::.z.P];
        0N!(.z.Z;first[wsh];`reconnected);
	  ];};
system"t 1000";

```
