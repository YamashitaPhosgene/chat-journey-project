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
        <view class="point-names-row">
          <text
            v-for="(point, idx) in day.points"
            :key="idx"
            class="point-name-row-item"
          >
            {{ point.name
            }}<span v-if="idx !== day.points.length - 1"> / </span>
          </text>
        </view>
        <view class="divider"></view>
        <!-- 横向时间轴（节点在一条横线上，节点下方展示内容） -->
        <scroll-view scroll-x class="timeline-horizontal">
          <view class="timeline-h-row">
            <view
              v-for="(point, idx) in day.points"
              :key="idx"
              class="timeline-h-node"
            >
              <view class="timeline-h-time">{{ point.time }}</view>
              <view class="timeline-h-dot"></view>
              <view
                class="timeline-h-line"
                v-if="idx !== day.points.length - 1"
              ></view>
              <image :src="point.image" class="timeline-h-img" />
              <button
                class="timeline-h-btn"
                @click="onChangeImage(dayIdx, idx)"
              >
                更换 ✎
              </button>
              <button
                class="timeline-h-del"
                @click="onDeletePoint(dayIdx, idx)"
              >
                ×
              </button>
            </view>
          </view>
        </scroll-view>
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
.point-names-row {
  display: flex;
  flex-wrap: wrap;
  font-size: 14px;
  color: #444;
  margin-bottom: 4px;
}
.point-name-row-item {
  margin-right: 2px;
}
.divider {
  height: 2px;
  background: #673ab7;
  opacity: 0.2;
  margin-bottom: 8px;
}
.timeline-horizontal {
  overflow-x: auto;
  padding: 24px 0 16px 0;
  white-space: nowrap;
}
.timeline-h-row {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
}
.timeline-h-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-width: 100px;
  margin-right: 40px;
  position: relative;
}
.timeline-h-time {
  font-size: 15px;
  font-weight: bold;
  color: #222;
  margin-bottom: 4px;
}
.timeline-h-dot {
  width: 14px;
  height: 14px;
  background: #673ab7;
  border-radius: 50%;
  margin-bottom: 8px;
  z-index: 2;
}
.timeline-h-line {
  position: absolute;
  top: 20px;
  left: 100%;
  width: 40px;
  height: 2px;
  background: #b39ddb;
  z-index: 1;
}
.timeline-h-img {
  width: 64px;
  height: 64px;
  border-radius: 8px;
  object-fit: cover;
  margin-bottom: 4px;
}
.timeline-h-btn {
  font-size: 12px;
  color: #222;
  background: #f5f5f5;
  border: none;
  border-radius: 6px;
  padding: 2px 10px;
  margin-bottom: 2px;
}
.timeline-h-del {
  position: absolute;
  top: 0;
  right: 0;
  background: #fff;
  color: #d0021b;
  border: 1px solid #eee;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  font-size: 14px;
  line-height: 18px;
  padding: 0;
  z-index: 2;
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
