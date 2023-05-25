<template>
    <div ref="list" class="container" @scroll="handleScroll()">
        <div class="phantom" :style="{ height: listHeight + 'px' }"></div>
        <div class="list" :style="{ transform: getTransform }">
            <div
                v-for="item in visibleData"
                :key="item.id"
                class="list-item"
                :style="{ height: itemSize + 'px', lineHeight: itemSize + 'px' }"
            >
                <!-- 列表行 -->
                <slot :item="item" name="row" />
            </div>
        </div>
    </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, computed } from 'vue';

interface IListItem {
    id: number;
    value: string;
}

const props = defineProps<{
    listData: IListItem[]; // 数据项
    itemSize: number; // 列表项的高度
}>();

const screenHeight = ref(0); // 可视区域高度
const startOffset = ref(0); // 偏移量
const start = ref(0); // 开始索引
const end = ref(0); // 结束索引

const listHeight = computed(() => props.listData.length * props.itemSize); // 列表总高度
const visibleCount = computed(() => Math.ceil(screenHeight.value / props.itemSize) + 1); // 可显示的列表数目  Math.ceil向上取整
const visibleData = computed(() => props.listData.slice(start.value, Math.min(end.value, props.listData.length))); // 获取渲染区数据 兼容数据不足一屏的情况
const getTransform = computed(() => `translate3d(0,${startOffset.value}px,0)`); // 偏移量对应的style

// 监听scroll，获取滚动位置scrollTop
const list = ref();
const handleScroll = () => {
    const scrollTop = list.value.scrollTop;
    start.value = Math.floor(scrollTop / props.itemSize);
    end.value = start.value + visibleCount.value;
    startOffset.value = scrollTop - (scrollTop % props.itemSize);
    console.log('scrollTop', scrollTop);
    console.log('startOffset', startOffset.value);
};

onMounted(() => {
    screenHeight.value = list.value.clientHeight;
    start.value = 0;
    end.value = start.value + visibleCount.value;
});
</script>

<style scoped lang="scss">
.container {
    height: 100%;
    overflow: auto;
    position: relative;

    .phantom {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        z-index: -1;
    }

}
</style>
