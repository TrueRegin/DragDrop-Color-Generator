<template>
    <div class="color-picker-container" @click.exact="openPicker">
        <div ref="color" class="color-picker"></div>
        <div
            class="color-swatch"
            :class="{ selected }"
            :style="{ background: value }"
        ></div>
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
    selected = false

    openPicker(clickEvent: MouseEvent) {
        if (this.picker) {
            this.picker.openHandler(clickEvent)
        }
        this.selected = true
    }

    onPickerClose() {
        this.selected = false
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
        this.picker.onClose = this.onPickerClose
    }
}
</script>

<style lang="scss">
.color-picker-container {
    width: 100%;
    height: 100%;
    box-shadow: 0 3px 6px #0005;
    border-radius: 3px;

    .color-swatch {
        width: 100%;
        height: 100%;
        border-radius: 3px;

        transition: transform 180ms;

        &:hover,
        &.selected {
            transform: scale(0.9);
        }
    }
}
</style>
