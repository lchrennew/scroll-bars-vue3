<template>
    <div ref="panel"
         :class="{scrolling}"
         class="scroll-panel"
         @wheel.passive="onScroll"
    >
        <div class="content"
             ref="content">
            <slot/>
        </div>
        <div
            v-if="scrollX && overflowX"
            class="scroll-bar x"
            @mouseenter.exact.self="onScrollBarHoverX"
            @click.exact.self.prevent.stop="onScrollBarClickX">
            <div ref="handlerX"
                 class="handler x"
                 @mousedown.left.prevent.stop="onStartHandlerScrollX"
            />
        </div>
        <div
            v-if="scrollY && overflowY"
            class="scroll-bar y"
            @mouseenter.exact.self="onScrollBarHoverY"
            @click.exact.self.prevent.stop="onScrollBarClickY">
            <div ref="handlerY"
                 class="handler y"
                 @mousedown.left.prevent.stop="onStartHandlerScrollY"
            />
        </div>
    </div>
</template>

<script>
import { reactive } from "vue";

const registry = reactive({})

setInterval(() => {
    for (const id in registry) {
        registry[id]()
    }
}, 10)

</script>

<script setup>
import { computed, onBeforeUnmount, onMounted, ref } from "vue";

const props = defineProps({
    scrollX: { type: Boolean, default: false },
    scrollY: { type: Boolean, default: false },
    zIndex: { type: Number, default: 2 }
})

const id = (Math.random() * 100000).toString(36)
const contentClientWidth = ref(0)
const contentScrollWidth = ref(0)
const contentClientHeight = ref(0)
const contentScrollHeight = ref(0)

const overflowX = computed(() => contentScrollWidth.value > contentClientWidth.value)
const overflowY = computed(() => contentScrollHeight.value > contentClientHeight.value)
const scaleX = computed(() => contentScrollWidth.value / contentClientWidth.value)
const scaleY = computed(() => contentScrollHeight.value / contentClientHeight.value)

const panel = ref(null)
const content = ref(null)
const handlerX = ref(null)
const handlerY = ref(null)

onMounted(() => {
    registry[id] = () => {
        const { clientWidth, scrollWidth, clientHeight, scrollHeight } = content.value
        contentClientWidth.value = clientWidth
        contentScrollWidth.value = scrollWidth
        contentClientHeight.value = clientHeight
        contentScrollHeight.value = scrollHeight
    }
})

onBeforeUnmount(() => {
    delete registry[id]
})

const scrolling = ref(false)
const timeoutHandler = ref(0)

const onScroll = e => {
    const {
        scrollTop: scrollTop0,
        scrollLeft: scrollLeft0,
    } = content.value

    const { clientWidth: xBarWidth, clientHeight: yBarHeight } = panel.value

    if (props.scrollX) {
        if (content.value.scrollHeight === content.value.clientHeight) return
        content.value.scrollLeft += e.deltaX
    }
    if (props.scrollY) content.value.scrollTop += e.deltaY

    const { scrollLeft: scrollLeft1, scrollTop: scrollTop1 } = content.value
    const scrollingX = scrollLeft0 !== scrollLeft1
    if (scrollingX) {
        handlerX.value.style.width = `${Math.floor(xBarWidth / scaleX.value - 6)}px`
        handlerX.value.style.left = `${Math.floor(scrollLeft1 / scaleX.value)}px`
    }
    const scrollingY = scrollTop0 !== scrollTop1
    if (scrollingY) {
        handlerY.value.style.height = `${Math.floor(yBarHeight / scaleY.value - 6)}px`
        handlerY.value.style.top = `${Math.floor(scrollTop1 / scaleY.value)}px`
    }

    if (scrollingX || scrollingY) {
        clearTimeout(timeoutHandler.value)
        scrolling.value = true
        timeoutHandler.value = setTimeout(() => scrolling.value = false, 300)
    }
}

const onScrollBarClickX = e => {
    const { clientWidth: xBarWidth } = panel.value
    const handlerLeft = [
        Math.floor(e.offsetX - handlerX.value.clientWidth / 2),
        0,
        xBarWidth - handlerY.value.clientWidth - 6
    ].sort()[1]
    handlerX.value.style.left = `${handlerLeft}px`

    const { scrollWidth, scrollTop } = content.value
    const scrollLeft = e.offsetX * scrollWidth / xBarWidth
    content.value.scrollTo({ top: scrollTop, left: scrollLeft, behavior: 'smooth' })
}

const onScrollBarClickY = e => {
    const { clientHeight: yBarHeight } = panel.value

    const handlerTop = [
        Math.floor(e.offsetY - handlerY.value.clientHeight / 2),
        0,
        yBarHeight - handlerY.value.clientHeight - 6
    ].sort()[1]
    handlerY.value.style.top = `${handlerTop}px`

    const { scrollHeight, scrollLeft } = content.value
    const scrollTop = e.offsetY * scrollHeight / yBarHeight;
    content.value.scrollTo({ top: scrollTop, left: scrollLeft, behavior: 'smooth' })
}

const onScrollBarHoverX = () => {
    if (!props.scrollX) return
    const { clientWidth: xBarWidth } = panel.value
    const { scrollWidth, clientWidth, } = content.value
    handlerX.value.style.width = `${Math.floor(xBarWidth / scaleX.value - 6)}px`
}

const onScrollBarHoverY = () => {
    if (!props.scrollY) return
    const { clientHeight: yBarHeight } = panel.value
    handlerY.value.style.height = `${Math.floor(yBarHeight / scaleY.value - 6)}px`
}

const onStartHandlerScrollX = e => {
    const { screenX } = e
    const origin = handlerX.value.offsetLeft - screenX
    const { clientWidth: xBarWidth } = panel.value
    const maxScrollLeft = xBarWidth - handlerX.value.clientWidth - 6

    scrolling.value = true
    const onScroll = e => {
        e.preventDefault()
        e.stopPropagation()
        e.stopImmediatePropagation()
        const handlerLeft = [
            origin + e.screenX,
            0,
            maxScrollLeft
        ].sort((a, b) => a - b)[1]
        handlerX.value.style.top = `${handlerLeft}px`

        content.value.scrollTo({
            top: content.value.scrollTop,
            left: content.value.scrollWidth * handlerLeft / xBarWidth,
        })
    }
    window.addEventListener('mousemove', onScroll)
    window.addEventListener('mouseup', e => {
        e.preventDefault()
        e.stopPropagation()
        e.stopImmediatePropagation()
        window.removeEventListener('mousemove', onScroll)
        scrolling.value = false
    }, { once: true })
}

const onStartHandlerScrollY = e => {
    const { screenY } = e
    const origin = handlerY.value.offsetTop - screenY
    const { clientHeight: yBarHeight } = panel.value
    const maxScrollTop = yBarHeight - handlerY.value.clientHeight - 6
    scrolling.value = true
    const onScroll = e => {
        e.preventDefault()
        e.stopPropagation()
        e.stopImmediatePropagation()

        const handlerTop = [
            origin + e.screenY,
            0,
            maxScrollTop
        ].sort((a, b) => a - b)[1]

        handlerY.value.style.top = `${handlerTop}px`

        content.value.scrollTo({
            top: content.value.scrollHeight * handlerTop / yBarHeight,
            left: content.value.scrollLeft
        })
    }
    window.addEventListener('mousemove', onScroll)
    window.addEventListener('mouseup', e => {
        e.preventDefault()
        e.stopPropagation()
        e.stopImmediatePropagation()
        window.removeEventListener('mousemove', onScroll)
        scrolling.value = false
    }, { once: true })
}

</script>

<style lang="less" scoped>
@import "./scroll-pannels.less";

.scroll-panel > .scroll-bar {
    z-index: v-bind(zIndex);
}
</style>