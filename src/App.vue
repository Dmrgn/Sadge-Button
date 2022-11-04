<template>
    <div @click="clicked" @touchstart="touch" @touchend="untouch" @touchcancel="untouch" @mousemove="updateMousePos" class="wrapper">
        <MainCanvas @received-click="receivedClick" :isClicked="this.isClicked" :mousePos="this.mousePos" />
        <div class="grid grid-cols-4 justify-evenly w-full absolute top-0">
            <div class=""></div>
            <MainMenu :hasTouched="this.hasTouched" />
            <div class="hidden md:block"></div>
        </div>
    </div>
	<div class="footer">
		<div class="mx-2">
			{{this.splash}}
		</div>
		<div class="mx-2">
			<a href="https://github.com/dmrgn/SadgeButton">
				<img src="https://img.icons8.com/ios-glyphs/30/000000/github.png"/>
			</a>
		</div>
	</div>
</template>

<script>
	const splashes = [
		"Made with ♥ ->",
		"The button winks ;)",
		"Yes its possible",
		"('-')",
		"I believe in you!",
		"Keep going!",
		"Click faster!",
		";-; sadge",
		":pittypat:",
		"Ⓐ for effort!",
		"Time flies, even when you're crying.",
		"Touchscreen is cringey anyways."
	];

    import MainMenu from "./components/MainMenu.vue"
    import MainCanvas from "./components/MainCanvas.vue"
    export default {
		name: "App",
		data() {
			return {
				mousePos: {x:0,y:0},
				isClicked: false,
				splash: splashes[Math.floor(Math.random()*splashes.length)],
				touched: false,
				hasTouched: false,
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
			clicked(e) {
				if (!this.touched)
					this.isClicked = true;
				console.log(e.type);
			},
			receivedClick() {
				this.isClicked = false;
			},
			touch(e) {
				this.touched = true;
				this.hasTouched = true;
			},
			untouch(e) {
				setTimeout(()=>{
					this.touched = false;
					console.log(e.type);
				}, 100);
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
	.footer {
		@apply absolute flex justify-center w-full text-white;
		top: 93%;
	}
</style>
