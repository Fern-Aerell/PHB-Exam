<script setup lang="ts">
const props = defineProps<{
    width?: string;
    height?: string;
}>();
</script>

<template>
    <svg :width="props.width ?? '800px'" :height="props.height ?? '800px'" viewBox="0 0 24 24" fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <g id="Edit / Text_Align_Left">
            <path id="Vector" d="M4 18H14M4 14H20M4 10H14M4 6H20" stroke="#000000" stroke-width="2"
                stroke-linecap="round" stroke-linejoin="round" />
        </g>
    </svg>
</template>