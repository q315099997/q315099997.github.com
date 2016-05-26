# q315099997.github.com
github blog page

|标记       |说明                    |
| --------- |:----------------------:| 
|"$url"     |传入的播放地址,字符串   |
|"$session" |播放器当前 Connection ID|
|"$pid"     |线程号                  |
|"$err"     |错误号                  |

HLS播放：
|重点环节         |标准Log          |Player           |
|-----------------|-----------------|-----------------|
|播放器获取播放地址|setDataSource("$url")|[$session] setDataSource(url)|
|播放器开始请求播放列表(M3U8)|fetchPlaylist starting("$url")|fetchPlaylist starting(url)|
|播放器开始准备|["$session"] prepareAsync|[$session] prepareAsync|
|播放器获取(M3U8)完成|fetchPlaylist done, 409:409 bytes cost 0.07s|fetchPlaylist done: cost 567MS|
|M3U8文件|#EXTM3U
#EXT-X-VERSION:3
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:20
#EXT-X-MEDIA-SEQUENCE:0
/play/slices/0.ts?id=7a8d57eb8cf6552fc584f251d032fa91&segment=0, D: 1.08.
/play/slices/1.ts?id=7a8d57eb8cf6552fc584f251d032fa91&segment=1, D: 2.92.
....
/play/slices/152.ts?id=7a8d57eb8cf6552fc584f251d032fa91&segment=152, D: 4.65
#EXT-X-ENDLIST||
|解析URL(M3U8)完成|success to parse .m3u8 playlist|success to parse .m3u8 playlist|
|请求M3U8超时|fetchPlaylist timeout, cost 5.12s|no, fail原因在“请求M3U8失败”的error msg中|
|请求M3U8失败|failed to load playlist at url ("$url") (vod请求或者live刷新)||
|播放器请求ts分片|fetching segment 1442897414 from (1442897414 .. 1442897416)(duration 9.28)
fetching ("$url")|fetching sequence 0, duration=2000ms, uri=http://10.154.250.32:8081/media/live/cloudvideo_robustness/protocol/audioformat/1080p/ver_00_16_0_0_1_1824352_0.ts|
|播放器下载完ts分片|fetch seq 1442897414,percet(100,100),size(1064,1064 KB),duration  9.28,cost  0.89 secs
说明：
1,下载时间“cost  0.89 secs”，会打印出实际下载时间，如果超时，会打印出超时的时间
2,超时时间，vod是30s, live是2*durationUs; 如果超时，之后的逻辑都是mSeqNumber++
3,下载错误在“播放器下载模块下载请求返回值”打印
4,其他的如404错误等，之后的逻辑也是mSeqNumber++|fetch sequence 0, cost 679ms, size=1824352 Bytes, duration=2000ms|
|播放器请求的ts分片SeqNum小于当前播放列表最小SeqNum|We've missed the boat, restarting playback, was looking for 1442897410 in 1442897414-1442897416|no，需要增加|
|播放器请求的ts分片SeqNum大于当前播放列表最大SeqNum|sequence number high: 1442897417 from (1442897414..1442897416), refresh m3u8(retry=2)|no，需要增加|
||||
||||
||||
||||||||
