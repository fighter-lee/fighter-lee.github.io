---
layout:     post
title:      "mqtt深入浅出系列"
subtitle:   "二、mqtt源码分析"
date:       2017-10-1
author:     "李洋彪"
header-img: "img/post-bg-js-module.jpg"
tags:
    - Android
---

>嗯，没抢到十一的票，明天才能回家，来，写篇博客吧。

### 连接

逐层跟代码，到ClientComms类，该类用于与server通讯，发送和接收mqtt协议消息。

先看connect代码：

	public void connect(MqttConnectOptions options, MqttToken token) throws MqttException {
		final String methodName = "connect";
		synchronized (conLock) {
			if (isDisconnected() && !closePending) {
				//@TRACE 214=state=CONNECTING

				conState = CONNECTING;

				conOptions = options;

                MqttConnect connect = new MqttConnect(client.getClientId(),
                        conOptions.getMqttVersion(),
                        conOptions.isCleanSession(),
                        conOptions.getKeepAliveInterval(),
                        conOptions.getUserName(),
                        conOptions.getPassword(),
                        conOptions.getWillMessage(),
                        conOptions.getWillDestination());

                this.clientState.setKeepAliveSecs(conOptions.getKeepAliveInterval());
                this.clientState.setCleanSession(conOptions.isCleanSession());
                this.clientState.setMaxInflight(conOptions.getMaxInflight());

				tokenStore.open();
				ConnectBG conbg = new ConnectBG(this, token, connect, executorService);
				conbg.start();
			}
			else {
				// @TRACE 207=connect failed: not disconnected {0}
				if (isClosed() || closePending) {
					throw new MqttException(MqttException.REASON_CODE_CLIENT_CLOSED);
				} else if (isConnecting()) {
					throw new MqttException(MqttException.REASON_CODE_CONNECT_IN_PROGRESS);
				} else if (isDisconnecting()) {
					throw new MqttException(MqttException.REASON_CODE_CLIENT_DISCONNECTING);
				} else {
					throw ExceptionHelper.createMqttException(MqttException.REASON_CODE_CLIENT_CONNECTED);
				}
			}
		}
	}

可以看到，无非是：获取之前我们设置的一些参数 --> 然后开启**ConnectBG**连接线程

看ConnectBG的run方法：

	NetworkModule networkModule = networkModules[networkModuleIndex];
	networkModule.start();
	receiver = new CommsReceiver(clientComms, clientState, tokenStore, networkModule.getInputStream());
	receiver.start("MQTT Rec: "+getClient().getClientId(), executorService);
	sender = new CommsSender(clientComms, clientState, tokenStore, networkModule.getOutputStream());
	sender.start("MQTT Snd: "+getClient().getClientId(), executorService);
	callback.start("MQTT Call: "+getClient().getClientId(), executorService);
	internalSend(conPacket, conToken);

这段代码的逻辑是：建立（tcp或者ssl或者WebSocket或者WebSocketSecure）连接 --> 开启接收线程 --> 开启发送线程 --> 发送消息

建立上文中的何种连接方式，是根据serverURI来判断，参考MqttAsyncClient的createNetworkModules方法。

我们分析最常见的tcp连接方式（类TCPNetworkModule的start方法）：

	SocketAddress sockaddr = new InetSocketAddress(host, port);
	if (factory instanceof SSLSocketFactory) {
		// SNI support
		Socket tempsocket = new Socket();
		tempsocket.connect(sockaddr, conTimeout*1000);
		socket = ((SSLSocketFactory)factory).createSocket(tempsocket, host, port, true);
	} else {
		socket = factory.createSocket();
		socket.connect(sockaddr, conTimeout*1000);
	}

嗯，最终在这建立的socket连接。

### 如何发送消息

先简单描述：

将message、topic等字段实例化IMqttToken --> 接收线程循环获取msg --> 写流传输

看看具体的代码：

获取msg:

	message = clientState.get();
	if (message != null) {
		//@TRACE 802=network send key={0} msg={1}
	
		if (message instanceof MqttAck) {
			out.write(message);
			out.flush();
		} else {
			MqttToken token = tokenStore.getToken(message);
			// While quiescing the tokenstore can be cleared so need
			// to check for null for the case where clear occurs
			// while trying to send a message.
			if (token != null) {
				synchronized (token) {
					out.write(message);
					try {
						out.flush();
					} catch (IOException ex) {
			
					....

### 如何接收消息

读取数据流，再分发通知。

看接收线程的代码：

	while (running && (in != null)) {
			try {
				//@TRACE 852=network read message
				receiving = in.available() > 0;
				MqttWireMessage message = in.readMqttWireMessage();
				receiving = false;

				// instanceof checks if message is null
				if (message instanceof MqttAck) {
					token = tokenStore.getToken(message);
					if (token!=null) {
						synchronized (token) {
							// Ensure the notify processing is done under a lock on the token
							// This ensures that the send processing can complete  before the
							// receive processing starts! ( request and ack and ack processing
							// can occur before request processing is complete if not!
							clientState.notifyReceivedAck((MqttAck)message);
						}
					} else if

					...

### 如何保持长连接的呢

通过闹钟定时任务（AlarmPingSender）：

	pendingIntent = PendingIntent.getBroadcast(service, 0, new Intent(action), PendingIntent.FLAG_UPDATE_CURRENT);

    schedule(comms.getKeepAlive());

从AlarmReceiver类onReceive方法入手切入，ClientState的checkForActivity方法有这么一段代码：

    token = new MqttToken(clientComms.getClient().getClientId());
    if(pingCallback != null){
    	token.setActionCallback(pingCallback);
    }
    tokenStore.saveToken(token, pingCommand);
    pendingFlows.insertElementAt(pingCommand, 0);

    nextPingTime = getKeepAlive();

    //Wake sender thread since it may be in wait state (in ClientState.get())                                                                                                                             
    notifyQueueLock();

原来是pendingFlows队列中插入消息，接收线程收到消息将数据写入流。

### 总结

源码结构清晰，功能很完整，写的还不错的，还是值得一读。

当然也便于自己扩展。