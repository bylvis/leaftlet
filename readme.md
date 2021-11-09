# leaflet 认识

##   绘制地图
    1.创建一个地图的实例 设置中心点 层级
    var mymap = L.map('map').setView([51.505, -0.09], 13);
    2.引入地图api
    L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
            attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
            maxZoom: 18,
            id: 'mapbox/streets-v11',
            tileSize: 512,
            zoomOffset: -1,
            accessToken: 'pk.eyJ1IjoiYmFpeXVsb25nIiwiYSI6ImNrdnJzczBnd2F4NmYybnM3cDQxNmVmYjUifQ.8_3b5GgzWBtkEVBvBclzQQ'
        }).addTo(mymap);

## 地图上的操作 通过addTo(map) 添加到地图上
    1.L.marker() 创建标记 
    2.L.circle() 绘制圆 参数有中心点以及配置项
    3.L.polygon() 绘制多边形  
    4.以上三种标记方法的实例 都有 bindPopup('内容') 方法 是弹出一个标签
    5.map.locate({ setView: true, maxZoom: 20 }); 定位到当前位置

## 事件对象
    leaflet的事件对象 有很多属性 
    e.latlng 表示当前事件对象的坐标

## 自定义图标 L.icon L.Icon 大写的用New来创建 小写的就是返回一个大写的对应配置的函数
    var greenIcon = L.icon({
            iconUrl: './img/leaf-green.png',
            shadowUrl: './img/leaf-shadow.png',
            iconSize: [38, 95], // size of the icon
            shadowSize: [50, 64], // size of the shadow
            iconAnchor: [22, 94], // point of the icon which will correspond to marker's location 标记点
            shadowAnchor: [4, 62],  // the same for the shadow
            popupAnchor: [-3, -76] // point from which the popup should open relative to the iconAnchor
        });

## 关于GeoJSON数据的引入
    两种方法 一种直接引入数据 一种先创建图层再引入数据
    1.L.geoJSON(需要的geoJSON数据).add(map)

    2.var myLayer = L.geoJSON().addTo(map);
      myLayer.addData(myLines);
    
    可以给自己的图层添加样式
    1.先写一个样式对象
    var myStyle = {
        "color": "#ff7800",
        "weight": 5,
        "opacity": 0.65
    };

    2.将样式对象赋给自己的图层
    L.geoJSON(myLines, {
        style: myStyle
    }).addTo(map);

    3.可以通过函数调用geoJSON属性从而给图层改变样式
    var states={
            "type":"Feature",
            "properties":{
                "party":"SUSE"
            },
            "geometry":{
                "type":'Polygon',
                "coordinates":[[
                    [104.660793,28.806536],
                    [104.671141,28.799129],
                    [104.677573,28.802928],
                    [104.673513,28.813023]
                    
                ]]
            }
        }

    L.geoJSON(states,{
        style:function(feature){
            switch(feature.properties.party){
                case 'SUSE':return {color:"#ff0000"}
            }
        }
    }).addTo(map)

    4.绘制标记点 geoJSON是点类型的 可以通过pointToLayer来绘制
    L.geoJSON(states, {
            onEachFeature: onEachFeature,
            pointToLayer: function (feature, latlng) {
                return L.circleMarker(latlng, someStyle)
            }
        }).addTo(map)