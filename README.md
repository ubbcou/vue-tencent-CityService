---
title: 使用VUE对腾讯地图API进行的操作
---

腾讯JavaScript API V2可用于在网站中加入交互性强的街景、地图，能很好地支持PC及手机设备，身材小巧，动画效果顺滑流畅，动感十足，提供地图操作、标注、地点搜索、出行规划、地址解析、街景等接口，功能丰富，并免费开放各种附加工具库。JavaScript API V2是免费服务，任何提供免费访问的网站都可以调用，请参见使用条款。

本文主要应用其服务类——[CityService ](http://lbs.qq.com/javascript_v2/doc/cityservice.html)根据城市名称、经纬度、IP地址（支持自动获取用户IP）、电话区号获取城市信息。

#### API 服务加载

需要在页面的前端（index.html 即可）使用script标签加载API服务，格式如下：

其中，v参数是引用的版本号，目前腾讯地图提供两个版本，分别是v=1，v=2.exp，推荐使用2.exp，可以获得最新最快的支持。Key参数YOUR_KEY是[Key鉴权](http://lbs.qq.com/javascript_v2/guide-base.html#link-two)中申请的key。

如果您要使用 https 服务，可以直接将协议替换成 https 即 url 为：https://map.qq.com/api/js?v=2.exp&Key=YOUR_KEY。

#### API 与 Vue结合

代码中大致把情况列出。

`map.vue`：

```vue
<template>
    <div class="map-positon">
        <p>{{ currentCity }}</p>
        <p>{{ currentCity_detail }}</p>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                citylocation: null,
                lat: 0,
                lng: 0,
                currentCity: "杭州市", // 没有正确得到经纬度时，默认显示杭州吧
                currentCity_detail: ''
            }
        },
        computed: {
        },
        methods: {
            initCityService() {
                var _this = this;
                var city = document.getElementById("city");
                //调用城市服务信息
                this.citylocation = new qq.maps.CityService({
                    complete: function (res) {
                        city.style.display = 'inline';
                        // 在pc测试时，会有经纬度负数的情况.....我没找原因=。=，直接处理了下
                        if (!res.detail.name) {
                            return;
                        } else {
                            _this.currentCity_detail = res.detail.detail;
                            _this.currentCity = res.detail.name;
                        }
                    }
                });
                // setError 设置检索失败后的回调函数。
                this.citylocation.setError(function () {
                    alert("出错了，没有获得正确的经纬度！！！");
                });

            },
            initGeolocation() {
                if (navigator.geolocation) {
                    navigator.geolocation.getCurrentPosition(this.initLatLng, this.initErro);
                }
                // 无法使用getCurrentPosition获取经纬度的情况
                else {
                    alert("无法获取位置");
                }
            },
            initLatLng(posi) {
                if (posi) {
                    this.lat = posi.coords.latitude;
                    this.lng = posi.coords.longitude;
                    // 初始化CityService 类
                    this.initCityService();
                    // 获取地理位置
                    this.geolocation_latlng();
                }
            },
            // 使用html5定位时不能获取经纬度的报错信息
            initErro(error) {
                console.log(error);
                switch (error.code) {
                    case error.PERMISSION_DENIED:
                        alert("用户拒绝对获取地理位置的请求。");
                        break;
                    case error.POSITION_UNAVAILABLE:
                        alert("位置信息是不可用的。")
                        break;
                    case error.TIMEOUT:
                        alert("请求用户地理位置超时。");
                        break;
                    case error.UNKNOWN_ERROR:
                        alert("未知错误。");
                        break;
                }
            },
            geolocation_latlng() {
                if (this.lat != 0 && this.lng != 0) {
                    var latLng = new qq.maps.LatLng(this.lat, this.lng);
                    this.citylocation.searchCityByLatLng(latLng);
                }
            }
        },
        mounted() {
            // 执行navigator.geolocation.getCurrentPosition 初始化经纬度
            this.initGeolocation();
        }
    }

</script>
```

