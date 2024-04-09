# MediaMetadataRetriever

MediaMetadataRetriever是Android中用于从媒体文件中提取元数据的类，可以获取音频、视频和图片文件的各种信息，例如时长、标题、封面等。

```java
MediaMetadataRetriever mRetriever = new MediaMetadataRetriever();

mRetriever.setDataSource(mContext, mVideoUri);
```

## 获取元数据

// 获取歌曲标题
`mRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_TITLE);`

根据key的不同还可以是时长、帧率、分辨率等。

## 获取缩略图

`public Bitmap getFrameAtTime(long timeUs, int option)`
用于获取音视频文件中的一帧画面，可以用于实现缩略图、视频预览等功能。 

- timeUs:是获取画面对应的时间戳，单位为微妙
- option:可以设置是获取离指定时间戳最近的一帧画面或者事最近的关键帧画面等。

## 获取专辑封面图

```java
byte[] bytes = mRetriever.getEmbeddedPicture();
Bitmap bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.length); 
```
需要注意的是，MediaMetadataRetriever只能用于读取已经完成的音视频文件，无法用于实时处理音视频数据流。

