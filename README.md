<%@ page language="java" contentType="text/html; charset=UTF-8"%>
<%@ page import="com.gxd.datacenter.context.*" %>
<%@ page import="com.gxd.datacenter.domain.*" %>
<%@ page import="java.util.*" %>
<%@ page import="net.sf.json.JSONArray" %>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
String serverPath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort();

Staff staff = null;
String staffId = "";
if(session.getAttribute(SessionParameter.LOGIN_USER)!=null){
	staff = (Staff)session.getAttribute(SessionParameter.LOGIN_USER);
	staffId = staff.getStaffId()+"";
}
String themes = "bootstrap";
String magtingId = (String)request.getAttribute("magtingId");
System.out.println("magtingId:"+magtingId);
List list = new ArrayList();
list.add("http://192.168.30.23:8090/g1/M01/00/00/wKgeGFg1QWiALrbRAAQ2mosy21E795.jpg");
list.add("http://192.168.30.23:8090/g1/M00/00/00/wKgeGFg1QXeAT0NxAAOOlOl1jes586.jpg");
list.add("http://192.168.30.23:8090/g1/M01/00/00/wKgeGFg1QYOAZhM9AASybsur4xw571.jpg");
list.add("http://192.168.30.23:8090/g1/M00/00/00/wKgeGFg1QY2AFV-RAAQ3H9pN9E8079.jpg");
list.add("http://192.168.30.23:8090/g1/M01/00/00/wKgeGFg1QZiAbyeLAAanDCZLJiE184.jpg");

JSONArray array = new JSONArray();
for(int i=0;i<list.size();i++){
	array.add(list.get(i));
}
//request.setAttribute("array", array);
int ii = list.size();
%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>押品人工处理平台</title>
<script src="<%=serverPath %>/arcgis_js_api/library/3.11/3.11/init.js" type="text/javascript"></script>
<link href="<%=serverPath %>/arcgis_js_api/library/3.11/3.11/dijit/themes/tundra/tundra.css" rel="stylesheet" type="text/css" />
<link href="<%=serverPath %>/arcgis_js_api/library/3.11/3.11/esri/css/esri.css" rel="stylesheet" type="text/css" />

<script src="<%=basePath %>res/easyui/jquery-1.7.2.min.js" type="text/javascript"></script>
<script src="<%=basePath %>res/easyui/jquery.easyui.min.js" type="text/javascript"></script>
<link href="<%=basePath %>res/easyui/themes/<%=themes %>/easyui.css" rel="stylesheet" type="text/css" />
<link href="<%=basePath %>res/easyui/themes/icon.css" rel="stylesheet" type="text/css" />
<script src="<%=basePath %>res/easyui/locale/easyui-lang-zh_CN.js" type="text/javascript"></script>
<script type="text/javascript">

dojo.require("esri.map");
dojo.require("esri.toolbars.draw");
dojo.require("esri.dijit.InfoWindow");

var mapobj;

var drawTool;

var mapobj;

var drawEdit = false;

var drawType = null;

var graphic;

var x;

var y;

var isDrawEdit = false;
/**
* 
地图相关
*/
$(function(){
	 var harbinExtent = new esri.geometry.Extent({"xmin":14069874.0762,"ymin":5723732.9038,"xmax":14131654.4081,"ymax":5757136.6164,"spatialReference":{"wkid":3785}});

		mapobj = new esri.Map("HuiStation",
				{
					logo          :false,
					slider        :true, 
					sliderStyle   :"large"
					});
		
		mapobj.setExtent(harbinExtent);
		mapobj.infoWindow.resize(200, 75);
		var BaseTileMapServiceLayer = new esri.layers.ArcGISTiledMapServiceLayer("http://111.40.214.181/arcgis/rest/services/comm/China/MapServer");
		mapobj.addLayer(BaseTileMapServiceLayer,0); 
})
/**
 * 地图画图函数
 */
 function draw(type){
		if(isDrawEdit==true){
			$.messager.alert('错误信息','有未保存的图形信息，请先保存!','error');
			return;
		}
		drawTool = new esri.toolbars.Draw(mapobj);
		mapobj.hideZoomSlider(); // 隐藏缩放按钮
		
		// 设置光标样式
		mapobj.setMapCursor("url(resource/image/cursors/edit.cur),auto");
		
		if(type=="point"){ // 标点
			drawTool.activate(esri.toolbars.Draw.POINT);
			drawType = esri.toolbars.Draw.POINT;
		}else if(type=="polygon"){ // 画多边形
			drawTool.activate(esri.toolbars.Draw.POLYGON);
			drawType = esri.toolbars.Draw.POLYGON;
		}else if(type=="polyline"){ // 折线
			drawTool.activate(esri.toolbars.Draw.POLYLINE);
			drawType = esri.toolbars.Draw.POLYLINE;
		}else if(type=="polyline"){ // 折线
			drawTool.activate(esri.toolbars.Draw.POLYLINE);
			drawType = esri.toolbars.Draw.POLYLINE;
		}else if(type=="extent"){ // 绘制矩形
			drawTool.activate(esri.toolbars.Draw.EXTENT);
			drawType = esri.toolbars.Draw.EXTENT;
		}else if(type=="multipoint"){ // 绘制多点
			drawTool.activate(esri.toolbars.Draw.MULTI_POINT);
			drawType = esri.toolbars.Draw.MULTI_POINT;
		}
		
		dojo.connect(mapobj, "onClick", gClick);
		dojo.connect(drawTool, "onDrawEnd",drawEndHandler); 
		
		isDrawEdit = true;
		
	}

	function gClick(event){
		//alert("gClick x,y = "+event.mapPoint.x+","+event.mapPoint.y);
		x = event.mapPoint.x;
		y = event.mapPoint.y;
	}
/**
* 画图结束响应函数
*/
function drawEndHandler(mapPoint){
	var symbol = "";
	
	if(drawType==esri.toolbars.Draw.POINT){ // 绘制点
		symbol = new esri.symbol.PictureMarkerSymbol('resource/image/marker/bluemarker.png', 16, 16);
	}else if(drawType==esri.toolbars.Draw.POLYGON){ // 绘制多边形
		symbol = new esri.symbol.SimpleFillSymbol(esri.symbol.SimpleFillSymbol.STYLE_SOLID,   
                 new esri.symbol.SimpleLineSymbol(esri.symbol.SimpleLineSymbol.STYLE_DASHDOT,new dojo.Color([255,0,0]),2),  
                 new dojo.Color([255,255,0,0.25]));  
	}else if(drawType==esri.toolbars.Draw.POLYLINE){ // 绘制折线
		symbol = new esri.symbol.SimpleLineSymbol(esri.symbol.SimpleLineSymbol.STYLE_SOLID, new dojo.Color([255,0,0]),1); 
	}else if(drawType==esri.toolbars.Draw.EXTENT){ // 绘制矩形
		symbol = new esri.symbol.SimpleFillSymbol(esri.symbol.SimpleFillSymbol.STYLE_SOLID,   
                 new esri.symbol.SimpleLineSymbol(esri.symbol.SimpleLineSymbol.STYLE_DASHDOT,new dojo.Color([255,0,0]),2),   
                 new dojo.Color([255,255,0,0.25]));  
	}else if(drawType==esri.toolbars.Draw.MULTI_POINT){
		symbol = new esri.symbol.SimpleMarkerSymbol(esri.symbol.SimpleMarkerSymbol.STYLE_DIAMOND,20,   
                new esri.symbol.SimpleLineSymbol(esri.symbol.SimpleLineSymbol.STYLE_SOLID,new dojo.Color([0,0,0]),1),   
                new dojo.Color([255,255,0,0.5]));
	}
    graphic = new esri.Graphic(mapPoint, symbol);//创建点图形
    mapobj.graphics.add(graphic);
    
    mapobj.showZoomSlider();
    mapobj.setMapCursor("url(resource/image/cursors/3dwarro.cur),auto");
    drawTool.deactivate();
    
}
/**
 * 清除图形
 */
function clear(){
	if(isDrawEdit==true){
		isDrawEdit=false
		mapobj.setMapCursor("url(resource/image/cursors/3dwarro.cur),auto");
		mapobj.graphics.clear();
	}else{
		mapobj.setMapCursor("url(resource/image/cursors/3dwarro.cur),auto");
		mapobj.graphics.clear();
	}
}

/**
 * 搜索按钮的响应事件
 */
function search(){ 
	var cityName = $('#roleNameTxt').val();
	$('#salePersionGrid').datagrid('reload',{ 
		cityName:cityName
	}); 
}
function cellStyler1(value,row,index){
		return 'color:red;';		
}
function cellStyler2(value,row,index){
		return 'color:green;';		
}		
function cellStyler3(value,row,index){
		return 'color:blue;';		
}
function search(){ 
	var districtName = $('#districtNameTxt').val();
	$('#districtGrid').datagrid('reload',{ 
		districtName:districtName
	}); 
}
/**
 * 显示站点位置
 */
var x1 ='';
var y1 ='';
function showpositon(){
	if(saveCount==1){
	 x1 = '${X}';
	 y1 = '${Y}';
	}else if(saveCount2!=saveCount){
		x1=x;
		y1=y;
		saveCount2+=1;
	}
	if(x1!=''&&y1!=''){
		mapobj.graphics.clear();
		showpoint(parseFloat(x1),parseFloat(y1),'${name}');
	}else{
		$.messager.confirm('Confirm', '站点没有坐标点,是否创建站点坐标点', function(r){
			if (r){
				mapobj.graphics.clear();
				draw('point');
			}
		});
	}
}
/**
 * 显示地图上的坐标点
 */
function showpoint(px,py,context){
	var point = new esri.geometry.Point(px, py, new esri.SpatialReference({wkid:3785}));  
	//var pt = new esri.geometry.Point(x,y,new SpatialReference({ wkid: "3785" }));
    var pictureMarkerSymbol = new esri.symbol.PictureMarkerSymbol('resource/image/marker/bluemarker.png', 20, 20);
    var g = new esri.Graphic(point, pictureMarkerSymbol);  
    var template = new esri.InfoTemplate();
    template.setTitle("站点名称");
    template.setContent(context);
    g.setInfoTemplate(template);
    mapobj.graphics.add(g);
    mapobj.centerAndZoom(point,15); 
}
/**
 * 保存新的坐标点
 */
 var saveCount = 1;
 var saveCount2 = 1;
 function savePoint(){
	 if(x==undefined||y==undefined){
		 $.messager.alert('警告','请先标点')
	 }else{
	$.messager.confirm('提示','你确定要更改此配套的位置点么？',function(r){
		if(r){
			mapobj.graphics.clear();
			$.ajax({
				url:'<%=basePath %>support.do?method=savePoint&matingId=<%=magtingId%>&X='+x+'&Y='+y,
				dataType:'json',
				returnType:'json',
				success:function(){
					saveCount+=1;
				}
			})
		}
	})}
	
}
$(function(){
	initCommProperty();
	$("#pictureId").val(0);
});


 function showCreateRoleDialog(){
 
 }
 
 
 function initCommProperty(){
	$('#communityproperty').propertygrid({
	    url: '<%=basePath %>support.do?method=selectSupportInfo&matingId=<%=magtingId%>',
	    showGroup: true,
	    scrollbarSize: 0,
	    columns: [[
			{field:'id',title:'类型',width:100,sortable:true},
			{field:'value',title:'内容',width:100,resizable:false}
	    ]],
	});
}
 
 
/* 
* Tooltip script 
* powered by 淅淅代码雨 
* 
* written by Wany 
* 
* 
*/ 
this.tooltip = function(){ 
var xOffset = 10;//定义x的偏移量 
var yOffset = 20;//定义y的偏移量 
$("img").mousemove(function(e){ 
var positionX=e.originalEvent.x-$(this).offset().left||e.originalEvent.layerX-$(this).offset().left||0;//获取当前鼠标相对img的x坐标 
var positionY=e.originalEvent.y-$(this).offset().top||e.originalEvent.layerY-$(this).offset().top||0;//获取当前鼠标相对img的y坐标，（以下用不着，可删除） 
var tipTitle;//定义提示标题 
if(positionX<=$(this).width()/2)//当当前鼠标相对x坐标小于图片宽度的一半时执行 
{ 
$('p').remove();//移除p标签 
$('a').attr('href','https://hao.360.cn/?360safe');//修改a标签的href属性以改变链接 
tipTitle='谷歌'; 
} 
else 
{ 
$('p').remove(); 
$('a').attr('href','https://www.taobao.com/');
//www.baidu.com'); 
tipTitle='百度'; 
} 
$("body").append("<p id='tooltip'>"+tipTitle+"</p>");//在body标签里添加这个p标签，实现提示功能 
$("#tooltip").css("top",(e.pageY - xOffset) + "px")//为id为tooltip的元素设置css样式 
.css("left",(e.pageX + yOffset) + "px") 
.fadeIn("fast");//添加动画效果 
}, 
function(){ 
$("#tooltip").remove();//鼠标移动时移除tooltip，实现提示和鼠标的相对位置不变 
}); 
$("img").mouseout(function(e){ 
$("#tooltip").remove();//鼠标移出img标签时不再显示tooltip，用remove函数将其移除 
}); 
}; 
$(document).ready(function(){ 
$('img').css('border','none'); 
tooltip(); 
}); 
 
function changeBefore(){
	var count = $("#pictureId").val();
	var i = parseInt(count)-1;
	if(count<=0){
		$.messager.alert('警告', '已经是第一张！');
		return;
	}
	$("#pictureId").val(i);
 	var pic = document.getElementById("picture");
 	var a = <%=array %>
 	pic.src=a[i];
 	
 }
 function changeNext(){
	var count = $("#pictureId").val();
	var i = parseInt(count)+1;
	$("#pictureId").val(i);
 	var pic = document.getElementById("picture");
 	var a = <%=array %>
 	if(count>=<%=list.size()-1%>){
 		$.messager.alert('警告', '已经是最后一张！');
 		return;
 	}
 	pic.src=a[i];
 	
 }
 
 
</script>
<style type="text/css">
	.colorblue{color: blue}
	.colorgreen{color:green}
	.colorred{color: red}
	.abody{
		position: absolute;
		left: 0;top: 0;bottom: 0;right: 0;
	}
	a{
		text-decoration:none;
	}
	h1{ 
		font-size:180%; 
		font-weight:normal; 
		color:#555; 
		} 
		a{ 
		text-decoration:none; 
		color:#f30; 
		} 
		p{ 
		clear:both; 
		margin:0; 
		padding:.5em 0; 
		} 
		pre{ 
		display:block; 
		font:100% "Courier New", Courier, monospace; 
		padding:10px; 
		border:1px solid #bae2f0; 
		background:#e3f4f9; 
		margin:.5em 0; 
		overflow:auto; 
		width:800px; 
		} 
		
		
		/* tooltip的样式 */ 
		
		#tooltip{ 
		position:absolute; 
		border:1px solid #333; 
		background:#f7f5d1; 
		padding:2px 5px; 
		color:#333; 
		display:none; 
		} 
</style>
</head>
<body >
	<div class="easyui-panel" collapsible="true" title="属性" style="height:450px">
        <div class="easyui-layout" data-options="fit:true">
		  <div data-options="region:'west',split:true" style="width:50%;padding:10px">
				<table id="communityproperty" style="width:100%"></table>
				<div style="text-align:center;margin-top:10px;">
				<a id="btn" href="#" class="easyui-linkbutton" data-options="iconCls:'icon-save'" style="width:120px;">保存</a></div>
		 </div>
		 <div data-options="region:'center'" title="">
				<div  class="easyui-panel" data-options="" title="照片" style="text-align:center;background:#FFFAF">
				<input id="pictureId" type="hidden" value=""></input>
				<a href="javaScript:changeBefore()">
				<img id ="c2"  src="http://192.168.30.23:8090/g1/M01/00/00/wKgeGFg1TPCAJH_8AAAEgOhp9QA697.jpg" width='27' height="56" />
				</a>
				<img id="picture" src="http://192.168.30.23:8090/g1/M01/00/00/wKgeGFg1QWiALrbRAAQ2mosy21E795.jpg" width="350" height="200"/>
				<a href="javaScript:changeNext()">
				<img id ="c" src="http://192.168.30.23:8090/g1/M00/00/00/wKgeGFg1TNGAa85KAAAEnUiscIE408.jpg" width='27' height="56" />
				</a>
				</div>
				
		</div>
		</div>
	</div>
	
	<div class="easyui-panel"  collapsible="true" title="位置点维护" >
		<div id="HuiStation" style="width:100%;height:300px"></div>
	</div>
	<div style="text-align:center;vertical-align: middle;margin-top:10px;">
		<a href="javascript:draw('point')" class="easyui-linkbutton" style="width:80px;" iconCls="icon-add">标点</a>&nbsp;
		<a href="javascript:showpositon()" class="easyui-linkbutton" style="width:80px;" iconCls="icon-add">定位</a>&nbsp;
		<a href="javascript:savePoint()" class="easyui-linkbutton" style="width:80px;" iconCls="icon-remove">保存坐标</a>&nbsp;
		<a href="javascript:clear()" class="easyui-linkbutton" style="width:80px;" iconCls="icon-remove">清除</a>&nbsp;
	</div>
</body>
</html>
