title: jave_ffmpeg的java语言封装 

#  JAVE(Java Audio Video Encoder) ffmpeg项目的Java语言封装 
JAVE (Java Audio Video Encoder) 类库是一个 ffmpeg 项目的 Java 语言封装。开发人员可以使用JAVE 在不同的格式间转换视频和音频。例如将 AVI 转成 MPEG 动画，等等 ffmpeg 中可以完成的在 JAVE 都有对应的方法。
官网：http://www.sauronsoftware.it/projects/jave/
The JAVE distribution **includes two pre-compiled executables of ffmpeg**: a Windows one and a Linux one, both compiled for i386/32 bit hardware achitectures. This should be enough in most cases. 

获取视频时长：
关键代码
```

MultimediaInfo info=encoder.getInfo(file);
long millis=info.getDuration();

```
```

/**
	 * 获取媒体文件的播放时长
	 * @param src
	 * @return
	 */
	public String getMediaDuration(File src){
		Encoder encoder=new Encoder();
		String duration="";
		try {
			MultimediaInfo info=encoder.getInfo(src);
			long millis=info.getDuration();
			duration=DateUtil.formatMediaDuration(millis);
		} catch (EncoderException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			log.warn("视频解析错误{}",e);
		}
		return duration;
	}
	/**
	 * 处理媒体文件播放间隔
	 * @param millis 时间时间毫秒
	 * @return 如03:01:00 时长3小时1分0秒
	 */
	public  static String formatMediaDuration(long millis){
			long seconds=millis/1000;
			int second=(int) (seconds%60);
			int minute=(int)((seconds/60)%60);
			int hour=(int)((seconds/(60*60))%24);
			String str=formatTimeUnit(hour)+":"+formatTimeUnit(minute)+":"+formatTimeUnit(second);
			
			return str;
	}
	/**
	 * 格式化时间单元，支持秒，时，分。如3格式化为03
	 * @param unit
	 * @return
	 */
	private static String formatTimeUnit(int unit){
		String str="";
		if(unit<10){
			if(unit==0){
				str=unit+"0";
			}else{
				str="0"+unit;
			}
		}else{
			str=""+unit;
		}
		return str;
	}

```
下面例子将 AVI 动画转成 FLV 格式：
```

File source = new File("source.avi");
File target = new File("target.flv");
AudioAttributes audio = new AudioAttributes();
audio.setCodec("libmp3lame");
audio.setBitRate(new Integer(64000));
audio.setChannels(new Integer(1));
audio.setSamplingRate(new Integer(22050));
VideoAttributes video = new VideoAttributes();
video.setCodec("flv");
video.setBitRate(new Integer(160000));
video.setFrameRate(new Integer(15));
video.setSize(new VideoSize(400, 300));
EncodingAttributes attrs = new EncodingAttributes();
attrs.setFormat("flv");
attrs.setAudioAttributes(audio);
attrs.setVideoAttributes(video);
Encoder encoder = new Encoder();
encoder.encode(source, target, attrs);

```