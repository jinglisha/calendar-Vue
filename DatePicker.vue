<template>
    <div class="date-pciker" v-click-outside>
        <div class="picker-input">
            <span class="input-prefix">
                <i class="iconfont icon-date"></i>
            </span>
            <input type="text" :value="chooseDate" />
        </div>
        <div class="picker-panel" v-if="showPanel">
            <div class="picker-arrow" />
            <div class="picker-body">
                <div class="picker-header">
                    <span class="picker-btn iconfont icon-prev-year" @click="onChangeYear('prev')" />
                    <span
                        class="picker-btn iconfont icon-prev-month"
                        @click="onChangeMonth('prev')"
                    />
                    <span class="picker-date">{{showDate.year}}年{{showDate.month + 1}}月</span>
                    <span
                        class="picker-btn iconfont icon-next-month"
                        @click="onChangeMonth('next')"
                    />
                    <span class="picker-btn iconfont icon-next-year" @click="onChangeYear('prev')" />
                </div>
                <div class="picker-content">
                    <div class="picker-weeks">
                        <div
                            v-for="week in [
								'日',
								'一',
								'二',
								'三',
								'四',
								'五',
								'六',
							]"
                            :key="week"
                        >{{ week }}</div>
                    </div>
                    <div class="picker-days">
                        <div
                            v-for="date in showDay"
                            :key="date.getTime()"
                            :class="{
                                'other-month':!isCur(date).month,
                                'is-select':isCur(date).select,
                                'is-today':isCur(date).today  
                            }"
                            @click="onChooseDate(date)"
                        >{{ date.getDate() }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    directives: {
        "click-outside": {
            // 给document绑定事件
            /**
             * el 为使用了特性的元素
             * binding 指令相关的一些信息
             * vnode.context 组件的实例对象
             */
            bind(el, binding, vnode) {
                const vm = vnode.context;
                document.onclick = function (e) {
                    // 判断当前元素是否被div包含，触发元素是否是使用特性的元素的子孙，是，显示pannel面板，不是，隐藏面板
                    const dom = e.target; // 触发元素
                    const isElSon = el.contains(dom); // 触发元素是否在父元素内

                    // 执行changePanel
                    //  如何在指令当中拿到methods里的函数？ 通过vnode.context来调用
                    if (isElSon && !vm.showPanel) {
                        vm.changePanel(true); // 展示panel面板
                    } else if (!isElSon && vm.showPanel) {
                        vm.changePanel(false); // 隐藏panel面板
                    }
                };
            },
        },
    },
    model: {
        prop: 'date',
        event:'choose-date'
    },
    // 注册特性，Date构造函数，使用组件时把Date传递过去
    props: {
        date: {
            type: Date,
            default: () => new Date(),
        },
    },
    computed: {
        chooseDate() {
            // 通过date拿到年月日
            const { year, month, day } = this.getYearMonthDay(this.date);

            return `${year}-${month + 1}-${day}`;
        },

        // 面板的展示  循环数组（每一天的日期）计算属性
        showDay() {
            // 首先拿到显示月份的第一天，showDate
            const { year, month } = this.showDate;

            // 本月第一天是星期几
            const firstDay = new Date(year, month, 1);
            const week = firstDay.getDay();

            // 起始的日期
            const startDay = firstDay - week * 24 * 60 * 60 * 1000;
            var arr = [];
            for (let i = 0; i < 42; i++) {
                // 往数组里添东西，添的是日期对象，如何拿到日期？
                arr.push(new Date(startDay + i * 24 * 60 * 60 * 1000));
            }
            return arr;
        },
    },

    // 一开始日历面板不存在，定义一个data控制面板的显示与隐藏
    data() {
        return {
            showPanel: false,
            // 显示的日期，一开始是根据传递过来的prop特性决定的。 写一个方法
            showDate: {
                year: 0,
                month: 0,
                day: 0,
            },
        };
    },

    // 生命周期函数
    // 此时显示0年0月，是因为没有执行函数getShowDate
    created() {
        this.getShowDate(this.date);
    },

    // 判断点击事件是否发生在整体的父元素内，不在这个div内就隐藏面板，监听点击事件，如何知道我没点到父元素上？监听整个文档的点击事件，判断触发的元素是不是我们点击的元素，如果是，显示panel面板，如果不是，隐藏面板。
    // 给document添加点击事件，写一个指令 v-click-outside
    methods: {
        changePanel(flag) {
            this.showPanel = flag;
        },
        getShowDate(date) {
            const { year, month, day } = this.getYearMonthDay(date);

            this.showDate = {
                year,
                month,
                day,
            };
        },
        onChooseDate(date) {
            // 父组件传递过来的属性不能在子组件里更改，改变父组件
            this.$emit("choose-date", date);
            this.changePanel(false);
            this.getShowDate(date);
        },
        onChangeMonth(type) {
            const { year, month, day } = this.showDate;
            const moveMonth = type === "prev" ? -1 : 1;
            const showDate = new Date(year, month, day);
            showDate.setMonth(month + moveMonth);
            const {
                year: showYear,
                month: showMonth,
                day: showDay,
            } = this.getYearMonthDay(showDate);

            this.showDate.year = showYear;
            this.showDate.month = showMonth;
            this.showDate.day = showDay;
        },
        onChangeYear(type) {
            const moveYear = type === "prev" ? -1 : 1;
            this.showDate.year = moveYear;
        },
        // 获取年月日
        getYearMonthDay(date) {
            const year = date.getFullYear();
            const month = date.getMonth();
            const day = date.getDate();

            return {
                year,
                month,
                day,
            };
        },
        isCur(date) {
            const chooseDate = new Date(this.chooseDate);
            const { year: showYear, month: showMonth } = this.showDate;
            const {
                year: chooseYear,
                month: chooseMonth,
                day: chooseDay,
            } = this.getYearMonthDay(chooseDate);
            const {
                year: curYear,
                month: curMonth,
                day: curDay,
            } = this.getYearMonthDay(new Date());
            const { year, month, day } = this.getYearMonthDay(date);

            //isMonth = year === showYear && month === showMonth
            return {
                month: year === showYear && month === showMonth,
                select:
                    year === chooseYear &&
                    month === chooseMonth &&
                    day === chooseDay,
                today: year === curYear && month === curMonth && day === curDay,
            };
        },
    },
};
</script>
<style scoped>
@import "./assets/font.css";

.date-pciker {
    display: inline-block;
}
.picker-input {
    position: relative;
}
.picker-input input {
    height: 40px;
    line-height: 40px;
    padding: 0 30px;
    background-color: #fff;
    border: 1px solid #dcdfe6;
    border-radius: 4px;
    outline: none;
    cursor: pointer;
}
.picker-input .input-prefix {
    position: absolute;
    left: 5px;
    width: 25px;
    height: 100%;
    line-height: 40px;
    text-align: center;
    color: #c0c4cc;
}
.picker-panel {
    position: absolute;
    width: 322px;
    height: 329px;
    margin-top: 5px;
    border: 1px solid #e4e7ed;
    border-radius: 4px;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
    background-color: #fff;
}
.picker-panel .picker-arrow {
    position: absolute;
    top: -12px;
    left: 30px;
    width: 0;
    height: 0;
    border: 6px solid transparent;
    border-bottom-color: #ebeef5;
}
.picker-panel .picker-arrow::after {
    position: absolute;
    left: -6px;
    top: 1px;
    content: "";
    display: block;
    width: 0;
    height: 0;
    border: 6px solid transparent;
    border-bottom-color: #fff;
    border-top-width: 0;
}
.picker-panel .picker-body {
}
.picker-panel .picker-header {
    display: flex;
    align-items: center;
    justify-content: center;
    padding-top: 15px;
    padding-bottom: 10px;
}
.picker-panel .picker-btn {
    margin-right: 5px;
    margin-left: 5px;
    font-size: 12px;
    color: #303133;
    cursor: pointer;
}
.picker-panel .picker-date {
    margin-left: 60px;
    margin-right: 60px;
    font-size: 14px;
    user-select: none;
}
.picker-panel .picker-content {
    padding: 0 10px 10px 10px;
    color: #606266;
    user-select: none;
}
.picker-panel .picker-weeks {
    display: flex;
    justify-content: space-around;
    height: 40px;
    line-height: 40px;
    border-bottom: 1px solid #ebeef5;
}
.picker-panel .picker-days {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
}
.picker-panel .picker-days div {
    width: 30px;
    height: 30px;
    line-height: 30px;
    text-align: center;
    margin: 4px 6px;
    font-size: 12px;
    cursor: pointer;
}
.picker-panel .picker-days div:hover {
    color: #409eff;
}
.picker-panel .picker-days div.is-today {
    color: #409eff;
    font-weight: 700;
}
.picker-panel .picker-days div.is-select {
    border-radius: 50%;
    background-color: #409eff;
    color: #fff;
}
.picker-panel .picker-days div.other-month {
    color: #c0c4cc;
}
</style>
