<script setup>
import { computed } from 'vue'

const props = defineProps({
    byte: Number,
    selected: Boolean,
    bgColor: {
        type: String,
        default: '#ddd',
    },
    fgColor: {
        type: String,
        default: '#000'
    }
})

// 值管理
let char = computed(() => {
    if (props.byte > 32 && props.byte < 127) {
        return String.fromCharCode(props.byte)
    } else {
        return '.'
    }
})

</script>

<template>
    <div class="cell" :style="{ 'background-color': props.bgColor, 'color': props.fgColor }">
        <span>{{ char }}</span>
    </div>
</template>

<style scoped>
span {
    user-select: none;
    width: 2em;
}
.cell.selected {
    background-color: sandybrown;
}
</style>
