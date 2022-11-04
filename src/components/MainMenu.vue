<template>
    <div class="middle col-span-2 select-none relative">
        <div class="box select-none">
            {{message}}
        </div>
    </div>
</template>

<script>
export default {
    name: "MainMenu",
    data() {
        return {
            message: "Click the button below to achieve happiness."
        }
    },
    props: {
        hasTouched: {
            type: Boolean,
            default: false
        },
    },
    watch: {
        hasTouched(newValue, oldValue) {
            if (newValue) this.message = "Hmmm... Touch screen is kinda cheating. You wouldn't want to cheat happiness would you?";
        }
    },
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