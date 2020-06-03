# call

类图关系：  
![call_class](./links/call/call.png)

## 1. call

核心流程：主要是用于创建  

1. 在PeerConnectionFactory::CreatePeerConnection中调用CreateCall_w创建call对象，随后会传入WebRtcVoiceMediaChannel和WebRtcVideoChannel
   - 调用call_factory_->CreateCall创建call对象（调用Call::Create）
2. 在WebRtcVoiceMediaChannel::WebRtcAudioSendStream构造函数中调用call_->CreateAudioSendStream(config_)创建audio发送stream_
3. 在WebRtcVoiceMediaChannel::WebRtcAudioReceiveStream构造函数中调用call_->CreateAudioReceiveStream(config_)创建audio接收stream_
4. 在WebRtcVoiceMediaChannel::OnPacketReceived中调用call_->Receiver()->DeliverPacket接收audio的rtp包
5. 在WebRtcVoiceMediaChannel::OnRtcpReceived中调用call_->Receiver()->DeliverPacket接收audio的rtcp包
6. 在WebRtcVoiceMediaChannel::OnNetworkRouteChanged中调用call_->GetTransportControllerSend()->OnNetworkRouteChanged和call_->OnTransportOverheadChanged通知网络路由发生改变
7. 在WebRtcVoiceMediaChannel::OnReadyToSend中调用call_->SignalChannelNetworkState设置网络连接或者断开
8. 在WebRtcVideoChannel::SetSendParameters中调用call_->GetTransportControllerSend()->SetSdpBitrateParameters将sdp协商好的码率设置到transport controller send中
9. 在WebRtcVideoChannel::GetStats中调用call_->GetStats()获取call层次的统计信息
10. 在WebRtcVideoChannel::OnPacketReceived中调用call_->Receiver()->DeliverPacket接收video的rtp包
11. 在WebRtcVideoChannel::OnRtcpReceived中调用call_->Receiver()->DeliverPacket接收video的rtcp包
12. 在WebRtcVideoChannel::OnReadyToSend中调用call_->SignalChannelNetworkState设置网络连接或者断开
13. 在WebRtcVideoChannel::OnNetworkRouteChanged中调用call_->GetTransportControllerSend()->OnNetworkRouteChanged和call_->OnTransportOverheadChanged通知网络路由发生改变
14. 在WebRtcVideoChannel::WebRtcVideoSendStream::RecreateWebRtcStream()中调用call_->CreateVideoSendStream创建video发送stream_
15. 在WebRtcVideoChannel::WebRtcVideoReceiveStream::RecreateWebRtcVideoStream()中调用call_->CreateVideoReceiveStream创建video接收stream_
16. 在WebRtcVideoChannel::WebRtcVideoReceiveStream::MaybeRecreateWebRtcFlexfecStream()中调用call_->CreateFlexfecReceiveStream创建flexfec stream_

## 2. internal::call

* call构造函数
  * 创建处理线程module_process_thread_（挂载几个主要的模块remote_estimator，call_stats_，receive_side_cc_），这个线程对象还会传递到其他stream对象中
  * 创建BitrateAllocator对象bitrate_allocator_，其中内部的limit_observer_对象是call本身
  * 创建ReceiveSideCongestionController对象receive_side_cc_
  * 由参数创建RtpTransportControllerSendInterface对象transport_send_，并调用transport_send->RegisterTargetTransferRateObserver(this)注册相关observer
* Receiver()返回this指针，很多地方用这个函数来接收rtp数据
* CreateAudioSendStream()创建AudioSendStream对象，保存在audio_send_ssrcs_中，需要注意的是这一路ssrc对应的流的rtcp是在另外一个AudioReceiveStream中管理的
* CreateAudioReceiveStream()创建AudioReceiveStream对象，保存在audio_receive_streams_中，需要注意的是其中会负责某一个send stream的rtcp的发送
* CreateVideoSendStream()创建VideoSendStream对象，保存在video_send_streams_中，另外也保存在video_send_ssrcs_中
* CreateVideoReceiveStream()创建VideoReceiveStream对象，保存在video_receive_streams_中
* CreateFlexfecReceiveStream()创建FlexfecReceiveStreamImpl对象，其中RecoveredPacketReceiver是call对象本身，恢复后的包会通过RecoveredPacketReceiver传递回来
* GetTransportControllerSend()返回transport_send_ptr_，即transport_send_的指针对象
* GetStats()返回call层次的统计信息
* OnSentPacket()会更新video_send_delay_stats_统计信息，也会更新底层send_side_cc_的信息。这个函数会在PeerConnection::OnSentPacket_w中调用
* OnTargetTransferRate()更新目标码率，这个函数会在RtpTransportControllerSend::OnNetworkChanged调用
* OnAllocationLimitsChanged()更新码率上下限，这个函数会在BitrateAllocator::UpdateAllocationLimits()中调用

