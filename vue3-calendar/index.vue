<!--
 * @Description: Vue3日历组件
 * @Author: Xiongjie.Xue(xiongjie.xue@luckincoffee.com)
 * @Date: 2021-12-07 17:40:33
 * @LastEditors: Xiongjie.Xue(xiongjie.xue@luckincoffee.com)
 * @LastEditTime: 2021-12-09 18:23:35
-->
<template>
  <div class="component-common-calendar">
    <div class="title">
      <div class="title-name">
        {{ state.selectDay.year }}.{{ state.selectDay.month > 9 ? state.selectDay.month : "0" + state.selectDay.month }}
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
    <div class="calendar-main" :style="{height : `${state.dateList.length/7*72}px`}">
      <div v-for="(item,index) in state.dateList" :key="index" class="day">
        <div
          :class="['bg',(item.year === state.selectDay.year && item.month === state.selectDay.month) ? (item.day === state.selectDay.day ? 'select' : '') : 'other-month']"
          @click="selectChange(item)"
        >{{ item.day }}</div>
        <div v-if="item.spot && !isShowSlotSpot" class="spot" />
        <slot name="spot" v-if="item.spot && isShowSlotSpot" />
      </div>
    </div>
  </div>
</template>
<script>
import { defineComponent, ref, reactive } from "vue";
export default defineComponent({
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
  setup(props, content) {
    const open = ref(true);
    const state = reactive({
      dateList: [], // 日历主体渲染数组
      selectDay: {
        // 选中时间
        year: 0,
        month: 0,
        day: 0,
        dateString: "",
      },
    });

    const formatNumber = (n) => {
      n = n.toString();
      return n[1] ? n : `0${n}`;
    };
    const getDate = (time, format) => {
      const formateArr = ["Y", "M", "D", "h", "m", "s"];
      const returnArr = [];
      const date = new Date(time);
      returnArr.push(date.getFullYear());
      returnArr.push(formatNumber(date.getMonth() + 1));
      returnArr.push(formatNumber(date.getDate()));
      returnArr.push(formatNumber(date.getHours()));
      returnArr.push(formatNumber(date.getMinutes()));
      returnArr.push(formatNumber(date.getSeconds()));
      returnArr.forEach((v, i) => {
        format = format.replace(formateArr[i], v);
      });
      return format;
    };
    const getDateDiff = (time) => {
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
    };
    /**
     * 时间戳转化为年 月 日 时 分 秒
     * time: 需要被格式化的时间，可以被new Date()解析即可
     * format：格式化之后返回的格式，年月日时分秒分别为Y, M, D, h, m, s，这个参数不填的话则显示多久前
     */
    const formatTime = (time, format) => {
      if (!format) {
        return getDateDiff(time);
      }
      return getDate(time, format);
    };
    // 日历主体的渲染方法
    const dateInit = (
      setYear = state.selectDay.year,
      setMonth = state.selectDay.month
    ) => {
      const dateList = []; // 需要遍历的日历数组数据
      const now = new Date(setYear, setMonth - 1); // 当前月份的1号
      const startWeek = now.getDay(); // 目标月1号对应的星期
      const dayNum = new Date(setYear, setMonth, 0).getDate(); // 当前月有多少天
      const forNum = Math.ceil((startWeek + dayNum) / 7) * 7; // 当前月跨越的周数
      if (open.value) {
        // 展开状态，需要渲染完整的月份
        for (let i = 0; i < forNum; i++) {
          const now2 = new Date(now);
          now2.setDate(i - startWeek + 1);
          let obj = {};
          obj = {
            day: now2.getDate(),
            month: now2.getMonth() + 1,
            year: now2.getFullYear(),
            dateString: formatTime(now2, "Y-M-D"),
          };
          dateList[i] = obj;
        }
      } else {
        // 非展开状态，只需要渲染当前周
        for (let i = 0; i < 7; i++) {
          const now2 = new Date(now);
          // 当前周的7天
          now2.setDate(
            Math.ceil((state.selectDay.day + startWeek) / 7) * 7 -
              6 -
              startWeek +
              i
          );
          let obj = {};
          obj = {
            day: now2.getDate(),
            month: now2.getMonth() + 1,
            year: now2.getFullYear(),
            dateString: formatTime(now2, "Y-M-D"),
          };
          dateList[i] = obj;
        }
      }
      state.dateList = dateList;
    };
    // 设置日历底下是否展示小圆点
    const setSpot = () => {
      const timeArr = props.spot.map((item) => {
        return formatTime(item, "Y-M-D");
      });
      state.dateList = state.dateList.map((item) => {
        if (timeArr.indexOf(item.dateString) !== -1) {
          item.spot = true;
        } else {
          item.spot = false;
        }
        return item;
      });
    };
    // 设置月份
    const setMonth = (setYear, setMonth, setDay) => {
      if (
        state.selectDay.year !== setYear ||
        state.selectDay.month !== setMonth
      ) {
        const day = Math.min(
          new Date(setYear, setMonth, 0).getDate(),
          state.selectDay.day
        );
        const time = new Date(setYear, setMonth - 1, setDay || day);
        state.selectDay = {
          year: setYear,
          month: setMonth,
          day: setDay || day,
          dateString: formatTime(time, "Y-M-D"),
        };
        if (!setDay) {
          open.value = true;
        }

        dateInit(setYear, setMonth);
        setSpot();
        content.emit("onSelect", state.selectDay);
      }
    };
    // 上月切换按钮点击
    const lastMonth = () => {
      const lastMonth = new Date(
        state.selectDay.year,
        state.selectDay.month - 2
      );
      const year = lastMonth.getFullYear();
      const month = lastMonth.getMonth() + 1;
      setMonth(year, month);
    };
    // 下月切换按钮点击
    const nextMonth = () => {
      const nextMonth = new Date(state.selectDay.year, state.selectDay.month);
      const year = nextMonth.getFullYear();
      const month = nextMonth.getMonth() + 1;
      setMonth(year, month);
    };
    // 展开收起
    const openChange = () => {
      open.value = !open.value;
      content.emit("onToggleOpen", open.value);
      dateInit();
      setSpot();
    };

    // 一天被点击时
    const selectChange = (item) => {
      const { year, month, day, dateString } = item;

      if (state.selectDay.year !== year || state.selectDay.month !== month) {
        setMonth(year, month, day);
      } else if (state.selectDay.day !== day) {
        state.selectDay = {
          year,
          month,
          day,
          dateString,
        };
        content.emit("onSelect", state.selectDay);
      }
    };

    // 初始化当前日期
    const now = props.defaultTime ? new Date(props.defaultTime) : new Date();
    const selectNowDay = {
      year: now.getFullYear(),
      month: now.getMonth() + 1,
      day: now.getDate(),
      dateString: formatTime(now, "Y-M-D"),
    };
    setMonth(selectNowDay.year, selectNowDay.month, selectNowDay.day);

    return {
      open,
      state,
      lastMonth,
      nextMonth,
      openChange,
      selectChange,
    };
  },
});
</script>
<style lang="less" scoped>
.component-common-calendar {
  background-color: #fff;
  width: 700px;
  padding: 0 25px;
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
        background: linear-gradient(-60deg, #0fdac5, #0028ab);
        box-shadow: 0px 5px 16px 0px #c6f3ed;
        margin: 0 auto;
      }
      .other-month {
        color: #ccc;
      }
      .spot {
        width: 8px;
        height: 8px;
        background-color: #0028ab;
        border-radius: 50%;
        margin: 6px auto 0;
      }
    }
  }
}
</style>
