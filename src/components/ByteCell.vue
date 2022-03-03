<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
    byte: Number,
    index: Number,
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
const emits = defineEmits(['change', 'select', 'mouseenter', 'mouseleave'])

// 管理编辑状态
const editStatus = ref(false)
function switchEditStatus(status) {
    if (status === undefined) {
        editStatus.value = !editStatus.value
    } else {
        editStatus.value = status
    }
}

// 值管理
let innerByteHexString = computed(() => props.byte.toString(16).padStart(2, '0').toUpperCase())
function changeByte(e) {
    let newValue = Number('0x' + e.target.value)
    if (!isNaN(newValue) && props.byte !== newValue) {
        let oldValue = props.byte
        props.byte = newValue
        emits('change', oldValue, newValue, props.index)
    }
    switchEditStatus(false)
}

// 被选中
function switchSelectStatus(status) {
    if (status === undefined) {
        emits('select', props.index, !props.selected)
    } else {
        emits('select', props.index, status)
    }

}

// 鼠标移入移出
function switchMouseStatus(status) {
    if (props.selected) return
    if (status) {
        emits('mouseenter', props.index)
    } else {
        emits('mouseleave', props.index)
    }
}
</script>

<template>
    <div
        class="cell"
        :style="{ 'background-color': props.bgColor, 'color': props.fgColor }"
        @click="switchSelectStatus()"
        @mouseenter="switchMouseStatus(true)"
        @mouseleave="switchMouseStatus(false)"
    >
        <span v-if="!editStatus">
            <!-- @dblclick="switchEditStatus(true)" -->
            {{ innerByteHexString }}
        </span>
        <input v-else :value="innerByteHexString" maxlength="2" @blur="changeByte" />
    </div>
</template>

<style scoped>
.cell {
    user-select: none;
    cursor: pointer;
}
.cell.selected {
    background-color: sandybrown;
}
input {
    width: 2em;
}
</style>
