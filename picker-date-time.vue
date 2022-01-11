<template>
  <view>
    <view
      :class="`${
        isOpacity
          ? 'pos_abs l_0 t_0 w_100_per h_100_per dis_flex opa_0'
          : 'flex_1'
      }`"
      @tap="isOpened = !isOpened"
      >{{ text || '请选择' }}</view
    >

    <!-- 弹窗 -->
    <AtActionSheet :isOpened="isOpened" :on-close="onClose">
      <!-- 操作 -->
      <view class="dis_flex ju_con_spa_bet font_30">
        <view
          class="dis_flex ju_con_cen ali_it_cen active_bg_col_ddd p_16_32"
          @tap="onClose"
          >取消</view
        >
        <view
          class="
            dis_flex
            ju_con_cen
            ali_it_cen
            col_theme
            active_bg_col_ddd
            p_16_32
          "
          @tap="onConfirm"
          >确定</view
        >
      </view>

      <!-- 选择 -->
      <picker-view
        class="w_100_per h_70_vw"
        indicator-style="height: 32px;"
        :value="value"
        @pickStart="picking = true"
        @pickEnd="picking = false"
        @change="onChange"
      >
        <picker-view-column>
          <view
            v-for="(item, index) in selector[0]"
            :key="index"
            class="h_64 dis_flex ju_con_cen ali_it_cen font_28"
            >{{ item }}</view
          >
        </picker-view-column>

        <picker-view-column>
          <view
            v-for="(item, index) in selector[1]"
            :key="index"
            class="h_64 dis_flex ju_con_cen ali_it_cen font_28"
            >{{ item }}</view
          >
        </picker-view-column>

        <!--  -->
      </picker-view>

      <!--  -->
    </AtActionSheet>
  </view>
</template>

<script>
import Taro from '@tarojs/taro';
import { AtActionSheet } from 'taro-ui-vue';
import 'taro-ui-vue/dist/style/components/action-sheet.scss';
import moment from 'moment';

const defaultValue = () => [0, 0];

export default {
  components: {
    AtActionSheet,
  },
  props: {
    // 显示
    text: {
      type: String,
      default: () => '',
    },

    // 是否透明
    isOpacity: {
      type: Boolean,
      default: () => false,
    },

    // 最小日期时间
    minDateTime: {
      type: String,
      eg: 'YYYY/MM/DD HH:mm',
      default: () => '',
    },

    // 最大日期时间
    maxDateTime: {
      type: String,
      eg: 'YYYY/MM/DD HH:mm',
      default: () => '',
    },

    // 指定固定时分
    constHourMinute: {
      type: String,
      eg: 'HH:mm',
      default: () => '',
    },
  },
  data() {
    return {
      isOpened: false,
      value: defaultValue(),
      picking: false,
      selector: [],

      //
    };
  },
  watch: {
    minDateTime() {
      this.init();
      this.value = defaultValue();
    },
    maxDateTime() {
      this.init();
      this.value = defaultValue();
    },
  },
  computed: {},
  created() {
    this.init();
  },
  methods: {
    moment,

    // 确认
    onConfirm() {
      if (this.picking) return;

      const v = this.value;
      let text = `${this.selector[0][v[0]]} ${this.selector[1][v[1]]}`;
      this.$emit('handleConfirm', text);
      this.onClose();
    },

    // 关闭
    onClose() {
      this.isOpened = false;
    },

    // 变化
    onChange(e) {
      this.value = e.detail.value;
      this.init();
    },

    // 获取一天某间隔所有时间
    getTimeSlot(step) {
      //  step = 间隔的分钟
      let date = new Date();
      date.setHours('00'); // 时分秒设置从零点开始
      date.setSeconds('00');
      date.setUTCMinutes('00');

      let arr = [],
        timeArr = [];
      let slotNum = (24 * 60) / step; // 算出多少个间隔
      for (let f = 0; f < slotNum; f++) {
        let time = new Date(
          Number(date.getTime()) + Number(step * 60 * 1000 * f)
        ); // 获取：零点的时间 + 每次递增的时间
        let hour = '',
          sec = '';
        time.getHours() < 10
          ? (hour = '0' + time.getHours())
          : (hour = time.getHours()); // 获取小时
        time.getMinutes() < 10
          ? (sec = '0' + time.getMinutes())
          : (sec = time.getMinutes()); // 获取分钟
        timeArr.push(hour + ':' + sec);
      }
      return timeArr;
    },

    // 获取两日期之间日期列表函数
    getdiffdate(stime, etime) {
      //初始化日期列表，数组
      let diffdate = new Array();
      let i = 0;
      //开始日期小于等于结束日期,并循环
      while (stime <= etime) {
        diffdate[i] = stime;

        //获取开始日期时间戳
        let stime_ts = new Date(stime.replace(/-/g, '/')).getTime();

        //增加一天时间戳后的日期
        let next_date = stime_ts + 24 * 60 * 60 * 1000;

        //拼接年月日，这里的月份会返回（0-11），所以要+1
        const next_date_obj = new Date(next_date);
        let next_dates_y = next_date_obj.getFullYear() + '-';
        let next_dates_m =
          next_date_obj.getMonth() + 1 < 10
            ? '0' + (next_date_obj.getMonth() + 1) + '-'
            : next_date_obj.getMonth() + 1 + '-';
        let next_dates_d =
          next_date_obj.getDate() < 10
            ? '0' + next_date_obj.getDate()
            : next_date_obj.getDate();

        stime = next_dates_y + next_dates_m + next_dates_d;

        //增加数组key
        i++;
      }
      return diffdate;
    },

    // 初始化
    init() {
      let startTime = moment().add('-1', 'year').format('YYYY-MM-DD');// 开始日期
      let endTime = moment().add('1', 'year').format('YYYY-MM-DD'); // 结束日期

      const minDateTime = this.$props.minDateTime
        ? new Date(this.$props.minDateTime.replace(/-/g, '-'))
        : null;
      const maxDateTime = this.$props.maxDateTime
        ? new Date(this.$props.maxDateTime.replace(/-/g, '-'))
        : null;

      if (minDateTime && !maxDateTime) {
        startTime = moment(minDateTime).format('YYYY-MM-DD');
        endTime = moment(minDateTime).add('1', 'year').format('YYYY-MM-DD');

        //
      } else if (!minDateTime && maxDateTime) {
        startTime = moment(maxDateTime).add('-1', 'year').format('YYYY-MM-DD');
        endTime = moment(maxDateTime).format('YYYY-MM-DD');

        //
      } else if (minDateTime && maxDateTime) {
        startTime = moment(minDateTime).format('YYYY-MM-DD');
        endTime = moment(maxDateTime).format('YYYY-MM-DD');
      }

      const dateArr = this.getdiffdate(startTime, endTime);

      this.selector = [
        dateArr,
        this.getTimeSlot(15).filter((item) => {
          if (this.$props.constHourMinute) {
            return item == this.$props.constHourMinute;
          } else {
            return (
              !this.$props.minDateTime ||
              (this.$props.minDateTime &&
                moment(new Date(`${dateArr[this.value[0]]} ${item}`)).diff(
                  minDateTime
                ) >= 0)
            );
          }

          //
        }),
      ];

      //
    },

    //
  },
};
</script>
<style>
.at-action-sheet__footer {
  padding: 0;
}
</style>
