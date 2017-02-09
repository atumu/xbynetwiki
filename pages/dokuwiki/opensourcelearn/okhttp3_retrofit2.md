title: okhttp3_retrofit2 

#  okhttp3+retrofit2进行HTTP请求 
采用retrofit2本身可以进行优雅的RESTFul请求,但是无法设置请求超时时间,需要配合okhttp3来设置请求超时.
```

<retrofit-version>2.0.2</retrofit-version>
<okhttp-version>3.3.1</okhttp-version>
<dependency>
				<groupId>com.squareup.retrofit2</groupId>
				<artifactId>retrofit</artifactId>
				<version>${retrofit-version}</version>
			</dependency>
			<dependency>
				<groupId>org.ligboy.retrofit2</groupId>
				<artifactId>converter-fastjson</artifactId>
				<version>${retrofit-version}</version>
			</dependency>
			<dependency>
				<groupId>com.squareup.okhttp3</groupId>
				<artifactId>okhttp</artifactId>
				<version>${okhttp-version}</version>
			</dependency>

```

相关网站:
http://square.github.io/retrofit/
https://github.com/square/retrofit

https://github.com/square/okhttp
http://square.github.io/okhttp/

```

	//Retrofit请求接口
	public interface CmsManager{   
		@GET("/sword/cms/web/sitesync/site")
		Call<JSONObject> verifySite(@QueryMap Map<String,String> params);
	}
	public static void sync(SyncMeta meta){
		Map<String,String> map=new HashMap<>();
		map.put("siteKey", meta.getSiteKey());
		map.put("siteMd5", meta.getSiteMd5());
		log.info("site key is:{} ;; site md5 is :{}",meta.getSiteKey(),meta.getSiteMd5());
		
		OkHttpClient.Builder httpBuilder=new OkHttpClient.Builder();
		OkHttpClient client=httpBuilder.readTimeout(3, TimeUnit.MINUTES)
				.connectTimeout(3, TimeUnit.MINUTES).writeTimeout(3, TimeUnit.MINUTES) //设置超时
				.build();
		Retrofit retrofit=new Retrofit.Builder().baseUrl(meta.getRemoteUrl()) //设置请求路径
				.addConverterFactory(FastJsonConverterFactory.create()) //设置转换器为FastJson提供的工厂
				.client(client).build(); //设置请求客户端为OkHttpClient
		CmsManager cms=retrofit.create(CmsManager.class);
		Call<JSONObject> call=cms.verifySite(map);
		//异步请求方式
//		call.enqueue(new Callback<JSONObject>() {
//			@Override
//			public void onResponse(Call<JSONObject> arg0, Response<JSONObject> arg1) {
//				JSONObject obj=arg1.body();
//				log.info("向cms管理端请求成功，返回值为{}",obj==null?"":obj.toJSONString());
//			}
//			@Override
//			public void onFailure(Call<JSONObject> arg0, Throwable arg1) {
//				log.error("向cms管理端请求失败",arg1);
//			}
//		});
		try {
			//同步请求方式
			Response<JSONObject> resp=call.execute();
			JSONObject obj=resp.body();
			log.info("向cms管理端请求成功，返回值为{}",obj==null?"":obj.toJSONString());
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			log.error("向cms管理端请求失败",e);
		}
		
	}

```

参考:
http://stackoverflow.com/questions/29380844/how-to-set-a-timeout-in-retrofit-library