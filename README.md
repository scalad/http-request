# Http 请求 [![Build Status](https://travis-ci.org/kevinsawicki/http-request.svg)](https://travis-ci.org/kevinsawicki/http-request)

一个基于[HttpURLConnection](http://download.oracle.com/javase/6/docs/api/java/net/HttpURLConnection.html)简单方便的创建HTTP请求并接收响应数据的工具库.

在[MIT License](http://www.opensource.org/licenses/mit-license.php)可以获取到该库

## 使用

http-request这个库可以从[Maven Central](http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.github.kevinsawicki%22%20AND%20a%3A%22http-request%22)获取.

```xml
<dependency>
  <groupId>com.github.kevinsawicki</groupId>
  <artifactId>http-request</artifactId>
  <version>6.0</version>
</dependency>
```

不使用[Maven](http://maven.apache.org/)? 只要复制 [HttpRequest](https://raw.githubusercontent.com/kevinsawicki/http-request/master/lib/src/main/java/com/github/kevinsawicki/http/HttpRequest.java)
类到你的工种当中, 更新包的报名，然后你就可以使用它了.

你可以从 [这里](http://kevinsawicki.github.com/http-request/apidocs/index.html).获取到javadoc文档

## FAQ

### 谁使用它?

[这里](https://github.com/kevinsawicki/http-request/wiki/Used-By)是已知的项目使用该库

### 为什么写这个?

写这个库是为了在使用`HttpURLConnection`来发送HTTP请求的时候更加方便快捷

像[Apache HttpComponents](http://hc.apache.org)这样的组件也是非常好用的，但是有些时候为了更加简单，或者可能因为你部署的环境的问题（比如Android），你只想使用例如`HttpURLConnection`一些过时但是又好用的库。

这个库寻找一种更加方便和一种更加通用的模式来模拟HTTP请求，并支持多种特性的请求，例如多个请求的任务.

### 该库的依赖?

**没有**. 整个库只有单一的一个类和一些静态的内部类，测试工程需要[Jetty](http://eclipse.org/jetty/)的支持，用来实现一个真实的HTTP服务器请求来测试请求.

### 如何管理异常?

`HttpRequest`类不抛出任何检查异常,相反的，所有低等级的异常被包装在继承了`RuntimeException`类的`HttpRequestException`中，你可以通过catching`HttpRequestException`以及调用`getCause`方法来获取潜在的异常信息，它总会返回最原始的`IOException`

### 请求是异步的吗?

**不是**.  每个`HttpRequest`对象都包装了`HttpUrlConnection`这个最根本的对象的一个同步API，因此，所有在`HttpRequest`对象中的方法都是同步的。

因此，不要在你的应用程序的主线程中使用`HttpRequest`对象是非常关键的.

这有一个在Android上使用的小例子
[AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html):

```java
private class DownloadTask extends AsyncTask<String, Long, File> {
  protected File doInBackground(String... urls) {
    try {
      HttpRequest request =  HttpRequest.get(urls[0]);
      File file = null;
      if (request.ok()) {
        file = File.createTempFile("download", ".tmp");
        request.receive(file);
        publishProgress(file.length());
      }
      return file;
    } catch (HttpRequestException exception) {
      return null;
    }
  }

  protected void onProgressUpdate(Long... progress) {
    Log.d("MyApp", "Downloaded bytes: " + progress[0]);
  }

  protected void onPostExecute(File file) {
    if (file != null)
      Log.d("MyApp", "Downloaded file to: " + file.getAbsolutePath());
    else
      Log.d("MyApp", "Download failed");
  }
}

new DownloadTask().execute("http://google.com");
```

## 示例
### 执行一个GET请求并返回响应的状态信息

```java
int response = HttpRequest.get("http://google.com").code();
```

### 执行一个GET请求并返回响应的body信息

```java
String response = HttpRequest.get("http://google.com").body();
System.out.println("Response was: " + response);
```

### 标准打印一个GET请求的响应数据

```java
HttpRequest.get("http://google.com").receive(System.out);
```

### 添加请求参数

```java
HttpRequest request = HttpRequest.get("http://google.com", true, 'q', "baseball gloves", "size", 100);
System.out.println(request.toString()); // GET http://google.com?q=baseball%20gloves&size=100
```

### 使用数组作为请求参数

```java
int[] ids = new int[] { 22, 23 };
HttpRequest request = HttpRequest.get("http://google.com", true, "id", ids);
System.out.println(request.toString()); // GET http://google.com?id[]=22&id[]=23
```

### 和请求/响应的请求头一起使用

```java
String contentType = HttpRequest.get("http://google.com")
                                .accept("application/json") //Sets request header
                                .contentType(); //Gets response header
System.out.println("Response content type was " + contentType);
```

### 执行一个带数据的POST请求并返回响应的状态

```java
int response = HttpRequest.post("http://google.com").send("name=kevin").code();
```

### 使用最基本的认证请求

```java
int response = HttpRequest.get("http://google.com").basic("username", "p4ssw0rd").code();
```

### 执行有文件的请求

```java
HttpRequest request = HttpRequest.post("http://google.com");
request.part("status[body]", "Making a multipart request");
request.part("status[image]", new File("/home/kevin/Pictures/ide.png"));
if (request.ok())
  System.out.println("Status was updated");
```

### 执行一个具有表单数据的POST请求

```java
Map<String, String> data = new HashMap<String, String>();
data.put("user", "A User");
data.put("state", "CA");
if (HttpRequest.post("http://google.com").form(data).created())
  System.out.println("User was created");
```

### 把响应的body信息存放在文件中

```java
File output = new File("/output/request.out");
HttpRequest.get("http://google.com").receive(output);
```
### POST请求包含文件

```java
File input = new File("/input/data.txt");
int response = HttpRequest.post("http://google.com").send(input).code();
```

### POST请求字符串二进制文件

```java
String base64 = base64(t[i].getAbsolutePath());
String body = HttpRequest.post(url)
	.part("base64Strs", new ByteArrayInputStream(base64.getBytes())).body();
```

### 使用实体标签进行缓存

```java
File latest = new File("/data/cache.json");
HttpRequest request = HttpRequest.get("http://google.com");
//Copy response to file
request.receive(latest);
//Store eTag of response
String eTag = request.eTag();
//Later on check if changes exist
boolean unchanged = HttpRequest.get("http://google.com")
                               .ifNoneMatch(eTag)
                               .notModified();
```

### 使用gzip压缩方式

```java
HttpRequest request = HttpRequest.get("http://google.com");
//Tell server to gzip response and automatically uncompress
request.acceptGzipEncoding().uncompress(true);
String uncompressed = request.body();
System.out.println("Uncompressed response is: " + uncompressed);
```

### 当使用HTTPS协议是忽略安全

```java
HttpRequest request = HttpRequest.get("https://google.com");
//Accept all certificates
request.trustAllCerts();
//Accept all hostnames
request.trustAllHosts();
```

### 配置一个HTTP请求代理

```java
HttpRequest request = HttpRequest.get("https://google.com");
//Configure proxy
request.useProxy("localhost", 8080);
//Optional proxy basic authentication
request.proxyBasic("username", "p4ssw0rd");
```

### 重定向

```java
int code = HttpRequest.get("http://google.com").followRedirects(true).code();
```

### 自定义连接工场

查看 [OkHttp](https://github.com/square/okhttp)来使用该库?
查看 [here](https://gist.github.com/JakeWharton/5797571).

```java
HttpRequest.setConnectionFactory(new ConnectionFactory() {

  public HttpURLConnection create(URL url) throws IOException {
    if (!"https".equals(url.getProtocol()))
      throw new IOException("Only secure requests are allowed");
    return (HttpURLConnection) url.openConnection();
  }

  public HttpURLConnection create(URL url, Proxy proxy) throws IOException {
    if (!"https".equals(url.getProtocol()))
      throw new IOException("Only secure requests are allowed");
    return (HttpURLConnection) url.openConnection(proxy);
  }
});
```

## 贡献

* [Kevin Sawicki](https://github.com/kevinsawicki) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=kevinsawicki)
* [Eddie Ringle](https://github.com/eddieringle) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=eddieringle)
* [Sean Jensen-Grey](https://github.com/seanjensengrey) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=seanjensengrey)
* [Levi Notik](https://github.com/levinotik) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=levinotik)
* [Michael Wang](https://github.com/michael-wang) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=michael-wang)
* [Julien HENRY](https://github.com/henryju) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=henryju)
* [Benoit Lubek](https://github.com/BoD) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=BoD)
* [Jake Wharton](https://github.com/JakeWharton) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=JakeWharton)
* [Oskar Hagberg](https://github.com/oskarhagberg) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=oskarhagberg)
* [David Pate](https://github.com/DavidTPate) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=DavidTPate)
* [Anton Rieder](https://github.com/aried3r) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=aried3r)
* [Jean-Baptiste Lièvremont](https://github.com/jblievremont) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=jblievremont)
* [Roman Petrenko](https://github.com/romanzes) :: [contributions](https://github.com/kevinsawicki/http-request/commits?author=romanzes)
