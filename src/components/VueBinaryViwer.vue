<script setup>
import { computed, ref } from 'vue';
import ByteCell from './ByteCell.vue'
import CharCell from './CharCell.vue'
import Palette from './Palette.vue'
import BinarySearch from './BinarySearch.vue';
import ByteWrapper from '../util/ByteWrapper'

// 选择文件并查看
let showLength = ref(1024) // 字节数
let page = ref(0)
const pageMax = ref(0)
let rowNum = computed(() => Math.ceil(bytesArray.value.length / 16))
let fileHandle = ref(null) // value: FileSystemFileHandle
let fileData = ref(null) // File
let bytesArray = ref([]) // value: ByteWrapper[]
const pickerOption = {
  multiple: false
}
async function getFile() {
  // open file picker
  try {
    let [tmpfileHandle] = await globalThis.showOpenFilePicker(pickerOption);
    fileHandle.value = tmpfileHandle
    if (!fileHandle.value) {
      return
    }
    fileData.value = await tmpfileHandle.getFile() // File
    bytesArray.value = await showBytes(fileData.value, page.value, showLength.value)
  } catch (error) {
    console.error(error)
  }
}

/**
 * 
 */
async function showBytes(fd, n, len) {
  try {
    const { size: fileSize, name: fileName } = fd // fileSize: 文件字节数 fileName: 文件名
    let bytes = new Uint8Array(await fd.slice(n * len, (n + 1) * len).arrayBuffer())
    pageMax.value = Math.floor(fileSize / showLength.value)
    let tmp = []
    bytes.forEach((byte, i) => {
      tmp.push(new ByteWrapper(byte, i2Index(i), false))
    })
    return tmp
  } catch (error) {
    console.error(error)
    return
  }
}
async function showPrevious() {
  page.value--
  bytesArray.value = await showBytes(fileData.value, page.value, showLength.value)
  showSearchResult()
}
async function showNext() {
  page.value++
  bytesArray.value = await showBytes(fileData.value, page.value, showLength.value)
  showSearchResult()
}

function i2Index(i) {
  return page.value * showLength.value + i
}
function index2I(index) {
  return index - page.value * showLength.value
}
function isIndexInThisPage(index) {
  return index >= page.value * showLength.value && index < (page.value + 1) * showLength.value
}

function byteSelect(index, value) {
  let i = index2I(index)
  bytesArray.value[i].selected = value
  if (value) {
    bytesArray.value[i].bgColor = bgColor.value
    bytesArray.value[i].fgColor = fgColor.value
  } else {
    bytesArray.value[i].bgColor = undefined
    bytesArray.value[i].fgColor = undefined
  }
}


let bgColor = ref('#ff0000')
let fgColor = ref('#ffffff')


let searchResult = ref([])
/**
 * 
 * @param {Number[][]} result 
 */
function showSearchResult(result) {
  if (result) {
    searchResult.value = result
  }
  searchResult.value.forEach(array => {
    array.forEach(index => {
      if (isIndexInThisPage(index)) {
        byteSelect(index, true)
      }
    })
  })
}
function clearSearchResult() {
  searchResult.value.forEach(array => {
    array.forEach(index => {
      if (isIndexInThisPage(index)) {
        byteSelect(index, false)
      }
    })
  })
  searchResult.value = []
}

function mouseEnter(index) {
  let i = index2I(index)
  bytesArray.value[i].bgColor = bgColor.value
  bytesArray.value[i].fgColor = fgColor.value
}

function mouseLeave(index) {
  let i = index2I(index)
  bytesArray.value[i].bgColor = undefined
  bytesArray.value[i].fgColor = undefined
}


let toolBoxStatus = ref(false)
function switchToolBoxStatus() {
  toolBoxStatus.value = !toolBoxStatus.value
}
</script>

<template>
  <h1 v-if="!fileHandle" @click="getFile" class="title">Pick you File</h1>
  <div v-else class="main">
    <div class="tool-box">
      <div @click="switchToolBoxStatus" class="tool-box_icon">Tool</div>
      <div v-show="toolBoxStatus" class="tool-box_content">
        <button class="btn" :disabled="page <= 0" @click="showPrevious">Previous Page</button>
        <button class="btn" :disabled="page >= pageMax" @click="showNext">Next Page</button>
        <label>
          Background color
          <Palette type="color" v-model:value="bgColor" />
        </label>
        <label>
          Foreground color
          <Palette type="color" v-model:value="fgColor" />
        </label>
        <div>
          Binary Search
          <BinarySearch :file="fileData" @search="showSearchResult" @clear="clearSearchResult"></BinarySearch>
        </div>
      </div>
    </div>
    <div style="display: flex;">
      <div class="byte-table">
        <div class="table-row">
          <div class="table-cell header"></div>
          <div class="table-cell header" v-for="x in 16">{{ (x - 1).toString(16).toUpperCase() }}</div>
        </div>
        <div v-for="rowIndex in rowNum" class="table-row">
          <div
            class="table-cell header"
          >{{ (page * showLength / 16 + (rowIndex - 1)).toString(16).padStart(6, '0').toUpperCase() + 'h' }}</div>
          <ByteCell
            v-for="item in bytesArray.slice((rowIndex - 1) * 16, rowIndex * 16)"
            :key="item.index"
            :byte="item.byte"
            :index="item.index"
            :selected="item.selected"
            :bg-color="item.bgColor"
            :fg-color="item.fgColor"
            :class="['table-cell']"
            @select="byteSelect"
            @mouseenter="mouseEnter"
            @mouseleave="mouseLeave"
          ></ByteCell>
        </div>
      </div>
      <div class="char-table">
        <div class="table-row">
          <div class="table-cell header" v-for="x in 16">{{ (x - 1).toString(16).toUpperCase() }}</div>
        </div>
        <div v-for="rowIndex in rowNum" class="table-row">
          <CharCell
            v-for="item in bytesArray.slice((rowIndex - 1) * 16, rowIndex * 16)"
            :class="['table-cell']"
            :key="item.index"
            :byte="item.byte"
            :selected="item.selected"
            :bg-color="item.bgColor"
            :fg-color="item.fgColor"
          ></CharCell>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.title {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  cursor: pointer;
  color: azure;
  font-size: 4rem;
}
.main {
  width: 80%;
  margin: 5% auto 0;
  text-align: center;
}
.byte-table,
.char-table {
  display: table;
  width: 50%;
}

.table-row {
  display: table-row;
  height: 1.5rem;
}
.table-cell {
  display: table-cell;
  text-align: center;
  vertical-align: middle;
  min-width: 2rem;
}
.header {
  background-color: aquamarine;
}

.tool-box {
  position: fixed;
  right: 5vw;
  bottom: 5vh;
}
.tool-box_icon {
  background-color: white;
  z-index: 1;
  width: 4vw;
  height: 4vw;
  line-height: 4vw;
  border-radius: 50%;
  text-align: center;
  cursor: pointer;
}
.tool-box_content {
  position: absolute;
  background-color: gray;
  width: 20vw;
  height: 40vh;
  top: -40vh;
  left: -20vw;
  color: white;
  overflow: auto;
}
.tool-box_content .btn {
  width: 8vw;
  height: 3vw;
  display: block;
  cursor: pointer;
  margin: 10px auto;
  border-radius: 4px;
  color: #fff;
  background-color: #337ab7;
  border: none;
  border: 1px solid #2e6da4;
}
.tool-box_content .btn:hover {
  color: #fff;
  background-color: #286090;
  border-color: #204d74;
}
.tool-box_content .btn:disabled {
  color: #fff;
  background-color: #547796;
  border-color: #204d74;
  cursor: not-allowed;
}
</style>
