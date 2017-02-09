title: ckeditor插件开发 

#  ckeditor插件开发 
目录结构![](/data/dokuwiki/web/pasted/20160108-152810.png)
plugin.js
```

(function()
{	
	function showUploadDialog( editor )
	{	
		require(["PDUtilDir/dialog", "PDUtilDir/grid", "PDUtilDir/fileupload/singleFileUpload","PDUtilDir/util","PDUtilDir/tool"],function(Dialog,Grid,SFU,Util,Tool)
		{
			var initGrid = function(){
				
				var config = {
						
		                id:'serchGridpicobjID',
		                placeAt:'encyclopedialPicFListId',            
		                pageSize:5,                                                       
		                index:true,
//						multi:'ture',
						multi:true,
				        pagination:true,
		                layout:[                           
		                     {name:'资源名',field:'resourceName',align:"center",width:"33%"},
		                     {name:'文件类型',field:'resourceTypeName',align:"center",width:"33%"},
		                     {name:'文件大小',field:'resourceSize',align:"center",width:"33%"}
		                ],//end layout	
		                toolbar : [
						   {name : '查询',icon : 'fa fa-search',callback : function(){
							   initGrid();
				           }},
				           {name : '重置',icon : 'fa fa-repeat',callback : function(){
				        	   $('#picNameID').val('');
				        	   initGrid();
				           }}						   
						   ],
						  
						data:[]   
//						data : {type : 'URL',value : getServer() + "/sword/cms/resources/QueryResourcesList"},
//			  			queryParam : {
//			            	'QueryResourceName' : $('#picNameID').val()
//			            }
		         	 
	            };
//				Grid.init(config);//渲染grid
				loadResourcesGrid(config);
			};
			
			var dialog = Dialog({
				
				id : 'uploadDialogInfoID',
				cache : false,
				title : '图片上传',
				pageSize : 5,
				width : '800px',
				url : getServer()+'/static/core/cms/publish/lib/ckeditor/views/uploadImg.html',
				
		  		buttons:[
		  		         {
		  		    	   name:'确定',
		  		    	   callback:function(dialog){
		  		    		   var picWidth=$("#picWidth").val();
		  		    		   var picHeight=$("#picHeight").val();
		  		    		   var style="";
		  		    		   if(picWidth&&picHeight){
		  		    			   style="width:"+picWidth.trim()+"px;height:"+picHeight.trim()+"px";
		  		    		   }else{
		  		    			   style="height:auto";
		  		    		   }
		  		    		  	dialog.hide();
		  		    		  var rows = Grid.getGrid("serchGridpicobjID").getSelectedRow();
		  		    		if (rows && rows.length > 0) {
		  		    			$.each(rows, function(i, row){
		  		    				appendImg(row.resourceLocalPath,style);
		  		    			});
		  		    		}
		  		    		if($("#queryPicLinksID").val()){
		  		    			appendImg($("#queryPicLinksID").val(),style);
		  		    		}
		 					if($("#resourceUrl").val()){
		 						appendImg($("#resourceUrl").val(),style);
		 					}		
		  		    	   }
		   		         }
		  		       ],
		  		       
		  		 afterLoad:function(dialog){
		  			 
		  			initGrid();
		  			initUploadFile(1);//初始化图片上传      
			        var noScopeObj = {};
			        //监听上传标签页保存按钮
			        $("#btn_picUpload_save").on("click",function(e){
			        	initUploadSaveTab("save");
			        	e.preventDefault();
			        });
			      //监听上传标签页继续上传按钮事件
	            	$("#btn_picContinue_upload").on("click", function(e) {
	            		initUploadSaveTab("upload");
	            		e.preventDefault();
	            	});
				        
		          }   	
		       });

			dialog.show();
			
			
			//辅助函数
			 function loadResourcesGrid(config) {
					$.ajax({
						url : getServer() + "/sword/cms/resources/QueryResourcesList",
						async:false,
						dataType:"json",
						method:"POST",
						data:{
							 'QueryResourceName' : $('#picNameID').val()
						},
						success : function(data) {
							if(data.model){
								data=data.model;
							}
							Grid.init($.extend(config, {
								data:data
							}));//渲染grid
						}
					});
			    }
			//btnType值为save,upload.
			function initUploadSaveTab(btnType){
				if ($("#form_picUploadEdit").valid()) {//校验
				  	var data = Tool.serialize("form_picUploadEdit");
					$.ajax({
						url : getServer() + "/sword/cms/resources/SaveOrUpdateResources",
						data : data,
						dataType:"json",
						method:"POST",
						success : function(data) {
							if(data.model){
								data=data.model;
							}
							Util.alert(data.message);
							initGrid();
							if(btnType=="upload"){
								initUploadFile(1);
							}
						}
					});
				}
			}
			 function initUploadFile(resType){
		         $("#testFileUpload1").empty();
		           var settings = {
		               placeAt:"testFileUpload1",
		               allowMC:false,
		               onDeleteSavedFile:function(file){
		                   //console.log(file)
		               },
		               onUploadSuccess:function(file){
		                   console.log(file);
		               	
		           		var filePath = file.swordFilePath;
		           		var fid = file.swordFileId;
		           		$.ajax({
		           			url:getServer() + "/sword/cms/resources/resourcesQueryLocalFilePath",
		   					data : {
		   						fid : fid,
		   						filePath:filePath
		   					},
		   					dataType:"json",
							method:"POST",
		           			success:function(data) {
		           				if(data.model){
									data=data.model;
								}
		           				$("#resourcePath").val(data.resourceFileName);
		           				$("#resourceUrl").val(data.resourceUrl);
		           				$("#resourceFileName").val(file.name);
		                   		var size = file.size/1024;
		                   		$("#resourceSize").val(size.toFixed(2));
		           			}
		           		});
		               }
		           };
		           if(resType==1){//图片
		           	settings.accept={
		   			    extensions: 'gif,jpg,jpeg,bmp,png',
		   			    mimeTypes: 'image/*'
		       		}
		       	}else if(resType==2){//音频
		       		settings.accept={
		       			    extensions: 'mp3,wma,wav,ogg,acc,ape,wmv',
		       			    mimeTypes: 'audio/*'
		           		}
		       	}else if(resType==3){//视频
		       		settings.accept={
		       			    extensions: 'avi,mp4,mpeg,3gp,flv,wmv,mkv,rmvb,rm,m4v,f4v',
		       			    mimeTypes: 'video/*'
		           		}
		       	}
		        SFU.init(settings);
		   	return null;
		   }
			
		});
		function appendImg(link,style){
			var imageElement = editor.document.createElement( 'img' );
				var eleSelpic = editor.getSelection().getStartElement();
				imageElement.setAttribute( 'src',link );
				imageElement.setAttribute( 'style',style );
				eleSelpic.append(imageElement,true);
		}
	}
	
	
	function execlrdbUpload( editor )
	{
		showUploadDialog(editor);
	}
	
	var lrdbUploadCommand = 
	{
		async : true,
		exec : function( editor ) {
			execlrdbUpload( editor );
		}
	};
	
	var uploadPluginN = 'lar_upload';
	
	CKEDITOR.plugins.add(uploadPluginN,
	{
		
		init : function( editor )
		{
			editor.addCommand( uploadPluginN, lrdbUploadCommand);
			editor.ui.addButton('Lar_upload',
			{
				label : "上传图片",
				command : uploadPluginN,
				icon : CKEDITOR.getUrl( this.path + 'images/lar_upload.gif' )
			});
			
			if ( editor.addMenuItems )
			{
				editor.addMenuItems(
				{
					lar_upload :
					{
						label : '上传图片',
						command : uploadPluginN,
						group : uploadPluginN,
						icon : CKEDITOR.getUrl( this.path + 'images/lar_upload.gif' )
					}
				});
			}
			
			if ( editor.contextMenu )
			{
				editor.contextMenu.addListener( function( element, selection )
				{
					return { lar_upload : CKEDITOR.TRISTATE_OFF };
				});
			}

		}
	});
	
})();



```