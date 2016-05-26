# q315099997.github.com
github blog page

|标记       |说明                    |
| --------- |:----------------------:| 
|"$url"     |传入的播放地址,字符串   |
|"$session" |播放器当前 Connection ID|
|"$pid"     |线程号                  |
|"$err"     |错误号                  |

##HLS播放：

|重点环节                    |标准Log                                     |Player             |
|----------------------------|:------------------------------------------:|:-----------------:|
|播放器获取播放地址          |setDataSource("$url")                       |[$session] setDataSource(url)|
|播放器开始请求播放列表(M3U8)|fetchPlaylist starting("$url")              |fetchPlaylist starting(url)|
|播放器开始准备              |["$session"] prepareAsync                   |[$session] prepareAsync|
|播放器获取(M3U8)完成        |fetchPlaylist done, 409:409 bytes cost 0.07s|fetchPlaylist done: cost 567MS|
|M3U8文件                    |```\#EXTM3U\#EXT-X-VERSION:3\#EXT-X-ALLOW-CACHE:YES\#EXT-X-TARGETDURATION:20\#EXT-X-MEDIA-SEQUENCE:0/play/slices/0.ts?id=7a8d57eb8cf6552fc584f251d032fa91&segment=0, D: 1.08. /play/slices/1.ts?id=7a8d57eb8cf6552fc584f251d032fa91&segment=1, D: 2.92..... /play/slices/152.ts?id=7a8d57eb8cf6552fc584f251d032fa91&segment=152, D: 4.65\#EXT-X-ENDLIST```|  |
|解析URL(M3U8)完成           |success to parse .m3u8 playlist             |success to parse .m3u8 playlist|
|请求M3U8超时                |fetchPlaylist timeout, cost 5.12s           |no, fail原因在“请求M3U8失败”的error msg中|
|请求M3U8失败                |failed to load playlist at url ("$url") (vod请求或者live刷新)||
|播放器请求ts分片            |```fetching segment 1442897414 from (1442897414 .. 1442897416)(duration 9.28) fetching("$url")```|```fetching sequence 0,duration=2000ms,uri=http://10.154.250.32:8081/media/live/cloudvideo_robustness/protocol/audioformat/1080p/ver_00_16_0_0_1_1824352_0.ts```|
|播放器下载完ts分片          |```fetch seq 1442897414,percet(100,100),size(1064,1064 KB),duration  9.28,cost  0.89 secs
  说明：
  1,下载时间“cost  0.89 secs”，会打印出实际下载时间，如果超时，会打印出超时的时间
  2,超时时间，vod是30s, live是2*durationUs; 如果超时，之后的逻辑都是mSeqNumber++
  3,下载错误在“播放器下载模块下载请求返回值”打印
  4,其他的如404错误等，之后的逻辑也是mSeqNumber++```|fetch sequence 0, cost 679ms, size=1824352 Bytes, duration=2000ms|

|播放器请求的ts分片SeqNum小于当前播放列表最小SeqNum|```We've missed the boat, restarting playback, was looking for 1442897410 in 1442897414-1442897416```|no，需要增加|

|播放器请求的ts分片SeqNum大于当前播放列表最大SeqNum|```sequence number high: 1442897417 from (1442897414..1442897416), refresh m3u8(retry=2)```|no，需要增加|

|播放器下载模块下载请求返回值|download ("$url") return ("$err") (strerror)|Unable to connect to:|
|播放器缓冲中插入Discontinuity|queueing discontinuity(DiscontinuityType)| |
|Video Decoder初始化         |initiating video decoder|initiating Decoder(OMX.MTK.VIDEO.DECODER.AVC)|
|Video Decoder初始化完成     |video decoder initialized|Decoder initialized(OMX.MTK.VIDEO.DECODER.AVC)|
|Audio Decoder初始化         |initiating audio decoder|initiating Decoder(OMX.google.aac.decoder)|
|Audio Decoder初始化完成     |audio decoder initialized|Decoder initialized(OMX.google.aac.decoder)|
|播放器Prepared              |```buffered time = %.2f, prep cost %.2f ["$session"] notify (0x4d3f73b0, 1, 0, 0) 或者 ["$session"] prepared```|  |
|播放器start                 |["$session"] start|  |
|播放器start done            |["$session"] start done|  |
|第一帧显示时间              |["$session"] notify (0x4d3f73b0, 200, 3, 0) 或者 ["$session"] First Frame Renderred|[$session] render start|
|播放器开始loading           |["$session"] notify (0x4d3f73b0, 200, 701, 0) 或者 ["$session"] 701, buffering start| |
|播放器loading进度           |["$session"] notify (0x4d3f73b0, 200, 704, 10) 或者 ["$session"] 704, in buffering, percent(10%)| |
|播放器loading结束           |["$session"] notify (0x4d3f73b0, 200, 702, 0) 或者 ["$session"] 702, buffering END, buf cost %.2f| |
|播放器预下载进度            |["$session"] notify (0x4d3f73b0, 3, 85, 0) 或者 ["$session"] buffering update,  percent(85/100)=85%| |
|Seek 开始                   |["$session"] seekTo(608144)| |
|first-seek                  |first-seek at time %lld us %.2f s| |
|Seek 完成                   |["$session"] notify (0x40450700, 4, 0, 0) 或者 ["$session"] Seek Complete| |
|停止播放器                  |["$session"] stop| |
|停止播放器完成              |["$session"] stop done| |
|释放播放器                  |disconnect("$session") from pid ("$pid“)| |
|释放播放器完成              |["$session"] disconnect done| |

##HTTP播放：

|重点环节         |标准Log          |Player           |
|-----------------|-----------------|-----------------|
|播放器获取播放地址|["$session"] setDataSource("$url")| |
|播放器请求下载URL|Connecting to ： $url| |
|Video Decoder初始化|initiating video decoder||
|Video Decoder初始化完成|video decoder initialized| |
|Audio Decoder初始化|initiating audio decoder| |
|Audio Decoder初始化完成|audio decoder initialized| |
|播放器Prepared|["$session"] notify (0x4d3f73b0, 1, 0, 0) 或者 ["$session"] prepared| |
|播放器start|["$session"] start| |
|播放器start done|["$session"] start done| |
|第一帧显示时间|`["$session"] notify (0x4d3f73b0, 200, 3, 0) 或者 ["$session"] First Frame Renderred`| |
|播放器开始loading|`["$session"] notify (0x4d3f73b0, 200, 701, 0) 或者 ["$session"] 701, Buffering Start`| |
|播放器loading结束|`["$session"] notify (0x4d3f73b0, 200, 702, 0) 或者 ["$session"] 702, Buffering End`| |
|Seek 开始|["$session"] seekTo(608144)| |
|Seek 完成|`["$session"] notify (0x40450700, 4, 0, 0) 或者 ["$session"] Seek Complete`| |
|停止播放器|["$session"] stop| |
|停止播放器完成|["$session"] stop done| |
|释放播放器|disconnect("$session") from pid ("$pid“)| |
|释放播放器完成|["$session"] disconnect done| |

##DASH播放：

|重点环节         |标准Log          |Player           |
|-----------------|-----------------|-----------------|
|播放器获取播放地址|["$session"] setDataSource("$url")||
|播放器请求下载URL|Connecting to ： $url||
|Video Decoder初始化|initiating video decoder||
|Video Decoder初始化完成|video decoder initialized||
|Audio Decoder初始化|initiating audio decoder||
|Audio Decoder初始化完成|audio decoder initialized||
|播放器Prepared| ["$session"] prepared||
|播放器start|["$session"] start||
|播放器start done|["$session"] start done||
|第一帧显示时间| ["$session"] First Frame Renderred||
|播放器开始loading| ["$session"] 701, Buffering Start||
|播放器loading结束| ["$session"] 702, Buffering End||
|Seek 开始|["$session"] seekTo(608144)||
|Seek 完成|["$session"] Seek Complete||
|停止播放器|["$session"] stop||
|停止播放器完成|["$session"] stop done||
|释放播放器|disconnect("$session") from pid ("$pid“)||
|释放播放器完成|["$session"] disconnect done||


##RTMP播放：

|重点环节         |标准Log          |Player           |
|-----------------|-----------------|-----------------|
|播放器获取播放地址|["$session"] setDataSource("$url")||
|播放器请求下载URL|Connecting to ： $url||
|Video Decoder初始化|initiating video decoder||
|Video Decoder初始化完成|video decoder initialized||
|Audio Decoder初始化|initiating audio decoder||
|Audio Decoder初始化完成|audio decoder initialized||
|播放器Prepared| ["$session"] prepared||
|播放器start|["$session"] start||
|播放器start done|["$session"] start done||
|第一帧显示时间| ["$session"] First Frame Renderred||
|播放器开始loading| ["$session"] 701, Buffering Start||
|播放器loading结束| ["$session"] 702, Buffering End||
|Seek 开始|["$session"] seekTo(608144)||
|Seek 完成|["$session"] Seek Complete||
|停止播放器|["$session"] stop||
|停止播放器完成|["$session"] stop done||
|释放播放器|disconnect("$session") from pid ("$pid“)||
|释放播放器完成|["$session"] disconnect done||