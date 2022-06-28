<template >
    <div class="text-xl m-3">
        <canvas ref="canvas" :width="WIDTH" :height="HEIGHT"></canvas>
    </div>
</template>
<script setup lang="ts">
const canvas = $ref<HTMLCanvasElement>();
const ctx = $computed(() => canvas!.getContext('2d')!);
const HEIGHT = 600;
const WIDTH = 600;
interface Point {
    x: number,
    y: number
}
interface Line {
    start: Point,
    length: number,
    theta: number
}
const pendingTask: Function[] = [];
function init() {
    ctx.strokeStyle = '#ddd'
    const line = {
        start: { x: WIDTH / 2, y: HEIGHT },
        length: 1,
        theta: -Math.PI / 2
    }
    step(line);
}
function step(branch: Line,depth=0) {
    const end = getEndPoint(branch);
    drawLine(branch);
    if (depth<4 || Math.random() < 0.5) {
        pendingTask.push(() => {
            step({
                start: end,
                length: branch.length +(Math.random()*5),
                theta: branch.theta - 0.4 *Math.random()
            },depth+1)
        })
    }
    if (depth<4 ||Math.random() < 0.5) {
        pendingTask.push(() => {
            step({
                start: end,
                length: branch.length+(Math.random()*5 ),
                theta: branch.theta + 0.4 *Math.random()
            },depth+1)
        })
    }
}
function frame() {
    //浅拷贝 把任务 堆到下一个
    const tasks = [...pendingTask];
    pendingTask.length = 0;
    tasks.forEach(fn => fn());
}
let frameCounts = 0;
function startAnime() {
    requestAnimationFrame(() => {
        frameCounts += 1;
        if (frameCounts % 10 == 0)
            frame();
        startAnime()
    });
}
startAnime();
function lineTo(p1: Point, p2: Point) {
    ctx.beginPath();
    ctx.moveTo(p1.x, p1.y);
    ctx.lineTo(p2.x, p2.y);
    ctx.stroke();
}
function drawLine(l: Line) {
    lineTo(l.start, getEndPoint(l));
}
function getEndPoint(l: Line) {
    return {
        x: l.start.x + l.length * Math.cos(l.theta),
        y: l.start.y + l.length * Math.sin(l.theta),
    }
}
onMounted(() => {
    init();
})
</script>
