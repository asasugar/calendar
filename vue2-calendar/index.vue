<!--
 * @Description: Vue日历组件
 * @Author: Xiongjie.Xue(xiongjie.xue@luckincoffee.com)
 * @Date: 2021-12-07 17:40:33
 * @LastEditors: Xiongjie.Xue(xiongjie.xue@luckincoffee.com)
 * @LastEditTime: 2021-12-09 16:32:58
-->
<template>
  <div class="component-common-calendar">
    <div class="title">
      <div class="title-name">
        {{ selectDay.year }}.{{ selectDay.month > 9 ? selectDay.month : "0" + selectDay.month }}
        <div class="icon" @click="lastMonth">👆</div>
        <div class="icon" @click="nextMonth">👇</div>
      </div>
      <div class="title-open" @click="openChange">
        <div>{{ open ? "收起" : "展开" }}</div>
      </div>
    </div>
    <!-- 日历头部 -->
    <div class="calendar-week">
      <div>日</div>
      <div>一</div>
      <div>二</div>
      <div>三</div>
      <div>四</div>
      <div>五</div>
      <div>六</div>
    </div>
    <!-- 日历主体 -->
    <div class="calendar-main" :style="{height : `${dateList.length/7*72}px`}">
      <div v-for="(item,index) in dateList" :key="index" class="day">
        <div
          :class="['bg',(item.year === selectDay.year && item.month === selectDay.month) ? (item.day === selectDay.day ? 'select' : '') : 'other-month']"
          @click="selectChange(item)"
        >{{ item.day }}</div>
        <div v-if="item.spot && !isShowSlotSpot" class="spot" />
        <slot name="spot" />
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    spot: {
      type: Array,
      default: () => [],
    },
    isShowSlotSpot: {
      type: Boolean,
      default: false,
    },
    defaultTime: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      open: true,
      dateList: [], // 日历主体渲染数组
      selectDay: {}, // 选中时间
    };
  },
  created() {
    const now = this.defaultTime ? new Date(this.defaultTime) : new Date();
    const selectDay = {
      year: now.getFullYear(),
      month: now.getMonth() + 1,
      day: now.getDate(),
      dateString: this.formatTime(now, "Y-M-D"),
    };
    this.setMonth(selectDay.year, selectDay.month, selectDay.day);
  },
  methods: {
    formatNumber(n) {
      n = n.toString();
      return n[1] ? n : `0${n}`;
    },
    getDate(time, format) {
      const formateArr = ["Y", "M", "D", "h", "m", "s"];
      const returnArr = [];
      const date = new Date(time);
      returnArr.push(date.getFullYear());
      returnArr.push(this.formatNumber(date.getMonth() + 1));
      returnArr.push(this.formatNumber(date.getDate()));
      returnArr.push(this.formatNumber(date.getHours()));
      returnArr.push(this.formatNumber(date.getMinutes()));
      returnArr.push(this.formatNumber(date.getSeconds()));
      returnArr.forEach((v, i) => {
        format = format.replace(formateArr[i], v);
      });
      return format;
    },
    getDateDiff(time) {
      let r = "";
      const ft = new Date(time);
      const nt = new Date();
      const nd = new Date(nt);
      nd.setHours(23);
      nd.setMinutes(59);
      nd.setSeconds(59);
      nd.setMilliseconds(999);
      const d = parseInt((nd - ft) / 86400000);
      switch (true) {
        case d === 0: {
          const t = parseInt(nt / 1000) - parseInt(ft / 1000);
          switch (true) {
            case t < 60:
              r = "刚刚";
              break;
            case t < 3600:
              r = `${parseInt(t / 60)}分钟前`;
              break;
            default:
              r = `${parseInt(t / 3600)}小时前`;
          }
          break;
        }
        case d === 1: {
          r = "昨天";
          break;
        }
        case d === 2: {
          r = "前天";
          break;
        }
        case d > 2 && d < 30: {
          r = `${d}天前`;
          break;
        }
        default:
          r = getDate(time, "Y-M-D");
      }
      return r;
    },
    /**
     * 时间戳转化为年 月 日 时 分 秒
     * time: 需要被格式化的时间，可以被new Date()解析即可
     * format：格式化之后返回的格式，年月日时分秒分别为Y, M, D, h, m, s，这个参数不填的话则显示多久前
     */
    formatTime(time, format) {
      if (!format) {
        return this.getDateDiff(time);
      }
      return this.getDate(time, format);
    },
    // 上月切换按钮点击
    lastMonth() {
      const lastMonth = new Date(this.selectDay.year, this.selectDay.month - 2);
      const year = lastMonth.getFullYear();
      const month = lastMonth.getMonth() + 1;
      this.setMonth(year, month);
    },
    // 下月切换按钮点击
    nextMonth() {
      const nextMonth = new Date(this.selectDay.year, this.selectDay.month);
      const year = nextMonth.getFullYear();
      const month = nextMonth.getMonth() + 1;
      this.setMonth(year, month);
    },
    // 设置月份
    setMonth(setYear, setMonth, setDay) {
      if (
        this.selectDay.year !== setYear ||
        this.selectDay.month !== setMonth
      ) {
        const day = Math.min(
          new Date(setYear, setMonth, 0).getDate(),
          this.selectDay.day
        );
        const time = new Date(setYear, setMonth - 1, setDay || day);

        this.selectDay = {
          year: setYear,
          month: setMonth,
          day: setDay || day,
          dateString: this.formatTime(time, "Y-M-D"),
        };
        if (!setDay) {
          this.open = true;
        }

        this.dateInit(setYear, setMonth);
        this.setSpot();
        this.$emit("onSelect", this.selectDay);
      }
    },
    // 展开收起
    openChange() {
      this.open = !this.open;
      this.$emit("onToggleOpen", this.open);
      this.dateInit();
      this.setSpot();
    },
    // 设置日历底下是否展示小圆点
    setSpot() {
      const timeArr = this.spot.map((item) => {
        return this.formatTime(item, "Y-M-D");
      });
      this.dateList = this.dateList.map((item) => {
        if (timeArr.indexOf(item.dateString) !== -1) {
          item.spot = true;
        } else {
          item.spot = false;
        }
        return item;
      });
    },
    // 日历主体的渲染方法
    dateInit(setYear = this.selectDay.year, setMonth = this.selectDay.month) {
      const dateList = []; // 需要遍历的日历数组数据
      const now = new Date(setYear, setMonth - 1); // 当前月份的1号
      const startWeek = now.getDay(); // 目标月1号对应的星期
      const dayNum = new Date(setYear, setMonth, 0).getDate(); // 当前月有多少天
      const forNum = Math.ceil((startWeek + dayNum) / 7) * 7; // 当前月跨越的周数
      if (this.open) {
        // 展开状态，需要渲染完整的月份
        for (let i = 0; i < forNum; i++) {
          const now2 = new Date(now);
          now2.setDate(i - startWeek + 1);
          let obj = {};
          obj = {
            day: now2.getDate(),
            month: now2.getMonth() + 1,
            year: now2.getFullYear(),
            dateString: this.formatTime(now2, "Y-M-D"),
          };
          dateList[i] = obj;
        }
      } else {
        // 非展开状态，只需要渲染当前周
        for (let i = 0; i < 7; i++) {
          const now2 = new Date(now);
          // 当前周的7天
          now2.setDate(
            Math.ceil((this.selectDay.day + startWeek) / 7) * 7 -
              6 -
              startWeek +
              i
          );
          let obj = {};
          obj = {
            day: now2.getDate(),
            month: now2.getMonth() + 1,
            year: now2.getFullYear(),
            dateString: this.formatTime(now2, "Y-M-D"),
          };
          dateList[i] = obj;
        }
      }
      this.dateList = dateList;
    },
    // 一天被点击时
    selectChange(item) {
      const { year, month, day, dateString } = item;
      const selectDay = {
        year,
        month,
        day,
        dateString,
      };
      if (this.selectDay.year !== year || this.selectDay.month !== month) {
        this.setMonth(year, month, day);
      } else if (this.selectDay.day !== day) {
        this.selectDay = selectDay;
        this.$emit("onSelect", this.selectDay);
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.component-common-calendar {
  width: 700px;
  padding: 0 25px;
  background-color: #fff;
  .title {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 40px;
    color: #333;
    padding: 30px;
    line-height: 60px;
    .title-name,
    .title-open {
      display: flex;
      align-items: center;
    }
    .year-month {
      margin-right: 20px;
    }
    .icon {
      padding: 0 16px;
      font-size: 32px;
      color: #999;
    }
    .title-open {
      background-color: #f6f6f6;
      color: #999;
      font-size: 22px;
      line-height: 36px;
      border-radius: 18px;
      padding: 0 14px;
    }
  }
  .calendar-week {
    display: flex;
    line-height: 40px;
    font-size: 28px;
    color: #999;
    div {
      width: 100px;
      text-align: center;
    }
  }

  .calendar-main {
    display: flex;
    flex-wrap: wrap;
    padding: 30px 0;
    transition: height 0.3s;
    overflow: hidden;
    .day {
      position: relative;
      width: 100px;
      color: #666;
      text-align: center;
      height: 72px;
      .bg {
        height: 56px;
        line-height: 56px;
        font-size: 28px;
        color: #333;
        font-weight: bold;
      }
      .select {
        width: 56px;
        border-radius: 50%;
        text-align: center;
        color: #fff;
        background: linear-gradient(-60deg, #0fdac5, #1bc7b0);
        box-shadow: 0px 5px 16px 0px #c6f3ed;
        margin: 0 auto;
      }
      .other-month {
        color: #ccc;
      }
      .spot {
        width: 8px;
        height: 8px;
        background-color: #1dcdb8;
        border-radius: 50%;
        margin: 6px auto 0;
      }
    }
  }
}
</style>
