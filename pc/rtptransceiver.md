# rtp transceiver

## 1. RtpTransceiverInterface

transceiver = RtpSender + RtpReceiver，并共享一个mid

* media_type()：返回媒体类型，如audio/video/data
* mid()：返回mid字符串
* sender()：返回RtpSenderInterface对象
* receiver()：返回RtpReceiverInterface对象
* stopped()：是否已停止
* direction()：方向，收发/只发/只收
* SetCodecPreferences()：设置内置期望的codecs

## 2. RtpTransceiver

RtpTransceiver管理着BaseChannel/RtpSenderInternal/RtpReceiverInternal

* 能直接访问类似：channel()/senders()/receivers()/sender_internal()/receiver_internal()，可以返回channel, sender, receiver等

## 3. 