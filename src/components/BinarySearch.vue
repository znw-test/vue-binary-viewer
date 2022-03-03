<script setup>
import { ref } from 'vue';

const props = defineProps({
    file: File
})

const emits = defineEmits(['search', 'clear'])

let hexArray = ref([])
function changeValue(i, e) {
    let n = Number('0x' + e.target.value)
    if (!isNaN(n)) {
        hexArray.value[i] = n
    } else {
        e.target.value = hexArray.value[i].toString(16)
    }
}
function addByte() {
    hexArray.value.push(0)
}
function deleteByte() {
    hexArray.value.pop()
}

let result = ref([])
let searching = ref(false)
async function search() {
    if (hexArray.value.length === 0) return
    console.time("FileSearch")
    const ab = await props.file.arrayBuffer()
    const ui8view = new Uint8Array(ab)
    let tmpresult = []
    for (let i = 0; i <= ui8view.length - hexArray.value.length; i++) {
        if (ui8view[i] !== hexArray.value[0]) {
            continue
        }
        let match = true
        for (let j = 1; j < hexArray.value.length; j++) {
            if (ui8view[i + j] !== hexArray.value[j]) {
                match = false
                break
            }
        }
        if (match) {
            tmpresult.push(new Array(hexArray.value.length).fill(0).map((_, y) => i + y))
        }
    }
    console.timeEnd("FileSearch")
    emits('search', tmpresult)
    result.value = tmpresult.map(arr=>arr.map(b=>b.toString(16) + 'h'))
    return tmpresult
}

function clear() {
    emits('clear')
    result.value = []
}

</script>

<template>
    <div>
        <div>
            <input
                v-for="(hex, i) in hexArray"
                :value="hex.toString(16)"
                @blur="(e) => changeValue(i, e)"
                maxlength="2"
            />
        </div>
        <button @click="addByte">Add</button>
        <button @click="deleteByte">Delete</button>
        <button @click="search">Search</button>
        <button @click="clear">Clear</button>
        <ul>
            <span v-show="searching">Search...</span>
            <span v-show="!searching && result.length === 0">No Result</span>
            <li v-show="!searching" v-for="r in result">{{ r.toString() }}</li>
        </ul>
    </div>
</template>

<style scoped>
span {
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
