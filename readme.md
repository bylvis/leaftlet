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
   
## onEachFeature 
    onEachFeature是一个函数，在将每个特性添加到geoJson图层之前会调用
    定义onEachFeature函数 给弹出层绑定geojson数据的特征
    function onEachFeature(feature, layer) {
    // does this feature have a property named popupContent?
    if (feature.properties && feature.properties.popupContent) {
            layer.bindPopup(feature.properties.popupContent);
        }
    }
    加载图层的特征
     L.geoJSON(states, {
            onEachFeature: onEachFeature,
            pointToLayer: function (feature, latlng) {
                return L.circleMarker(latlng, someStyle)
            }
        }).addTo(map)
## 过滤器
    过滤掉用户不想展示的部分 接受的值是布尔类型
     L.geoJSON(someFeatures, {
            filter: function (feature, layer) {
                return feature.properties.show_map
            }
    }).addTo(map)

## 通过监听事件给地图设置hover以及mouseout移除hover的效果
    var layer = e.target
    将当前事件的宿主设置颜色
    layer.setStyle({
                weight:5,
                color:'#666',
                dashArray:'',
                fillOpacity:0.7
            });
    移除高亮
    function resetHighlight(e) {
            重置图层
            myLayer.resetStyle(e.target);
        }
    点击放大 
    function zoomToFeature(e) {
           map.fitBounds(e.target.getBounds());
        }
    所有事件统一到onEachFeature里面 加载开头统一添加事件
    function onEachFeature(feature, layer) {
            layer.on({
            mouseover: highlightFeature,
            mouseout: resetHighlight,
            click: zoomToFeature
        }); 
## 在地图里面添加控件
    创建控件
    1.var info = L.control();
    2.向控制组件里面添加DOM元素 并且赋予类名
    info.onAdd = function (map) {
        this._div = L.DomUtil.create('div', 'info'); // create a div with a class "info"
        this.update();
        return this._div;
    };
    3.给控件添加方法 接收geo的数据 更新控件
    info.update = function (props) {
            this._div.innerHTML = '<h4>US Population Density</h4>' +  (props ?
                '<b>' + props.name + '</b><br />' + props.density + ' people / mi<sup>2</sup>'
                : 'Hover over a state');
        };
    4.把控件添加到map里面
     info.addTo(map);
    5.在每次触发悬停事件的时候需要调用控件方法
## 创建带有图例的控件
    动态添加dom以及动态添加dom属性样式即可
    var legend = L.control({position: 'bottomright'});
    legend.onAdd = function(map){
            var div = L.DomUtil.create('div','info legend'),
            grades = [0, 10, 20, 50, 100, 200, 500, 1000],
            labels = [];
            // 循环创建i标签 动态添加背景颜色
            for (var i = 0; i < grades.length; i++) {
                    div.innerHTML += 
                    '<i style="background:' + getColor(grades[i] + 1) + '"></i> ' 
                    + grades[i] + (grades[i + 1] ? '&ndash;' + grades[i + 1] + '<br>' : '+');
            }
            return div;
        }
    legend.addTo(map);
## 图层控制
    1.互斥的基础层
    
    2.覆盖层