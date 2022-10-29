<template>
    <div @click="clicked" @mousemove="updateMousePos" class="wrapper">
        <MainCanvas :mousePos="this.mousePos" />
        <div class="grid grid-cols-4 justify-evenly w-full absolute top-0">
            <div class=""></div>
            <MainMenu />
            <div class="hidden md:block"></div>
        </div>
    </div>
</template>

<script>
    import MainMenu from "./components/MainMenu.vue"
    import MainCanvas from "./components/MainCanvas.vue"
    export default {
		name: "App",
		data() {
			return {
				mousePos: {x:0,y:0},
			}
		},
		components: {
			MainMenu,
			MainCanvas
		},
		methods: {
			updateMousePos(event) {
				this.mousePos.x = event.clientX;
				this.mousePos.y = event.clientY;
			},
			clicked() {
				
			}
		},
    }
</script>

<style lang="scss">
	@import './assets/tailwind.scss';
	body {
		font-family: monospace;
		font-size: 18px;
	}
	.wrapper {
		background-color: $black;
	}
</style>
