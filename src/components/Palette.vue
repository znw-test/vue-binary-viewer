<script setup>
const props = defineProps({
    value: String
})
const emits = defineEmits(['update:value'])
</script>

<template>
    <input type="color" :value="props.value" @blur="(e) => emits('update:value', e.target.value)" />
</template>

<style scoped>
</style>
