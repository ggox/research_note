### URI & URL

##### 1. uri 组成部分：

`[scheme:]<scheme-specific-part>[#fragment]`

* Scheme name

* Scheme-specific part

  `<scheme-specfic-part>=[//authority]<path>[:query]`

  `authority=[userinfo@]<host>[:port]`

* Fragment

##### 2. 代码示例：

```java
@Test
public void uri_research() throws URISyntaxException {
  //创建一个uri,根据[scheme:]<scheme-specific-part>[#fragment]
  URI uri = new URI("http", "//qinlin@localhost:8080/oi/oi?user='aieg'", "iewio");
  // all
  Assert.assertEquals("http://qinlin@localhost:8080/oi/oi?user='aieg'#iewio",uri.toString());
  // schema
  Assert.assertEquals("http",uri.getScheme());
  // ssp
  Assert.assertEquals("//qinlin@localhost:8080/oi/oi?user='aieg'",uri.getSchemeSpecificPart());
  // fragment
  Assert.assertEquals("iewio",uri.getFragment());
  // authority
  Assert.assertEquals("qinlin@localhost:8080", uri.getAuthority());
  // userInfo
  Assert.assertEquals("qinlin",uri.getUserInfo());
  // host
  Assert.assertEquals("localhost",uri.getHost());
  // port
  Assert.assertEquals(8080, uri.getPort());
  // path
  Assert.assertEquals("/oi/oi",uri.getPath());
}
```

```java
@Test
public void url_research() throws IOException {
  URL url = new URL("http://qinlin@localhost:8080/oi/oi?user='aieg'#iewio");
  // protocol
  Assert.assertEquals("http",url.getProtocol());
  // authority
  Assert.assertEquals("qinlin@localhost:8080", url.getAuthority());
  // userInfo
  Assert.assertEquals("qinlin",url.getUserInfo());
  // host
  Assert.assertEquals("localhost",url.getHost());
  // port
  Assert.assertEquals(8080, url.getPort());
  // path
	Assert.assertEquals("/oi/oi", url.getPath());
}
```

