<template>
    <div class="color-picker-container" @click="openPicker">
        <div ref="color" class="color-picker"></div>
        <div class="color-swatch" :style="{ background: value }"></div>
    </div>
</template>

<script lang="ts">
import { Component, Prop } from 'nuxt-property-decorator'
import Picker from 'vanilla-picker'
import Vue from 'vue'

@Component
export default class ColorPicker extends Vue {
    @Prop({ required: true }) value!: string
    picker?: Picker

    openPicker(clickEvent: MouseEvent) {
        if (this.picker) {
            this.picker.openHandler(clickEvent)
        }
    }

    changeColor(color: any) {
        this.$emit('input', color.hex)
    }

    mounted() {
        const color = this.$refs.color as HTMLDivElement
        this.picker = new Picker({
            parent: color,
            alpha: false,
            color: this.value,
        })
        this.picker.onChange = this.changeColor
    }
}
</script>

<style lang="scss">
.color-picker-container {
    width: 100%;
    height: 100%;

    .color-swatch {
        width: 100%;
        height: 100%;
        border-radius: 3px;
    }
}
</style>
