<template>
    <div class="container">
        <div id="new-color">
            <ColorPicker v-model="new_color" />
            <button @click="addColor">Add Color</button>
        </div>

        <div id="colors" @dragend="dragEnd">
            <div
                v-for="(color, index) in colors"
                :key="index"
                class="item"
                @dragover.prevent
                @drop.prevent="dragDrop(index, $event)"
                @dragenter="dragEnter(index)"
                @dragleave="dragLeave"
            >
                <ColorPicker
                    draggable="true"
                    :value="color.color"
                    :class="{
                        hide: dragIndex === index,
                    }"
                    @input="updateColor(index, $event)"
                    @dragstart.native="dragStart(index, $event)"
                />
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component } from 'nuxt-property-decorator'
import ServerStatus from '@/components/ServerStatus.vue'
import ColorPicker from '@/components/ColorPicker.vue'

@Component({
    components: {
        ServerStatus,
        ColorPicker,
    },
})
export default class HomePage extends Vue {
    colors: { color: string; _id: number }[] = []
    id = 0
    dragIndex = -1
    dragging_on = -1

    getColor(index: number, color: { color: string; _id: number }) {
        const dragging_on = this.dragging_on
        const dragIndex = this.dragIndex
        const colors = this.colors
        const retVal =
            dragging_on !== -1 && dragIndex !== -1 && dragging_on === index
                ? colors[dragIndex].color
                : dragging_on !== -1 && dragIndex !== -1 && dragIndex === index
                ? colors[dragging_on].color
                : color.color

        return retVal
    }

    dragStart(index: number, event: DragEvent) {
        if (event.dataTransfer) {
            event.dataTransfer?.setData('colorIndex', String(index))
            setTimeout(() => {
                this.dragIndex = index
            }, 0)
        }
    }

    dragEnd() {
        this.dragIndex = -1
    }

    dragEnter(index: number) {
        this.dragging_on = index
    }

    dragLeave() {
        this.dragging_on = -1
    }

    dragDrop(drop_index: number, event: DragEvent) {
        const indexStr = event.dataTransfer?.getData('colorIndex')
        console.log({ indexStr })
        if (indexStr) {
            const index = parseInt(indexStr)
            const colors = [...this.colors]
            const temp = colors[drop_index]
            colors[drop_index] = colors[index]
            colors[index] = temp
            this.colors = colors
        }
    }

    new_color: string = '#3af'
    addColor() {
        this.colors.push({ color: this.new_color, _id: this.id++ })
        this.new_color = '#3af'
    }

    updateColor(index: number, color: string) {
        const colors = [...this.colors]
        colors[index].color = color
        this.colors = colors
    }
}
</script>

<style lang="scss">
/* Sample `apply` at-rules with Tailwind CSS
.container {
@apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/
.container {
    box-sizing: border-box;
    padding: 20px;
    display: grid;
    margin: 0 auto;
    min-height: 100vh;
    align-content: flex-start;
    text-align: center;
}

.title {
    font-family: 'Quicksand', 'Source Sans Pro', -apple-system,
        BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial,
        sans-serif;
    display: block;
    font-weight: 300;
    font-size: 100px;
    color: #35495e;
    letter-spacing: 1px;
}

#colors {
    display: grid;
    gap: 5px;
    grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    grid-auto-rows: 100px;
}

#new-color {
    display: grid;
    grid-template-columns: 100px 100px;
    gap: 20px;
    margin: 10px 0;

    button {
        outline: none;
        appearance: none;
        background: #1c1b2c;
        color: #fff;
        box-shadow: 0 4px 9px #0005;
        border-radius: 5px;
        transition: box-shadow 0.2s;

        &:hover {
            box-shadow: 0 8px 16px #0005;
        }
    }
}

.subtitle {
    font-weight: 300;
    font-size: 42px;
    color: #526488;
    word-spacing: 5px;
    padding-bottom: 15px;
}

.links {
    padding-top: 15px;
}

.item {
    // background: #1c2e42;
    border-radius: 3px;
    position: relative;
    cursor: pointer;
}

.hide {
    display: none;
}
</style>
