## Overview
- HTTP是app常用的连接网络方式，是我们交换数据和媒体的方式。提高使用HTTP的效率可以令你的工作提高速度并节省带宽
- OkHttp通常是一个高效的HTTP客户端
	- 支持HTTP/2，这可以令所有的对主机的请求都共用一个socket
	- 如果不支持HTTP/2，那么可以进行连接池化，从而减少请求等待
	- 透明的GZIP，可缩小下载大小
	- 响应缓存避免了重复的请求
- OkHttp在网络出现问题时依然坚持不懈
	- 它会静静地从连接中恢复
	- 如果你的服务有多个IP地址，那么OkHttp会在连接失败的情况下尝试这些可选的地址。这对IPv4+IPv6和在不同的数据中心提供了相同服务而言有必要
	- OkHttp支持最近推出的TLS特性（TLS1.3，ALPN，证书固定）
- OkHttp是易用的，它的请求-响应API设计有流畅的构建器和不变性。它既支持同步阻塞调用，也支持异步调用

#### 获取URL
```Java
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
	Request request = new Request.Builder()
		.url(url)
		.build();
	
	try (Response response = client.newCall(request).execute()) {
		return response.body().string();
	}
}
```

#### 发送至服务器
```Java
public static final MediaType JSON
    = MediaType.get("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
	RequestBody body = RequestBody.create(json, JSON);
	Request request = new Request.Builder()
		.url(url)
		.post(body)
		.build();
	try (Response response = client.newCall(request).execute()) {
		return response.body().string();
	}
}
```

#### 限制
- OkHttp必须工作在Android5.0+、Java8+
- OkHttp基于实现高性能I/O的Okio和Kotlin基础库。两者的库虽小，但是向后兼容性极佳
- 我们十分推荐保持使用最新的OkHttp
	- 与自动更新的Web浏览器保持一致，与HTTPS客户端保持同步是防止潜在安全问题的的重要措施
	- 我们追踪了动态TLS生态并调整OkHttp以适应连接和安全
- OkHttp使用平台建立的TLS实现。在Java平台上OkHttp还支持Conscrypt，它将Java与BoringSSL集成一起