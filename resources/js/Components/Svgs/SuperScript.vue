<script setup lang="ts">
const props = defineProps<{
    width?: string;
    height?: string;
}>();
</script>

<template>
    <svg xmlns="http://www.w3.org/2000/svg" :width="props.width ?? '24'" :height="props.height ?? '24'" viewBox="0 0 24 24" fill="none" stroke="#000000"
        stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="m4 19 8-8" />
        <path d="m12 19-8-8" />
        <path
            d="m20 12h-4c0-1.5 0.4417-2 1.5-2.5s2.5-1.1662 2.5-2.4985c7e-4 -0.47197-0.1706-0.92895-0.4837-1.29-0.313-0.36106-0.7475-0.60291-1.2266-0.6827s-0.9719 0.00763-1.3909 0.24676c-0.4191 0.23914-0.7375 0.61457-0.8988 1.0598" />
    </svg>
</template>