<template>
    <div class="scroll-panel"
         :class="{scrolling}"
         @wheel.passive="onScroll"
    >
        <div class="content"
             ref="content">
            <slot/>
        </div>
        <div v-if="scrollX" class="scroll-bar x"
             @click.prevent="onScrollBarClickX"
             @mouseenter="onScrollBarHoverX">
            <div ref="handlerX"
                 class="handler x"
                 @click.prevent.stop
                 @mousedown.left.prevent.stop="onStartHandlerScrollX"
            />
        </div>
        <div v-if="scrollY" class="scroll-bar y"
             @click.prevent="onScrollBarClickY"
             @mouseenter="onScrollBarHoverY">
            <div ref="handlerY"
                 class="handler y"
                 @click.prevent.stop
                 @mousedown.left.prevent.stop="onStartHandlerScrollY"
            />
        </div>
    </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
    scrollX: { type: Boolean, default: false },
    scrollY: { type: Boolean, default: false },
})


const content = ref(null)
const handlerX = ref(null)
const handlerY = ref(null)

const scrolling = ref(false)
const timeoutHandler = ref(0)

const onScroll = e => {
    const {
        scrollTop: scrollTop0,
        scrollLeft: scrollLeft0,
        scrollWidth,
        clientWidth,
        scrollHeight,
        clientHeight,
    } = content.value

    if (props.scrollX) content.value.scrollLeft += e.deltaX
    if (props.scrollY) content.value.scrollTop += e.deltaY

    const { scrollLeft: scrollLeft1, scrollTop: scrollTop1 } = content.value
    const scrollingX = scrollLeft0 !== scrollLeft1
    if (scrollingX) {
        handlerX.value.style.width = `${Math.floor(clientWidth * clientWidth / scrollWidth - 6)}px`
        handlerX.value.style.left = `${Math.floor(scrollLeft1 * clientWidth / scrollWidth)}px`
    }
    const scrollingY = scrollTop0 !== scrollTop1
    if (scrollingY) {
        handlerY.value.style.height = `${Math.floor(clientHeight * clientHeight / scrollHeight - 6)}px`
        handlerY.value.style.top = `${Math.floor(scrollTop1 * clientHeight / scrollHeight)}px`
    }

    if (scrollingX || scrollingY) {
        clearTimeout(timeoutHandler.value)
        scrolling.value = true
        timeoutHandler.value = setTimeout(() => scrolling.value = false, 300)
    }
}

const onScrollBarClickX = e => {
    const handlerLeft = [
        Math.floor(e.offsetX - handlerX.value.clientWidth / 2),
        0,
        e.target.clientWidth - handlerY.value.clientWidth - 6
    ].sort()[1]
    handlerX.value.style.left = `${handlerLeft}px`

    const { scrollWidth, clientWidth, scrollTop } = content.value
    const scrollLeft = e.offsetX * scrollWidth / clientWidth
    content.value.scrollTo({ top: scrollTop, left: scrollLeft, behavior: 'smooth' })
}

const onScrollBarClickY = e => {
    const handlerTop = [
        Math.floor(e.offsetY - handlerY.value.clientHeight / 2),
        0,
        e.target.clientHeight - handlerY.value.clientHeight - 6
    ].sort()[1]
    handlerY.value.style.top = `${handlerTop}px`

    const { scrollHeight, clientHeight, scrollLeft } = content.value
    const scrollTop = e.offsetY * scrollHeight / clientHeight;
    content.value.scrollTo({ top: scrollTop, left: scrollLeft, behavior: 'smooth' })
}

const onScrollBarHoverX = () => {
    if (!props.scrollX) return
    const { scrollWidth, clientWidth, } = content.value
    handlerX.value.style.width = `${Math.floor(clientWidth * clientWidth / scrollWidth - 6)}px`
}

const onScrollBarHoverY = () => {
    if (!props.scrollY) return
    const { scrollHeight, clientHeight, } = content.value
    handlerY.value.style.height = `${Math.floor(clientHeight * clientHeight / scrollHeight - 6)}px`
}

const onStartHandlerScrollX = e => {
    const { screenX } = e
    const origin = handlerX.value.offsetLeft - screenX
    const maxScrollLeft = content.value.clientWidth - handlerX.value.clientWidth - 6

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
            top: content.value.scrollHeight,
            left: content.value.scrollLeft * handlerLeft / content.value.clientWidth,
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
    const maxScrollTop = content.value.clientHeight - handlerY.value.clientHeight - 6
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
            top: content.value.scrollHeight * handlerTop / content.value.clientHeight,
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

<style scoped lang="less" src="./scroll-pannels.less"/>