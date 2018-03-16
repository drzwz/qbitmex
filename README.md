# qbitmex
kdb+/q interface for BitMex API (REST and WebSocket)

# examples

## settings: apiHost,apiKey,apiSecret

settings:\`apiHost\`apiKey\`apiSecret!("testnet.bitmex.com";"";"")   //testnet

## BitMex REST API (<https://www.bitmex.com/app/restAPI>)
restapi[host;verb;path;data;apiKey;apiSecret]  // Not authenticating when apiKey or apiSecret = ""

r:restapi[settings\`apiHost;"GET";"/api/v1/position";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/apiKey?reverse=false";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/chat?count=100&reverse=true";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/announcement";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/announcement/urgent";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/execution";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/execution/tradeHistory";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/funding";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/instrument";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/order";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/orderBook/L2?symbol=XBTUSD";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/quote?symbol=XBTUSD&reverse=true";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/schema";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/schema/websocketHelp";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/stats";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/trade?symbol=XBTUSD&count=5&start=0&startTime=2018-03-01 00:20:00";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/user";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/user/wallet";"";settings\`apiKey;settings\`apiSecret];r\`body
r:restapi[settings\`apiHost;"GET";"/api/v1/user/walletSummary";"";settings\`apiKey;settings\`apiSecret];r\`body


## WebSocket API (<https://www.bitmex.com/app/wsAPI>)

### /wsapi: connect to websocket: Not authenticating when apiKey or apiSecret = ""
wsapi[host;apiKey;apiSecret]

### /wsapi_cmd:   wshandle: the first element returned from wsapi[...], command: a dict for command args, ex: \`op\`args!(`subscribe;enlist \`$"trade:XBTUSD")
wsapi_cmd[wshandle;command]

### /subscribe:  r:wsapi[settings\`apiHost; settings\`apiKey; settings\`apiSecret];  wsapi_sub[first[r];"trade:XBTUSD"]
wsapi_sub[wshandle;sub_args]

### /unsubscribe:   wsapi_unsub[first[r];"trade:XBTUSD"] 
wsapi_unsub[wshandle;unsub_args]

### /wsapi_ping: send ping
wsapi_ping[wshandle]

### WebSocket API examples:
####  /subscribe trade
trade:([]timestamp:\`timestamp$();price:\`float$();size:\`float$());

.z.ws:{xx::.j.k[x];if[key[xx]~\`table\`action\`data;
	if[xx[`action]~"insert"; \`trade insert select ltime\`timestamp$"Z"$timestamp,\`float$price,\`float$size from xx[\`data] ]];
	};

wsh:wsapi[settings\`apiHost; settings\`apiKey; settings\`apiSecret];  

wsapi_sub[first[wsh];"trade:XBTUSD"]

wsapi_unsub[first[wsh];"trade:XBTUSD"]

#### /Heartbeats: https://www.bitmex.com/app/wsAPI#Heartbeats

wsapi_dt:.z.P;

WS:([]time:\`time$();data:());

.z.ws:{wsapi_dt::.z.P; \`WS insert (.z.T;x);0N!(.z.T;.z.w)};

wsh:wsapi[settings\`apiHost; settings\`apiKey; settings\`apiSecret];  

wsapi_sub[first[wsh];"trade:XBTUSD"];

.z.ts:{if[00:00:05<.z.P-wsapi_dt; @[wsapi_ping;first wsh;\`]];

    if[00:00:05<.z.P-wsapi_dt;
	
		@[hclose;first[wsh];\`];
		
        wsh::.[wsapi;(settings\`apiHost;settings\`apiKey;settings\`apiSecret);\`];
		
        if[0<first wsh;.[wsapi_sub;(first[wsh];"trade:XBTUSD");\`];wsapi_dt::.z.P];
		
        0N!(.z.Z;first[wsh];\`reconnected);
		
	  ];};

system"t 1000";

