查看设备列表
> ffmpeg -f dshow -list_devices true -i dummy

录制音频和视频

> ffmpeg -framerate 30 -f gdigrab -i desktop -f dshow -i audio="音频设备" -c:v libx264rgb -preset:v ultrafast -tune:v zerolatency output.mp4

```
c:v 编码格式 h264/libx264rgb
b:v 比特率 2000k
framerate 帧率
-offset_x 0 -offset_y 0 -s 480*320 指定区域
```