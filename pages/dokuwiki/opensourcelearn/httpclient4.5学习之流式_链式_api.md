title: httpclient4.5学习之流式_链式_api 

#  HttpClient4.5学习之流式（链式）API 
HttpClient 4.2 中 简化了基于流式接口的外观API。
```

// Execute a GET with timeout settings and return response content as String.
Request.Get("http://somehost/")
    .connectTimeout(1000)
    .socketTimeout(1000)
    .execute().returnContent().asString();


// Execute a POST with the 'expect-continue' handshake, using HTTP/1.1,
// containing a request body as String and return response content as byte array.
Request.Post("http://somehost/do-stuff")
    .useExpectContinue()
    .version(HttpVersion.HTTP_1_1)
    .bodyString("Important stuff", ContentType.DEFAULT_TEXT)
    .execute().returnContent().asBytes();


// Execute a POST with a custom header through the proxy containing a request body
// as an HTML form and save the result to the file
Request.Post("http://somehost/some-form")
    .addHeader("X-Custom-header", "stuff")
    .viaProxy(new HttpHost("myproxy", 8080))
    .bodyForm(Form.form().add("username", "vip").add("password", "secret").build())
    .execute().saveContent(new File("result.dump"));

```
响应处理
fluent facade API简化了使用者处理连接管理器和分配资源。在大多数情况下，它付出了在内存中缓存响应的代价。` 强烈建议使用ResponseHandler避免在内存中缓存响应。 `
```

Document result = Request.Get("http://somehost/content")
        .execute().handleResponse(new ResponseHandler<Document>() {
    public Document handleResponse(final HttpResponse response) throws IOException {
        StatusLine statusLine = response.getStatusLine();
        HttpEntity entity = response.getEntity();
        if (statusLine.getStatusCode() >= 300) {
            throw new HttpResponseException(
                statusLine.getStatusCode(),
                statusLine.getReasonPhrase());
        }
        if (entity == null) {
            throw new ClientProtocolException("Response contains no content");
        }
        DocumentBuilderFactory dbfac = DocumentBuilderFactory.newInstance();
        try {
            DocumentBuilder docBuilder = dbfac.newDocumentBuilder();
            ContentType contentType = ContentType.getOrDefault(entity);
            if (!contentType.equals(ContentType.APPLICATION_XML)) {
                throw new ClientProtocolException("Unexpected content type:" +
                    contentType);
            }
            String charset = contentType.getCharset();
            if (charset == null) {
                charset = HTTP.DEFAULT_CONTENT_CHARSET;
            }
            return docBuilder.parse(entity.getContent(), charset);
        } catch (ParserConfigurationException ex) {
            throw new IllegalStateException(ex);
        } catch (SAXException ex) {
            throw new ClientProtocolException("Malformed XML document", ex);
        }
    }
    });

```