# MediaExtractor、MediaCodec、MediaMuxer


## MediaExtractor

MediaExtractor是一个Android系统用于从多媒体文件中提取音频和视频数据的类。   

它可以从本地文件或网络流中读取音频和视频数据，并将其解码为原始的音频和视频帧。它的主要功能就是解封装，也就是从媒体文件中提取出原始的音频和视频数据流。这些数据流可以被送入解码器进行解码，然后进行播放或者其他处理。      

MediaExtractor可以用于开发音视频播放器、视频编辑器、音频处理器等应用程序。


## MediaMuxter

MediaMuxter是Android系统提供的一个用于混合音频和视频数据的API。    

它可以将音频和视频的原始数据流混合封装成媒体文件，例如MP4、WebM等。    

MediaMuxter通常与MediaExtractor一起使用，MediaExtractor用于从媒体文件中提取音频和视频数据，MediaMuxter用于将这些数据混合成新的媒体文件。   

简单说就是: MediaExtractor提供了解封装的能力，而MediaMuxer提供了视频封装的能力。   


## MediaCodec

MediaCodec是Android提供的用于对音视频进行编码的类，是Android Media基础框架的一部分，一般和MediaExtractor、MediaMuxer、Surface和AudioTrack一起使用。 




MediaExtractor仅仅是解封装数据，不会对数据进行解码。要对媒体数据进行解码，需要使用MediaCodec类。    

而且MediaExtractor只能解封装媒体文件中的音视频等媒体轨道，而不能解析整个媒体文件的结构。如果需要解析整个媒体文件的结构，需要使用其他库或框架。   

 
