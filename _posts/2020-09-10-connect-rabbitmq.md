---
layout: post
title:  "Connection NodeJS With RabbitMQ"
date:   2020-09-10 11:30:00 +0700
categories: [java, rabbitmq, nodeJs]
---

### 자바 서버코드가 rabbitmq를 이용하여 nodejs서버로 정버 송신###

**Main**

```java
public class ConnectNodeJsQ {

	private final static String QUEUE_NAME = "hello";
	
	public void connectionQ(String id, String status, String check) throws IOException, TimeoutException {
		ConnectionFactory factory = new ConnectionFactory();
		factory.setHost("localhost");
		try (Connection connection = factory.newConnection(); Channel channel = connection.createChannel()) {
			channel.queueDeclare(QUEUE_NAME, false, false, false, null);
			String message = id + "/" + status + "/" + check;
			channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
			System.out.println(" [x] Set '" + message + "'");
			Thread.sleep(10);
		} catch (TimeoutException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

```