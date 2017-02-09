title: jquery练习代码记录 

#  jquery练习代码记录 
##  enter.js 
```

/**为了防止多个入口js往header.html导入的情况，在此统一定义一个入口js。
 * 对于需要特定效果的入口js代码段可以放在此处定义一个函数，然后在其中被调用。，统一导入header.html
 * 记住，加好注释，分好行。
 * 
 * 这种解决措施设计上确实存在问题。只是作为一种暂时的解决方案，或许可能会有改进。
 */
$(document).ready(function(){
		/**定义的初始化入口函数放在此处*/
		function init(){
			initImgFadeNew();
			//发布页面中，导航菜单图片切换功能
			//switchMenuImg();
			//初始化栏目列表插件分页逻辑
			initPagePlugin();
		}
		/**初始化栏目列表插件分页逻辑*/
		function initPagePlugin(){
			//调用分页插件进行渲染。
			$(".p-cl-page").each(function(i,e){
				var pluginData={};
				pluginData.newsNo=$(e).parent(".channelListTemplate:first").find(".channelListNewsNo").val();
				pluginData.dictNewsSort=$(e).parent(".channelListTemplate:first").find(".channelListDictNewsSort").val();
				pluginData.dictNewsType=$(e).parent(".channelListTemplate:first").find(".channelListDictNewsType").val();
				
				//查询分页总数
				$.ajax({
					url:getServer() + '/news/queryNewsBySortPageCount',
					data:{
						dictNewsSort:pluginData.dictNewsSort,
						pageSize:pluginData.newsNo,
						dictNewsType:pluginData.dictNewsType
					},
					async:false,
					success:function(d){
						var ss = $.parseJSON(d);
						pluginData.pageCount=ss.pageCount;
					}
				});
				$(this).pagePlugin({
				 	page:1, //当前页
				 	pageCount:pluginData.pageCount,//页面大小
		            onPaged:function(pageNo){  //回调函数,pageNO为需要请求的页面number
		            	initPageArgs(pluginData,pageNo,$(e).parent(".channelListTemplate:first"));
		            }
				});
			});
		}
		
		/**初始化图片横幅淡入淡出效果代码*/
		function initImgFadeNew(){
		 	var roll = ".p-sl-rolling";
	    	//匀速运动
	    	var move = ".p-sl-uniformMove";
	    	var fade = ".p-sl-fade";		
	    	if($( roll ).length>0){
	    		$(".dotMask").remove();
	    		$( roll ).setImgMove({
	             speed : '2',
	             status:'public' 
	         });	
	    	}  						
	    	if($(fade).length>0){
//	    		$('#body').slider();
	    		var imgMove=$('.imgMove');
	    		for(var i=0;i<imgMove.length;i++){
	    			$(imgMove[i]).slider();
	    			//解决发布时，图片横幅在IE 火狐高度未设置的问题
	    			var tempheight = $(imgMove[i]).css("height");
	    			$(".p-slider").css("height",tempheight);
	    		}
//	    		$('.imgMove').slider();
//	    		$( fade ).setImgMove({
//	             speed : '2',
//	             status:'template' 
//	         });		                             		 
	    	}   
	    	if($( move ).length>0){
	    		var uniformMove = $(move);
	    		for(var i=0;i<uniformMove.length;i++){
	    			$(uniformMove[i]).setImgMove({
	    				speed:'2',
	    				status:'public'
	    			});
	    		}
	    	} 	                	
	    	
	
		}
		
		/**导航菜单图片切换效果*/
		function switchMenuImg(){
			//绑定鼠标穿过事件 modify by wangh
			$(' .pluginBody .navMenu li img').bind("mouseenter",function(event){
				var _id = event.currentTarget.id;
				var temp = $("#switchImgUrl"+_id);
				var switchImgUrl = temp.attr("value");
				//存在切换的图片，进行图片的替换
				if(switchImgUrl!=null&&switchImgUrl!=''){
					$("#"+_id).attr("src",switchImgUrl);
				}
			});
			//绑定鼠标离开事件 modify by wangh
			$(' .pluginBody .navMenu li img').bind("mouseleave",function(event){
				var _id = event.currentTarget.id;
				var temp = $("#imgUrl" + _id);
				var imgUrl = temp.attr("value");
				//存在背景图片，进行图片的显示
				if(imgUrl!=null&&imgUrl!=''){
					$("#"+_id).attr("src",imgUrl);
				}
			});
		}
		   
		/**
		 * 用于分页插件初始化参数获取。
		 * pluginData:数据传输对象
		 * pageNo:要请求的页面
		 * */
		function initPageArgs(pluginData, pageNo,$channelListTemp){
		
			//分页请求
			$.ajax({
				url:getServer() + '/news/queryNewsBySortPage',
				data:{
					dictNewsSort:pluginData.dictNewsSort,
					pageNumber:pageNo,
					pageSize:pluginData.newsNo,
					dictNewsType:pluginData.dictNewsType
				},
				async:false,
				success:function(d){
					var ss = $.parseJSON(d);
					var currentData = ss.curPageData;
					var newsThumbnail;
					var pictureUrl;
					for(var i=0;i<pluginData.newsNo&&i<currentData.length;i++){
						//每次查询初始化
						newsThumbnail = "";
						pictureUrl = "";
						if(currentData[i].newsThumbnail!=null&&currentData[i].newsThumbnail!=''){
							newsThumbnail = currentData[i].newsThumbnail;
							//获取fileId和bizType
							var fileId;
							var bizType;
                        	var start = newsThumbnail.indexOf("=");
                            var end = newsThumbnail.indexOf("&");
                            var last = newsThumbnail.lastIndexOf("=");
                            fileId = newsThumbnail.substring(start+1,end);
                            bizType = newsThumbnail.substring(last+1,newsThumbnail.length);
                            pictureUrl = getServer() + '/file/downloadPic?fileId=' + fileId + '&bizType=' + bizType;
						}
						pluginData.newsThumbnail = newsThumbnail;
						pluginData.newsTitle =currentData[i].newsTitle;
//						pluginData.margin=pluginData.newsSpace/2;
						pluginData.pictureUrl=pictureUrl;
						pluginData.link=currentData[i].publishPath;
//						console.log(ele);
//						$channelListTemp=$(ele).parent("div.p-cl-page:first").parent("div.channelListTemplate:first");
						$channelListTemp.find("div.p-cl-channelSingleNews:eq("+i+")").css("display","");
						$channelListTemp.find("div.p-cl-channelSingleNews:eq("+i+")").find("a").attr("href",pluginData.link);
						$channelListTemp.find("div.p-cl-channelSingleNews:eq("+i+")").find("a:eq(1)").attr("title",pluginData.newsTitle).text(pluginData.newsTitle);	
						$channelListTemp.find("div.p-cl-channelSingleNews:eq("+i+")").find("img").attr("src",pluginData.pictureUrl);
					}
					for(var i=currentData.length;i<pluginData.newsNo;i++){
						$channelListTemp.find("div.p-cl-channelSingleNews:eq("+i+")").css("display","none");
//						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("a").attr("href","");
//						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("a:eq(1)").attr("title","").text("");	
//						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("img").attr("src","");
					}
					
				}
			});
		};
		function getServer(){
//			return 'http://localhost:8081/wyjg'; //服务器路径。
			return 'http://10.1.100.163/wyjg'; //服务器路径。
		}
	    return init();
});

```
ChannelList.js
```

define(['Plugin','Mustache', 'TemplateUtil', 'Config','UtilPath/dropList','PagePlugin','css!' + getPublishServer() + '/plugin/common/pagePlugin/pagePlugin.css'], function (Plugin,mustache, tUtil, config,dropList,PagePlugin) {
	function channelList(config){
		Plugin.call(this, config);
		var _this = this;
		
		
		this.setInitPage = function() {
			//新闻类别列表数据加载	
			_this.channelList();
			
			//保存按钮点击事件
			$("#myplugin").bind("click",function(){
				//输入校验
				if($("#dictSort").valid()&&$("#newsNo").valid()&&$("#singleNewsNo").valid()){
						require(['css!' + getPublishServer() + '/plugin/slider/slider.css'], function(){
							_this.setPlugin();
							
						});
				}else{
					alert('input error');
				}
			});
			
		};
		
		//编辑回调页面
		this.setEditPage = function( data ){
			//新闻类别列表数据加载	
			_this.channelList();
			$("#sortId").val(data.content.dictNewsSort);
			$("#dictSort").val(data.content.sortName);
			$("#newsNo").val(data.content.newsNo);
			$("#newsSpace").val(data.content.newsSpace);
			$("#singleNewsNo").val(data.content.singleNewsNo);
			$("#dictNewsType").val(data.content.dictNewsType);
			var flag=data.content.titlePosition;
			var selectEffect="input[name='titlePosition']";
			if(flag=="merge"){
				$(selectEffect+"[value='merge']").prop("checked",true);
			}else{
				$(selectEffect+"[value='separate']").prop("checked",true);
			}
			//保存按钮点击事件
			$("#myplugin").bind("click",function(){
				//输入校验
				if($("#dictSort").valid()&&$("#newsNo").valid()&&$("#singleNewsNo").valid()){
						_this.setPlugin('edit');
				}else{
					alert('input error');
				}
			});
			
			
		};
		/**
		 * 用于分页插件初始化参数获取。
		 * pluginData:数据传输对象
		 * pageNo:要请求的页面
		 * isCallBack是否处于分页回调环境
		 * */
		this.initPageArgs=function(pluginData, pageNo,isCallBack){
			
			//分页请求
			$.ajax({
				url:getServer() + '/news/queryNewsBySortPage',
				data:{
					dictNewsSort:pluginData.dictNewsSort,
					pageNumber:pageNo,
					pageSize:pluginData.newsNo,
					dictNewsType:pluginData.dictNewsType
				},
				async:false,
				success:function(d){
					var ss = $.parseJSON(d);
					var currentData = ss.curPageData;
					var newsThumbnail;
					var pictureUrl;
					for(var i=0;i<pluginData.newsNo&&i<currentData.length;i++){
						//每次查询初始化
						newsThumbnail = "";
						pictureUrl = "";
						if(currentData[i].newsThumbnail!=null&&currentData[i].newsThumbnail!=''){
							newsThumbnail = currentData[i].newsThumbnail;
							//获取fileId和bizType
							var fileId;
							var bizType;
                        	var start = newsThumbnail.indexOf("=");
                            var end = newsThumbnail.indexOf("&");
                            var last = newsThumbnail.lastIndexOf("=");
                            fileId = newsThumbnail.substring(start+1,end);
                            bizType = newsThumbnail.substring(last+1,newsThumbnail.length);
                            pictureUrl = getServer() + '/file/downloadPic?fileId=' + fileId + '&bizType=' + bizType;
						}
						if(!isCallBack){
							pluginData.news.push({
								newsThumbnail: newsThumbnail,
								newsTitle : currentData[i].newsTitle,
								margin:pluginData.newsSpace/2,
								pictureUrl:pictureUrl,
								link:currentData[i].publishPath
							});
						}
					
						pluginData.newsThumbnail = newsThumbnail;
						pluginData.newsTitle =currentData[i].newsTitle;
						pluginData.margin=pluginData.newsSpace/2;
						pluginData.pictureUrl=pictureUrl;
						pluginData.link=currentData[i].publishPath;
						
						    $('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").css("display","");
							$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("a").attr("href",pluginData.link);
    						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("a:eq(1)").attr("title",pluginData.newsTitle).text(pluginData.newsTitle);	
    						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("img").attr("src",pluginData.pictureUrl);
					}
					for(var i=currentData.length;i<pluginData.newsNo;i++){
						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").css("display","none");
//						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("a").attr("href","");
//						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("a:eq(1)").attr("title","").text("");	
//						$('#' + _this.id +" div.p-cl-channelSingleNews:eq("+i+")").find("img").attr("src","");
					}
					
				}
			});
		};
		this.setPlugin = function( status , pluginData ) {
			//如果有data 直接从数据渲染插件
			var pluginInfo = this.dataSupport( status , pluginData );
			
			$.get( getPublishServer() + '/plugin/channelList/channelListTemplate.html' , function( channelList ){	
				var pluginData=pluginInfo.content;
				//查询分页总数
				$.ajax({
					url:getServer() + '/news/queryNewsBySortPageCount',
					data:{
						dictNewsSort:pluginData.dictNewsSort,
						pageSize:pluginData.newsNo,
						dictNewsType:pluginData.dictNewsType
					},
					async:false,
					success:function(d){
						var ss = $.parseJSON(d);
						pluginData.pageCount=ss.pageCount;
					}
				});
				if('load'!=status){
					_this.initPageArgs(pluginData,1,false);
				}
				//模板渲染
				var output = mustache.render(channelList,pluginData);
				$( '#' + _this.id + ' .plugin').remove();
				$( '#' + _this.id + ' .pluginBody').empty();
				$( '#' + _this.id + ' .pluginBody').append(output);
				//调用分页插件进行渲染。
				$('#' + _this.id +" .p-cl-page").pagePlugin({
				 	page:1, //当前页
				 	pageCount:pluginData.pageCount,//页面大小
		            onPaged:function(pageNo){  //回调函数,pageNO为需要请求的页面number
//		            	alert(pageNo);
		            	_this.initPageArgs(pluginData,pageNo,true);
		            }
				});
				//获取当前区域的大小
				var height = $( '#' + _this.id + ' .pluginBody').height();
				var width = $('#' + _this.id + ' .pluginBody').width();
				//自定义标题高度 没有写死可调整
				var titleHeight = 50;
				
				//获取显示的图片数量和图片间距
				var newsNo = pluginData.newsNo;
				var newsSpace = pluginData.newsSpace;
				var singleNewsNo = pluginData.singleNewsNo;
				//单个新闻占用的宽度
				var imgAdpWidth = width/parseInt(singleNewsNo)-parseInt(newsSpace)-10;
				var $imgs = $('#' + _this.id).find('img');
				//等待最新图片加载完成进行图片缩放调整
				var n = 1;
				$($imgs[0]).load(function(){
				    if(!--n){
				        // 加载完成
						var percent = $imgs[0].width/imgAdpWidth;
						var imgAdpHeight = $imgs[0].height/percent;
						for(var i=0;i<$imgs.length;i++){
							$('#' + _this.id + " img:eq("+i+")").css({"height":imgAdpHeight,"width":imgAdpWidth});
							$('#' + _this.id + " .p-cl-channelSingleNewsTitle:eq(" + i + ")").css({"width":imgAdpWidth});
							if(i>0&&i%parseInt(singleNewsNo)==0){
								//换行 清除float
								$('#' + _this.id + " .p-cl-channelSingleNews:eq(" + i + ")").css({"clear":"both"});
							}
						}
						//向上取整求行数
						var heightNo = Math.ceil(parseInt(newsNo)/parseInt(singleNewsNo));
						var totalHeight = (imgAdpHeight+titleHeight) * heightNo+26;//26是分页插件的高度
						if(totalHeight>height){
							imgAdpHeight = height/heightNo - titleHeight;
							percent = $imgs[0].height/imgAdpHeight;
							imgAdpWidth = $imgs[0].width/percent;
							for(var i=0;i<$imgs.length;i++){
								$('#' + _this.id + " img:eq("+i+")").css({"height":imgAdpHeight,"width":imgAdpWidth});
								$('#' + _this.id + " .p-cl-channelSingleNewsTitle:eq(" + i + ")").css({"width":imgAdpWidth});
								if(i>0&&i%parseInt(singleNewsNo)==0){
									//换行 清除float
									$('#' + _this.id + " .p-cl-channelSingleNews:eq(" + i + ")").css({"clear":"both"});
								}
							}
						}
				    }
				});
			});
		}
		
		
		//插件数据处理中心
		this.getData = function(){
			var data = {
					type : this.type,
					content : null
			}		
			var dictNewsSort = $("#sortId").val();
			var sortName = $("#dictSort").val();
			var newsNo = $("#newsNo").val();
			var singleNewsNo = $("#singleNewsNo").val();
			var newsSpace = $("#newsSpace").val();
			var titlePosition = $("input[name='titlePosition']:checked").val();
			var dictNewsType = $("#dictNewsType").val();
			var view = {
					news : [],
					dictNewsSort:dictNewsSort,
					sortName:sortName,
					newsNo:newsNo,
					newsSpace:newsSpace,
					titlePosition:titlePosition,
					singleNewsNo:singleNewsNo,
					pageCount:1,
					dictNewsType:dictNewsType
			};
			data.content = view;
			return data;
		}
		
		
		
		//新闻类别列表
		this.channelList = function (val){
			var data =null;
			$.ajax({ 
		          type : "post", 
		          url : getServer() + '/dictTree/queryByType?dictType=D_NEWS_SORT',
		          async : false, 
		          success : function(d){ 
		          	var ss =  $.parseJSON(d);
		            if("200" == ss.status){
						dropList({
			                id:"dictSort",
			                type : "select",
			                key : {
			                    id : "id",
			                    name : "name",
			                    data : "data"
			                },
			                data :ss.curPageData,
			                initData:[val],
			                searchAble : true,
			                callback : function(id, data) {
			                	if(data!=null&&data!=''){
			                		$("#sortId").val(data[0].id);
			                	}
			                }					
						});	
					}
		          } 
			}); 	
		}
	}
	channelList.prototype = Plugin.prototype;
	return channelList;
	
	
});

```
templateCtrl.js
```

define(["jquery","UtilDir/util","UtilDir/grid","app/template/templateSupport"],function($,util,grid,service){

	return function($http, $scope){
		
		$scope.$apply(function(){
			
			$scope.template = {
					template:{
						editRole: getServer() + "/static/app/template/views/templateEdit.html"
					},
					query:{
						templateName:"",
						templateType:""
					},
					type:[]
			};
		});
		service.queryTemplateType($scope);
		service.templateListInit($scope);
	};
});

```
