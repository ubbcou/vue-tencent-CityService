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
                            _this,currentCity_detail = res.detail.detail;
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