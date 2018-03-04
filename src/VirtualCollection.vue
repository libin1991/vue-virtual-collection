<style lang="less" scoped>
    .vue-virtual-collection {
        overflow: auto;
        &-container {
            position: relative;
        }
        .cell-container {
            position: absolute;
            top: 0;
            background: #ddd;
        }
    }
</style>

<template>
    <div class="vue-virtual-collection" :style="outerStyle" @scroll="onScroll" ref="outer">
        <div class="vue-virtual-collection-container" :style="scrollHeight">
            <div v-for="(item, index) in displayItems" class="cell-container" :key="index" :style="getComputedStyle(item, index)">
                <slot name="cell" :data="collection[item.index].data"></slot>
            </div>
        </div>
    </div>
</template>

<script>
    import Vue from 'vue'
    export default {
        props: ['cellSizeAndPositionGetter', 'collection', 'height', 'width'],
        data() {
            return {
                displayItems: []
            }
        },
        created() {
            this.flushDisplayItems()
        },
        methods: {
            getComputedStyle(item, index) {
                const { width, height, x, y } = item
                return {
                    left: `${x}px`,
                    top: `${y}px`,
                    width: `${width}px`,
                    height: `${height}px`
                }
            },
            checkIfNeedRender(item, index) {
                console.log(checkIfNeedRender)
                return true
            },
            onScroll() {
                const { scrollTop, scrollLeft } = this.$refs.outer
                this.flushDisplayItems()
            },
            isInViewPort({ x, y, width, height }, { scrollTop, scrollLeft }) {
                const { height: outerHeight, width: outerWidth } = this
                if (x + width < scrollLeft || x > scrollLeft + outerWidth || y > outerHeight + scrollTop || y + height < scrollTop) {
                    return false
                } else {
                    return true
                }
            },
            flushDisplayItems() {
                let scrollTop = 0
                let scrollLeft = 0
                if (this.$refs.outer) {
                    scrollTop = this.$refs.outer.scrollTop
                    scrollLeft = this.$refs.outer.scrollLeft
                }
                this.displayItems = this.cellSizeAndPosition.filter(sizeAndPosition => this.isInViewPort(sizeAndPosition, { scrollTop, scrollLeft }))
            }
        },
        computed: {
            cellSizeAndPosition() {
                return this.collection.map((item, index) => ({
                    ...this.cellSizeAndPositionGetter(item),
                    index
                }))
            },
            scrollHeight() {
                let containerHeight = 0
                const { cellSizeAndPosition } = this
                cellSizeAndPosition.forEach(sizeAndPosition => {
                    const { y, height } = sizeAndPosition
                    const bottom = y + height
                    if (bottom > height) {
                        containerHeight = bottom
                    }
                })
                return {
                    height: containerHeight + 'px'
                }
            },
            outerStyle() {
                return {
                    height: this.height + 'px',
                    width: this.width + 'px'
                }
            }
        }
    }
</script>