<script setup lang="ts">
import { onMounted, onUnmounted, reactive, watch, computed } from "vue";
/**
 * 本组件实现了整数，小数，人民币金额的动态变化
 * 组件直接使用，根据传入开始和结束值，实现从start到end的缓动变化，缓动函数默认开启，可根据传入的布尔值是否开启
 * 缓动效果，开始很快最后渐渐慢下来。动态时长可根据duration传入的毫秒值决定
 */
// 定义state的接口类型
interface CountUpState {
  localStart: number;
  displayValue: string;
  printVal: number | null;
  paused: boolean;
  localDuration: number;
  startTime: number | null;
  timestamp: number | null;
  remaining: number | null;
  rAF: number | null;
  lastTime: number;
}
const props = defineProps({
    // 初始值
    start: {
        type: Number,
        required: false,
        default: 0,
    },
    // 结果值
    end: {
        type: [Number, String],
        required: false,
        default: 0,
    },
    // 动态时长
    duration: {
        type: Number,
        required: false,
        default: 10000,
    },
    // 自动播放，不传值默认开启
    autoPlay: {
        type: Boolean,
        required: false,
        default: true,
    },
    // 千分位分隔符
    separator: {
        type: String,
        required: false,
    },
    // 前缀
    prefix: {
        type: String,
        required: false,
        default: "",
    },
    // 后缀
    suffix: {
        type: String,
        required: false,
        default: "",
    },
    // 是否使用缓动函数
    useEasing: {
        type: Boolean,
        required: false,
        default: true,
    },
    // 缓动函数
    easingFn: {
        type: Function,
        default(t: number, b: number, c: number, d: number) {
            return (c * (-Math.pow(2, (-10 * t) / d) + 1) * 1024) / 1023 + b;
        },
    },
});
// 判断是不是一个数字
const isNumber = (val: any) => {
    return !isNaN(parseFloat(val));
};

// 格式化数据，返回想要展示的数据格式
const formatNumber = (num: number) => {
    // 保留几位小数点
    let decimals = 0
    // 判断有没有小数点
    if (String(props.end).indexOf(".") !== -1) {
        decimals = String(props.end).split(".")[1].length
    }
    // 保留小数的值
    let val = Number(num).toFixed(decimals);
    val += "";
    const x = val.split(".");
    let x1 = x[0];
    const x2 = x.length > 1 ? "." + x[1] : "";
    // 千分位正则
    const rgx = /(\d+)(\d{3})/
    if (props.separator && !isNumber(props.separator)) {
        while (rgx.test(x1)) {
            x1 = x1.replace(rgx, '$1' + props.separator + '$2')
        }
        // 重新拼接带有千分位
        return props.prefix + x1 + x2 + props.suffix;
    } else {
        // 拼接普通数字
        return props.prefix + x1 + x2 + props.suffix
    }
};

// 基础数据
const state = reactive<CountUpState>({
    // 初始值为起始值
    localStart: props.start,
    // 显示值为格式化后的起始值
    displayValue: formatNumber(props.start),
    // 实际打印值初始为空
    printVal: null,
    // 是否暂停初始为 false
    paused: false,
    // 本地持续时间初始为总时间
    localDuration: props.duration,
    // 开始时间初始为 null
    startTime: null,
    // 时间戳初始为 null
    timestamp: null,
    // 剩余时间初始为 null
    remaining: null,
    // requestAnimationFrame ID 初始为 null
    rAF: null,
    // 上一次时间初始为 0
    lastTime: 0
});

// 定义一个计算属性，当开始数字大于结束数字时返回true
const stopCount = computed(() => {
    return Number(props.start) > Number(props.end);
});
// 定义开始计时函数
const startCount = () => {
    // 将 localStart 设置为起始值
    state.localStart = props.start;
    // 重置 startTime 为 null
    state.startTime = null;
    // 将 localDuration 设置为总时间
    state.localDuration = props.duration;
    // 将 paused 状态重置为 false
    state.paused = false;
    // 请求下一个动画帧
    state.rAF = requestAnimationFrame(count);
};
// 监听开始
watch(
    () => props.start,
    () => {
        if (props.autoPlay) {
            startCount();
        }
    }
);
// 监听结束
watch(
    () => props.end,
    () => {
        if (props.autoPlay) {
            startCount();
        }
    }
);

// 定义暂停函数
const pause = () => {
    // 取消动画帧请求
    cancelAnimationFrame(state.rAF as number);
};
// 定义恢复函数
const resume = () => {
    // 重置 startTime 为 null
    state.startTime = null;
    // 将 localDuration 设置为剩余时间
    state.localDuration = +state.remaining!;
    // 将 localStart 设置为当前的显示值
    state.localStart = +state.printVal!;
    // 请求下一个动画帧
    requestAnimationFrame(count);
};

// 定义暂停/恢复函数
const pauseResume = () => {
    // 如果之前已经暂停，则执行 resume 恢复动画，并将 paused 状态标记为 false
    if (state.paused) {
        resume();
        state.paused = false;
    } else {
        // 否则执行 pause 暂停动画，并将 paused 状态标记为 true
        pause();
        state.paused = true;
    }
};

// 定义重置函数
const reset = () => {
    // 重置 startTime 为 null
    state.startTime = null;
    // 取消动画帧请求
    cancelAnimationFrame(state.rAF as number);
    // 将 displayValue 重置为初始值
    state.displayValue = formatNumber(props.start);
};
// 核心函数，对数字进行转动
const count = (timestamp: any) => {
    if (!state.startTime) state.startTime = timestamp;
    state.timestamp = timestamp;
    const progress = timestamp - state.startTime!;
    state.remaining = state.localDuration - progress;
    // 是否使用速度变化曲线
    if (props.useEasing) {
        if (stopCount.value) {
            state.printVal =
                state.localStart -
                props.easingFn(
                    progress,
                    0,
                    state.localStart - Number(props.end),
                    state.localDuration
                );
        } else {
            state.printVal = props.easingFn(
                progress,
                state.localStart,
                Number(props.end) - state.localStart,
                state.localDuration
            );
        }
    } else {
        if (stopCount.value) {
            state.printVal =
                state.localStart -
                (state.localStart - Number(props.end)) * (progress / state.localDuration);
        } else {
            state.printVal =
                state.localStart +
                (Number(props.end) - state.localStart) * (progress / state.localDuration);
        }
    }
    if (stopCount.value) {
        state.printVal = state.printVal! < Number(props.end) ? Number(props.end) : state.printVal;
    } else {
        state.printVal = state.printVal! > Number(props.end) ? Number(props.end) : state.printVal;
    }

    state.displayValue = formatNumber(state.printVal as number);
    if (progress < state.localDuration) {
        state.rAF = requestAnimationFrame(count);
    }
};

// 执行动画定时器
const requestAnimationFrame = function (callback:Function) {
    // 获取当前的时间戳
    const currTime = new Date().getTime()
    // 计算需要延迟多少时间，以使回调函数的执行时间接近每秒60帧
    const timeToCall = Math.max(0, 16 - (currTime - state.lastTime))
    // 使用 setTimeout 函数来模拟 requestAnimationFrame 函数，调用回调函数
    const id = window.setTimeout(() => {
        callback(currTime + timeToCall)
    }, timeToCall)
    // 更新上一次的时间戳，用于计算下一次延迟时间
    state.lastTime = currTime + timeToCall
    return id
}

// 定义取消动画帧请求函数
const cancelAnimationFrame = function (id:number) {
    // 取消由 setTimeout 设置的计时器
    window.clearTimeout(id)
};
// dom挂在完成后执行一些操作
onMounted(() => {
    if (props.autoPlay) {
        startCount();
    }
});
// 组件销毁时取消动画
onUnmounted(() => {
    cancelAnimationFrame(state.rAF as number);
});
</script>
<template>
    <span>{{ state.displayValue }}</span>
</template>



<style scoped></style>