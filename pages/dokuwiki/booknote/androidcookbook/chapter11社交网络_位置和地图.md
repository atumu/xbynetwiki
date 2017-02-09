title: chapter11社交网络_位置和地图 

#   社交网络 

Created Monday 11 January 2010

##  用JSON加载用户的Twitter动态 

-------------------
```

StringBuilder sb=new StringBuilder();
HttpClient client=new DefaultHttpClient();
HttpGet get=new HttpGet("http://twitter.com/..../.....json");
try{
	HttpResponse response=client.execute(get);
	StatusLine statusLine=response.getStatusLine();
	int statusCode=statusLine.getStatusCode();
	if(statusCode==200){
		HttpEntity entity=response.getEntity();
		InputStream content=entity.getContent();
		BufferedReader reader=new BufferedReader(new InputStreamReader(content));
		...
	}
}

```


#  位置和地图应用程序 
Created Monday 11 January 2010

##  获得位置信息 
解决：使用android内建的位置提供器
Android提供两个级别的位置信息：基于GPS的FINE；以及基于基站的COARSE
示例程序OpenStreetMap的JPStrack
获取位置数据：
```

LocationManager locm=(LocationManager)getSystemService(LOCATION_SERVICE);
for(String prov: locm.getAllProviders){
	Log.i(TAG,prov);
}
//GPS设置
Criteria criteria=new Criteria();
criteria.setAccuracy(Criteria.ACCURACY_FINE);
List<String> providers=locm.getProviders(criteria,true);
if(providers==null || providers.size()==0){

}
String preferred=providers.get(0);

暂停和恢复位置更新
protected void onResume(){
super.onResume();
if(preferred!=null){
	locm.requestLocationUpdates(preferred,MIN_SECONDS*1000,MIN_METRES,this);
}
}
protected void onPause(){
	super.onPause();
	if(preferred!=null){
	//节约电量
		locm.removeUpdates(this);
	}
}
最后,LocationListener的onLocationChanged()方法在位置变化时调用。可以在这里对位置信息进行某些操作。
public void onLocationChanged(Location loc){
	loc.getTime();
	loc.getLatitude();
	loc.getLongitude();
	..
}

```
##  在应用程序中访问GPS信息 
解决:LocationListener接口的类。
LocationManager.GPS_PROVIDER;getLastKnownLocation;

##  使用地理解析（Geocoding）和反向地理解析 
使用内建的android.location.Geocoder类
**不应该在UI线程中执行相关解析操作**
```

Geocoder gc=new Geocoder(context);
if(gc.isPresent()){
	List<Address> list=gc.getFromLocationName(".......",1);
	Address address=list.get(0);
	double lt=address.getLatitude();
	double lng=address.getLongitude();
	
}
反向解析：
List<Address> list=gc.getFromLocation(37.1123,-111.
Address address=list.get(0);
...............................

```
##  准备Google Maps开发 
问题：你希望在Android应用中使用Google MapView布局元素
解决：使用Google Map API库，MapView布局元素和MapActivity
```

<com.google.android.maps.MapView
	android:id=
	android:layout_width=
	android:layout_height=
	android:apiKey=
	**android:clickable="true"/>**
	mapView.setBuiltInZoomControls(true);
	//设置为卫星地图 mapView.setSatellite(true);

```
####  将设备当前位置添加到Google Maps，使用com.google.android.maps.MyLocationOverlay类。可以在地图上绘制设备的当前位置。 
添加权限：
''android.permission.INTERNET
android.permission.ACCESS_FINE_LOCATION
android.permission.ACCESS_COARSE_LOCATION

myLocationOverlay=new MyLocationOverlay(this,mapView);
mapView.getOverlays().add(myLocationOverlay);
**mapView.invalidate();**
注意：
onPuse(){
super.onPause();
myLocationOverlay.diableMyLocation();
}
onResume(){
super.onResume();
myLocationOverlay.enableMyLocation();
}''

####  在Google MapView上绘制位置标志 
创建一个Overlay实例，绘制你的标志，然后将其添加到MapView图层中。移动到指定的地理位置。
GeoPoint参数使用整数而非浮点数，所以可以乘1E6.表示一个地图点
''MapController mc=mapView.getController();
mc.setZoom(18);
mc.animateTo(geoPoint);''
通过扩

