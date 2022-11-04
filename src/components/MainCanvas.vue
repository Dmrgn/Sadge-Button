<template>
    <canvas class="can" ref="can" :width="this.windowSize.width" :height="this.windowSize.height"></canvas>
</template>

<script>
const primary = "rgb(84, 103, 169)";
const secondary = "rgb(202, 118, 28)";
const black = "rgb(24,26,27)";
const confettiCols = [
    "red",
    "orange",
    "yellow",
    "green",
    "blue",
    "purple",
    "pink",
];
// dont spoil it for yourself smh >:|
const links = [
    "https://www.amazon.ca/Duck-Tape-Brand-241818-Unicorns/dp/B099P413C2/ref=sr_1_1?crid=2ZODFF4N32WKC&keywords=unicorn+duck+tape+%3A%29&qid=1666964933&sprefix=unicorn+duck+tape+%2Caps%2C104&sr=8-1",
    "https://www.google.ca/search?q=cats&sxsrf=ALiCzsbb80XzgDtjewvVMWgvZn7mxIYp8Q:1667441121824&source=lnms&tbm=isch&sa=X&ved=2ahUKEwii4f-d9pD7AhVarYkEHVlIBJUQ_AUoAXoECAEQAw&biw=2560&bih=1239&dpr=1",
    "https://youtu.be/y5OXoEtcen8?t=25",
];

export default {
    name:"MainCanvas",
    data() {
        return {
            state: "unclicked",
            transitionAmount: 1,
            startedRedirect: false,
            confetti: [],
            windowSize: {width:0, height:0},
            range: 300,
            circleWidth: 10,
            waveSize: 10,
            waveX: 0,
            waveIntensity: 0.01,
            frameCount: 0,
            sensitivity: 0.1,
            buttonPos: {x:0,y:0},
            buttonVel: {x:0,y:0},
            buttonSize: {width: 132, height: 40},
            buttonColour: primary,
            buttonOrigin: {x:0,y:0},
            ctx: null
        }
    },
    props: {
        mousePos: {
            type: Object,
            default: {x:0, y:0}
        },
        isClicked: {
            type: Boolean,
            default: false
        }
    },
    mounted() {
        // start listeners
        window.addEventListener("resize", this.handleResize);
        this.handleResize();
        setInterval(this.interval, 1000/60);
        this.ctx = this.$refs.can.getContext("2d");
        // set button to default position
        this.buttonPos.x = this.buttonOrigin.x;
        this.buttonPos.y = this.buttonOrigin.y;
    },
    destroyed() {
        window.removeEventListener("resize", this.handleResize);
        clearInterval(this.interval);
    },
    watch: {
        isClicked(newValue, oldValue) {
            if (newValue) {
                this.$emit("receivedClick");
                if (this.mousePos.x > this.buttonPos.x && this.mousePos.x < this.buttonPos.x+this.buttonSize.width
                && this.mousePos.y > this.buttonPos.y && this.mousePos.y < this.buttonPos.y+this.buttonSize.height) {
                    this.state = "clicked";
                }
            }
        }
    },
    methods: {
        handleResize() {
            this.windowSize.width = window.innerWidth;
            this.windowSize.height = window.innerHeight;
            this.buttonOrigin.x = this.windowSize.width/4;
            if (this.windowSize.width > this.windowSize.height)
                this.buttonOrigin.y = this.windowSize.height/3 + 125;
            else
                this.buttonOrigin.y = this.windowSize.height/2 + 125;
        },
        interval() {
            this.frameCount++;
            // draw background
            this.ctx.clearRect(0, 0, this.windowSize.width, this.windowSize.height);

            switch (this.state) {
                case "unclicked":
                    // move the button away from the mouse
                    this.moveButton();
                    this.drawButton();
                    // draw the sine wave
                    this.sineWave(0, 0, 1);
                    this.sineWave(0, this.windowSize.height, -1);
                    break;
                case "clicked":
                    // draw but don't move the button
                    this.drawButton();
                    // draw the sine wave
                    this.sineWave(0, 0, 1);
                    this.sineWave(0, this.windowSize.height, -1);
                    // draw transition
                    this.updateTransitionAmount();
                    // draw the sine wave transition
                    this.drawSineWaveTransition(0, 0, 50);
                    this.drawSineWaveTransition(0, 0, 50, true);
                    this.drawSineWaveTransition(0, this.windowSize.height-50, 50);
                    this.drawSineWaveTransition(0, this.windowSize.height-50, 50, true);
                    // draw confetti
                    this.drawConfettiTransition();
                    break;
                case "transitioned":
                    if (this.startedRedirect == false) {
                        this.startedRedirect = true;
                        this.redirectToHappiness();
                    }
                    break;
                default:
                    console.log("invalid state", this.state);
                    break;
            }
        },
        mapRange(n, min1, max1, min2, max2) {
            let range1 = max1-min1;
            let percent1 = n/range1;
            let range2 = max2-min2;
            let value = range2*percent1;
            return value+min2;
        },
        moveButton() {
            // move button away from the mouse
            const difference = {
                x: (this.buttonPos.x+this.buttonSize.width/2) - this.mousePos.x,
                y: (this.buttonPos.y+this.buttonSize.height/2) - this.mousePos.y
            }
            let total = Math.abs(difference.x) + Math.abs(difference.y);
            const unitDifference = {x:difference.x/total, y:difference.y/total};
            let distance = Math.sqrt((difference.x**2) + (difference.y**2));
            this.waveIntensity += 0.01 + this.mapRange(Math.min(distance,500), 0, 500, 0.5, 0);
            this.waveIntensity *= 0.8;
            this.waveSize += 1 + this.mapRange(Math.min(distance,500), 0, 500, 0.5, 0) * 20;
            this.waveSize *= 0.8;
            // add to vel
            this.buttonVel.x += unitDifference.x * Math.max(this.range-distance,0)*this.sensitivity;
            this.buttonVel.y += unitDifference.y * Math.max(this.range-distance,0)*this.sensitivity;
            // if the vel is 0, and the mouse is not in the way,
            // start moving back to origin
            if (this.buttonVel.x == 0 && this.buttonVel.y == 0) {
                const diffMouseToOrigin = {
                    x: this.mousePos.x-this.buttonOrigin.x,
                    y: this.mousePos.y-this.buttonOrigin.y
                }
                const distMouseToOrigin = Math.sqrt(diffMouseToOrigin.x**2 + diffMouseToOrigin.y**2);
                if (distMouseToOrigin > 300) {
                    const originDifference = {
                        x: this.buttonPos.x - this.buttonOrigin.x,
                        y: this.buttonPos.y - this.buttonOrigin.y,
                    }
                    this.buttonPos.x -= originDifference.x * this.sensitivity;
                    this.buttonPos.y -= originDifference.y * this.sensitivity;
                }
            }
            // limit vel
            this.buttonVel.x *= 0.8;
            this.buttonVel.y *= 0.8;
            if (Math.abs(this.buttonVel.x) < 0.01) this.buttonVel.x = 0;
            if (Math.abs(this.buttonVel.y) < 0.01) this.buttonVel.y = 0;
            // add vel to pos
            this.buttonPos.x += this.buttonVel.x;
            this.buttonPos.y += this.buttonVel.y;
            // limit pos
            if (this.buttonPos.x < 0 && this.buttonVel.x < 0) this.buttonVel.x *= -5;
            if (this.buttonPos.y < 0 && this.buttonVel.y < 0) this.buttonVel.y *= -5;
            if (this.buttonPos.x+this.buttonSize.width > this.windowSize.width && this.buttonVel.x > 0) this.buttonVel.x *= -5;
            if (this.buttonPos.y+this.buttonSize.height > this.windowSize.height && this.buttonVel.y > 0) this.buttonVel.y *= -5;
        },
        drawButton() {
            // draw button
            this.ctx.fillStyle= this.buttonColour;
            this.ctx.beginPath();
            this.ctx.roundRect(this.buttonPos.x, this.buttonPos.y, this.buttonSize.width, this.buttonSize.height, 4);
            this.ctx.fill();
            this.ctx.fillStyle= "white";
            this.ctx.font = "18px monospace";
            this.ctx.fillText(this.frameCount%600<20?"Click ;)":"Click :)", this.buttonPos.x+this.buttonSize.width/5, this.buttonPos.y+this.buttonSize.height/2+5);
        },
        sineWave(transX, transY, inverted) {
            const numCirclesWidth = this.windowSize.width/this.circleWidth;
            const numCirclesHeight = this.windowSize.height/this.circleHeight;
            let origPoints = [];
            this.waveX+= this.waveIntensity/2;
            for (let i = 0; i < numCirclesWidth; i++) {
                origPoints.push({x:i*this.circleWidth+transX, y:Math.sin(i+this.waveX)*(this.waveSize)+transY});
            }
            this.ctx.lineWidth = 18;
            this.ctx.beginPath();
            let points = [...origPoints];
            this.ctx.moveTo(points[0].x, points[0].y);
            while(points.length > 1) {
                const prev = points.shift();
                let halfway = {
                    x: prev.x + (points[0].x - prev.x),
                    y: prev.y + (points[0].y - prev.y) + 10*inverted
                }
                this.ctx.bezierCurveTo(prev.x, prev.y, halfway.x, halfway.y, points[0].x, points[0].y);
            }
            this.ctx.strokeStyle = secondary;
            this.ctx.stroke();
            this.ctx.beginPath();
            points = [...origPoints];
            this.ctx.moveTo(points[0].x, points[0].y);
            while(points.length > 1) {
                const prev = points.shift();
                let halfway = {
                    x: prev.x + (points[0].x - prev.x),
                    y: prev.y + (points[0].y - prev.y) + 10*inverted
                }
                this.ctx.bezierCurveTo(prev.x+4*inverted, prev.y+4*inverted, halfway.x+4*inverted, halfway.y+4*inverted, points[0].x+4*inverted, points[0].y+4*inverted);
            }
            this.ctx.strokeStyle = primary;
            this.ctx.stroke();
        },
        updateTransitionAmount() {
            if (this.transitionAmount > this.windowSize.width) {
                this.state = "transitioned";
                return;
            }
            this.transitionAmount += (this.windowSize.width/60)*Math.abs(Math.sin(this.mapRange(this.transitionAmount, 0, this.windowSize.width, Math.PI/8, Math.PI/2)));
        },
        drawSineWaveTransition(transX, transY, height, inverted) {
            this.ctx.fillStyle = black;
            this.ctx.fillRect(inverted ? (this.windowSize.width-this.transitionAmount)+transX : 0, transY, this.transitionAmount, height);
            this.ctx.beginPath();
            this.ctx.ellipse(this.transitionAmount, transY, height, height, 0, 0, 2 * Math.PI);
            this.ctx.fill();
        },
        drawConfettiTransition() {
            function createConfetti(x, y, context) {
                return {
                    x: x,
                    y: y,
                    xVel: Math.random()*context.mapRange(x, 0, context.windowSize.width, 1, -1)*10,
                    yVel: Math.random()*-10-10,
                    col: confettiCols[Math.floor(Math.random()*confettiCols.length)]
                }
            }
            // create new confetti on each side
            this.confetti.push(createConfetti(0, this.windowSize.height, this));
            this.confetti.push(createConfetti(this.windowSize.width, this.windowSize.height, this));
            // update existing confetti
            this.confetti.forEach((item, i)=>{
                // update position
                item.x += item.xVel;
                item.y += item.yVel;
                item.yVel += 0.5;
                // draw
                this.ctx.beginPath();
                this.ctx.fillStyle = item.col;
                this.ctx.ellipse(item.x, item.y, 10, 10, 0, 0, 2 * Math.PI);
                this.ctx.fill();
            });
        },
        redirectToHappiness() {
            window.location.href = links[Math.floor(Math.random()*links.length)];
        }
    },
}
</script>

<style>
    .can {
        @apply w-full;
    }
</style>