<template>
    <div class="container">
        <h1>Ctrl + click a color to delete it.</h1>
        <h1>Right click a color to copy it.</h1>
        <h1>There is no undo functionality.</h1>
        <div id="new-color">
            <ColorPicker v-model="new_color" />
            <button @click="addColor">Add Color</button>
        </div>

        <transition-group
            id="colors"
            name="colorpop"
            tag="div"
            @dragend="dragEnd"
        >
            <div
                v-for="(color, index) in colors"
                :key="color._id"
                class="item"
                @dragover.prevent
                @drop.prevent="dragDrop(index, $event)"
                @dragenter="dragEnter(index)"
                @dragleave="dragLeave"
                @click.right.prevent.exact="copy(index)"
                @click.ctrl.exact="deleteColor(index)"
            >
                <ColorPicker
                    draggable="true"
                    :value="
                        dragIndex === index || dragging_on === index
                            ? getColor(index, color)
                            : color.color
                    "
                    :class="{
                        hide:
                            (dragIndex === index && dragging_on === -1) ||
                            dragging_on === index,
                    }"
                    @input="updateColor(index, $event)"
                    @dragstart.native="dragStart(index, $event)"
                />
            </div>
        </transition-group>
    </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component } from 'nuxt-property-decorator'
import ServerStatus from '@/components/ServerStatus.vue'
import ColorPicker from '@/components/ColorPicker.vue'
import { TYPE } from 'vue-toastification/src/ts/constants'
import { v4 } from 'uuid'

@Component({
    components: {
        ServerStatus,
        ColorPicker,
    },
})
export default class HomePage extends Vue {
    colors: { color: string; _id: string }[] = []
    dragIndex = -1
    dragging_on = -1

    beforeMount() {
        if (localStorage) {
            const colors = localStorage.getItem('colors')
            if (colors) {
                console.log(colors)
                const colors_arr: any[] = JSON.parse(colors)
                this.colors = colors_arr
            }
        }
    }

    getColor(index: number, color: { color: string; _id: string }) {
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

    deleteColor(index: number) {
        const colors = [...this.colors]
        colors.splice(index, 1)
        this.colors = colors
        this.updateStore()
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
        this.dragging_on = -1
        this.dragIndex = -1
        console.log({ indexStr })
        if (indexStr && Number(indexStr) !== drop_index) {
            const index = parseInt(indexStr)
            const colors = [...this.colors]
            const temp = colors[drop_index]
            colors[drop_index] = colors[index]
            colors[index] = temp
            this.colors = colors
            this.updateStore()
        }
    }

    new_color: string = '#3af'
    addColor() {
        this.colors.push({ color: this.new_color, _id: v4() })
        const rgb = []
        for (let i = 0; i < 3; i++) {
            let val = Math.floor(Math.random() * 256).toString(16)
            if (val.length === 1) val = '0' + val
            rgb[i] = val
        }
        this.new_color = `#${rgb[0]}${rgb[1]}${rgb[2]}`
        this.updateStore()
    }

    updateColor(index: number, color: string) {
        const colors = [...this.colors]
        colors[index].color = color
        this.colors = colors
        this.updateStore()
    }

    updateStore() {
        localStorage.setItem('colors', JSON.stringify(this.colors))
    }

    copy(index: number) {
        const color = this.colors[index].color
        this.$toast.clear()
        this.$toast(`Copied ${color} to clipboard!`, {
            timeout: 2000,
            type: TYPE.SUCCESS,
        })
        navigator.clipboard.writeText(color)
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
    position: relative;
    cursor: pointer;
    padding: 5px;
    box-sizing: border-box;
}

.hide > * {
    opacity: 0;
}

.colorpop-enter,
.colorpop-leave-to {
    opacity: 0;
    transform: scale(0);
}

.colorpop-leave,
.colorpop-enter-to {
    opacity: 1;
    transform: scale(1);
}

.colorpop-enter-active {
    transition: transform 0.13s, opacity 0.13s;
}
</style>
