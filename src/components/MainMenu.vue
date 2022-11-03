<template>
    <div class="middle col-span-2 select-none relative">
        <div class="box select-none">
            Click the button below to achieve happiness.
        </div>
    </div>
</template>

<script>
export default {
    name: "MainMenu",
}
</script>

<style lang="scss">
    @use 'sass:color';
    @import '../assets/tailwind.scss';

    .box {
        @apply p-4 w-full bg-slate-300 rounded text-white shadow-md;
        background-color: color.adjust($light-black, $alpha: -0.2);
    }
    .middle {
        margin-top: 33vh;
    }
</style>