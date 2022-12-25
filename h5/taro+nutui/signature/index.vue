<template>
    <div class="pad-20">
        <div class="written">
            <div id="canvas-content" style="width: 100%; height: 100%">
                <canvas
                    class="canvasSign"
                    id="canvasSign"
                    @touchstart="startEventHandler"
                    @touchmove="move"
                    @touchend="end"
                    @touchleave="leaveEventHandler"
                />
            </div>

            <div class="flex-center">
                <button class="button_clear button" @click="() => clearClick()">重签</button>
                <button class="button_submit button" @click="() => saveClick()">确认</button>
            </div>
        </div>
    </div>
</template>
<script lang="ts" setup>
import { reactive, onMounted, ref } from 'vue';
import Taro from '@tarojs/taro';

const emit = defineEmits<{
    (e: 'confirm', ...args: any): void;
}>();

let content: any = null; // canvas 的 绘图工具对象

// 存储canvas的基础属性
const state = reactive({
    width: 0,
    height: 0,
    hasDraw: false, // 用户是否有签名
    // signImage: '',
    flag: true,
    top: 0,
    left: 0
});
const startX = ref(0); // 第一点的 x 坐标
const startY = ref(0); // 第一点的 y 坐标
const endX = ref(0); // 第二点的 x 坐标
const endY = ref(0); // 第二点的 y 坐标

const draw = () => {
    content.beginPath();
    content.moveTo(startX.value < 0 ? 0 : startX.value, startY.value);
    content.lineTo(endX.value < 0 ? 0 : endX.value, endY.value);
    content.lineCap = 'round';
    content.lineWidth = 4;

    content.stroke();
};

// 落笔: 开始签名
const startEventHandler = event => {
    event.preventDefault();

    let target = event.targetTouches?.[0] || {}; // 获取去点击的元素以及位置信息

    startX.value = target.pageX - state.left; // x 偏移量 - x方向的 padding
    startY.value = target.pageY - state.top;

    // 开始签名时,更新签名结束的点位信息
    endX.value = startX.value;
    endY.value = startY.value;
    // 描点连线
    draw();
};

// 签名 ing
const move = e => {
    state.hasDraw = true; // 更新用户的绘画状态,用户判断用户是否签名

    e.preventDefault();
    endX.value = e.targetTouches[0].clientX - state.left;
    endY.value = e.targetTouches[0].clientY - state.top;
    draw();
    // 绘画连线之后需要更新结束的点位信息,每一次都是从上一次的最后一个点开始的
    // 所以 startX startY 直接取 endX endY
    startX.value = endX.value;
    startY.value = endY.value;
};

// 停止签名
const end = e => {
    e.preventDefault();
};

// 起笔: 签名结束
const leaveEventHandler = event => {
    event.preventDefault();
};
// 清空画布
const clearClick = () => {
    content.clearRect(0, 0, state.width, state.height);
    state.hasDraw = false;
    init();
};

const saveClick = () => {
    console.log('state.flag ', state.flag);
    if (state.hasDraw) {
        setTimeout(() => {
            Taro.canvasToTempFilePath({
                canvasId: 'canvasSign',
                fileType: 'png',
                success: res => {
                    emit('confirm', res.tempFilePath);
                },
                fail(error) {
                    console.log(error);
                }
            });
        }, 200);
    } else {
        Taro.showToast({
            icon: 'none',
            title: '未进行签名！不能保存'
        });
    }
};

// 2. 初始化方法
const init = () => {
    // 因为在taro中 Taro.createSelectorQuery().select("**") 获取不到节点,就使用document获取
    const canvasNode = document.getElementById('canvasSign')?.childNodes[0] || {};
    // 在 taro中 canvas被包裹了一层节点 <taro-canvas-core></taro-canvas-core>,所以读取 canvasSign 的childNodes
    const parentNode = document.querySelector('#canvas-content') || {};
    // 根据父节点设置 canvas 的宽高     !!!!
    canvasNode.width = parentNode?.clientWidth;
    canvasNode.height = parentNode?.clientHeight;

    // 用于清空签名
    state.width = canvasNode.width;
    state.height = canvasNode.height;

    content = canvasNode.getContext('2d');

    state.top = 10; // 根据情况设置 上下左右 初始偏移量 !!!
    state.left = 20;
};

// 1.  页面构建完成之后,初始化数据
onMounted(() => {
    state.hasDraw = false;
    // 确保 canvas 节点已渲染完毕
    setTimeout(() => {
        init();
    }, 500);
});
</script>
<style lang="scss">
.pad-20 {
    padding: 20px;
}
.flex-center {
    display: flex;
    align-items: center;
    justify-content: center;
}
.canvasSign {
    background-color: #fff;
    border: 1px solid #ccc;
    width: 100%;
    height: 100%;
}
.secondCanvas {
    width: 100%;
    height: 100%;
    z-index: 0;
}

.written {
    width: 100%;
    height: 400px;
    background-color: #ffffff;

    .button_clear {
        margin-top: 15px;
        background: #e5e5e4;
        color: #666666;
    }
    .button_submit {
        background: #5750fe;
        color: #ffffff;
    }
    .button {
        text-align: center;
        width: 200px;
        height: 70px;
        border-radius: 35px;
        font-weight: 500;
        font-size: 28px;
        z-index: 10000;
    }
}
</style>
