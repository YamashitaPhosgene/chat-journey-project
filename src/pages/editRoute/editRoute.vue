<template>
  <view class="edit-route-container">
    <!-- 顶部地图 -->
    <view class="map-section">
      <v-chart
        ref="echarts"
        :option="option"
        autoresize
        style="width: 100%; height: 260px"
      />
    </view>
    <!-- 标题 -->
    <view class="route-title">{{ routeTitle }}</view>
    <!-- 行程编辑区 -->
    <view class="days-section">
      <view v-for="(day, dayIdx) in days" :key="dayIdx" class="day-block">
        <view class="day-title">第{{ dayIdx + 1 }}天：</view>
        <view v-for="(point, idx) in day.points" :key="idx" class="point-block">
          <view class="point-name">{{ point.name }}</view>
          <view class="time-row">
            <text>{{ point.time }}</text>
          </view>
          <view class="img-row">
            <image :src="point.image" class="point-img" />
            <button class="img-btn" @click="onChangeImage(dayIdx, idx)">
              更换
            </button>
            <button class="del-btn" @click="onDeletePoint(dayIdx, idx)">
              ×
            </button>
          </view>
        </view>
        <button class="add-btn" @click="onAddPoint(dayIdx)">+ 添加地点</button>
      </view>
    </view>
    <!-- 底部操作栏 -->
    <view class="bottom-bar">
      <button @click="onBack">后退</button>
      <button @click="onForward">前进</button>
      <button @click="onAdjustPlace">调整地点</button>
      <button @click="onAdjustTime">调整时间</button>
      <button class="save-btn" @click="onSave">保存</button>
    </view>
  </view>
</template>

<script>
import * as echarts from "echarts";
import VChart from "vue-echarts";

export default {
  components: { "v-chart": VChart },
  data() {
    return {
      routeTitle: "成都3天2晚旅游攻略-路线",
      days: [
        {
          points: [
            {
              name: "春熙路",
              time: "8:00",
              image: "/static/images/ifs1.jpg",
            },
            {
              name: "景点名称xxx",
              time: "9:00",
              image: "/static/images/ifs1.jpg",
            },
          ],
        },
        {
          points: [
            {
              name: "地点名称xxx",
              time: "时间点",
              image: "/static/images/ifs1.jpg",
            },
          ],
        },
        {
          points: [
            {
              name: "地点名称xxx",
              time: "时间点",
              image: "/static/images/ifs1.jpg",
            },
          ],
        },
      ],
      option: {},
    };
  },
  mounted() {
    this.loadMap();
  },
  methods: {
    loadMap() {
      fetch("https://geo.datav.aliyun.com/areas_v3/bound/510000_full.json")
        .then((res) => res.json())
        .then((geoJson) => {
          echarts.registerMap("sichuan", geoJson);
          this.option = {
            geo: {
              map: "sichuan",
              roam: false,
              label: { show: true },
              itemStyle: {
                areaColor: "#4da3ff",
                borderColor: "#fff",
              },
            },
            series: [
              {
                type: "scatter",
                coordinateSystem: "geo",
                data: [
                  { name: "太古里IFS", value: [104.080989, 30.657689] },
                  { name: "春熙路", value: [104.082321, 30.657378] },
                  // ...可根据days自动生成
                ],
                symbolSize: 16,
                label: {
                  show: true,
                  formatter: "{b}",
                  color: "#222",
                  backgroundColor: "#fff",
                  borderRadius: 4,
                  padding: [2, 6],
                },
                itemStyle: {
                  color: "#d0021b",
                },
              },
              {
                type: "lines",
                coordinateSystem: "geo",
                zlevel: 2,
                effect: {
                  show: true,
                  period: 6,
                  trailLength: 0.1,
                  color: "#fff",
                  symbolSize: 4,
                },
                lineStyle: {
                  color: "#d0021b",
                  width: 2,
                  opacity: 0.6,
                  curveness: 0.2,
                },
                data: [
                  {
                    coords: [
                      [104.080989, 30.657689],
                      [104.082321, 30.657378],
                    ],
                  },
                  // ...可根据days自动生成
                ],
              },
            ],
          };
        });
    },
    onChangeImage(dayIdx, idx) {
      // TODO: 实现图片更换逻辑
      uni.showToast({ title: "更换图片", icon: "none" });
    },
    onDeletePoint(dayIdx, idx) {
      this.days[dayIdx].points.splice(idx, 1);
    },
    onAddPoint(dayIdx) {
      this.days[dayIdx].points.push({
        name: "新地点",
        time: "时间点",
        image: "/static/images/ifs1.jpg",
      });
    },
    onBack() {},
    onForward() {},
    onAdjustPlace() {},
    onAdjustTime() {},
    onSave() {
      uni.showToast({ title: "保存成功", icon: "success" });
    },
  },
};
</script>

<style scoped>
.edit-route-container {
  background: #fff;
  min-height: 100vh;
}
.map-section {
  width: 100%;
  margin-bottom: 10px;
}
.route-title {
  font-size: 18px;
  font-weight: bold;
  text-align: center;
  margin: 10px 0;
}
.days-section {
  padding: 0 16px;
  padding-bottom: 80px;
}
.day-block {
  margin-bottom: 24px;
  border-bottom: 1px solid #eee;
  padding-bottom: 8px;
}
.day-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 8px;
}
.point-block {
  margin-bottom: 12px;
  border-bottom: 1px solid #eee;
  padding-bottom: 8px;
}
.point-name {
  font-size: 15px;
  margin-bottom: 4px;
}
.time-row {
  color: #666;
  margin-bottom: 4px;
}
.img-row {
  display: flex;
  align-items: center;
  margin-bottom: 4px;
}
.point-img {
  width: 60px;
  height: 60px;
  border-radius: 8px;
  margin-right: 8px;
}
.img-btn {
  font-size: 12px;
  margin-right: 8px;
}
.del-btn {
  font-size: 16px;
  color: #d0021b;
  background: none;
  border: none;
}
.add-btn {
  font-size: 13px;
  color: #3d1fff;
  background: none;
  border: none;
  margin-top: 4px;
}
.bottom-bar {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  background: #fff;
  padding: 10px 0;
  box-shadow: 0 -2px 8px #eee;
}
.save-btn {
  background: #3d1fff;
  color: #fff;
  border-radius: 8px;
  padding: 8px 24px;
}
</style>
