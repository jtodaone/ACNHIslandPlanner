<template lang="html">
    <main>
        <div id="canvas">

        </div>
        <div class="toolbar">
            <a @click="mode = 'terrain'">
                <img src="./terrain.png" alt="">
                <span>EDIT TERRAIN</span>
            </a>
            <a @click="mode = 'entity'">
                <img src="./object.png" alt="">
                <span>EDIT OBJECTS</span>
            </a>
            <div v-show="mode == 'terrain'">
                <a :class="{ 'active' : currentOperation == brushes.water }" @click="currentOperation = brushes.water">WATER</a>
                <a :class="{ 'active' : currentOperation == brushes.sand }" @click="currentOperation = brushes.sand">SAND</a>
                <a :class="{ 'active' : currentOperation == brushes.land }" @click="currentOperation = brushes.land">LAND</a>
                <input type="number" v-show="currentOperation == brushes.land" v-model="currentElevation" :max="maxElevation"></input>
                <a :class="{ 'active' : currentOperation == brushes.path }" @click="currentOperation = brushes.path">PATH</a>
            </div>
            <div v-show="mode == 'terrain'">
                Brush Size
                <input type="number" v-model="brushSize">
            </div>
        </div>
    </main>
</template>

<script>
import p5 from 'p5'

const acreBlocks = 16

const columns = acreBlocks * 7
const rows = acreBlocks * 6

const landMargin = 13
const beachWidth = 8

let ppb = 6

const lineDotLength = 0.5

let colorScheme = {
    land: []
}

function newIsland() {
    let result = new Array(columns)

    for (let x = 0; x < columns; x++) {
        result[x] = new Array(rows)

        for (let y = 0; y < rows; y++) {
            if (isInLandBoundary(x, y)) {
                result[x][y] = {
                    type: 'land',
                    elevation: 1
                }
            } else if (isInBeachBoundary(x, y)) {
                result[x][y] = {
                    type: 'sand',
                    elevation: 1
                }
            } else {
                result[x][y] = {
                    type: 'water',
                    elevation: 1
                }
            }
        }
    }

    return {
        name: '',
        author: '',
        map: result,
        entities: [{
            x: acreBlocks + Math.floor(Math.random() * (columns - acreBlocks * 2 - 12)),
            y: acreBlocks + Math.floor(Math.random() *  (rows - acreBlocks * 2 - 10)),
            width: 12,
            height: 10,
            name: 'Residential Service Centre'
        }]
    };
}

function isInLandBoundary(x, y) {
    return x >= landMargin && y >= landMargin && x < columns - landMargin && y < rows - landMargin;
}

function isInBeachBoundary(x, y) {
    return x >= landMargin - beachWidth && y >= landMargin - beachWidth && x < columns - (landMargin - beachWidth) && y < rows - (landMargin - beachWidth);
}

function isInBeach(x, y) {
    return isInBeachBoundary(x, y) && !isInLandBoundary(x, y);
}

export default {
    data() {
        return {
            island: {},
            currentOperation: () => {},
            currentElevation: 1,
            currentEntity: null,
            currentColor: null,
            maxElevation: 3,
            brushSize: 1,
            mode: 'terrain',
            brushes: {
                land(cell) {
                    return { ...cell,
                        type: 'land',
                        elevation: this.currentElevation
                    }
                },
                sand(cell) {
                    return { ...cell,
                        type: 'sand'
                    }
                },
                water(cell) {
                    return { ...cell,
                        type: 'water'
                    }
                },
                path(cell) {
                    return { ...cell,
                        type: 'path',
                        color: this.currentColor
                    }
                }
            },
            sketch: (p) => {
                p.setup = () => {
                    ppb = document.getElementById('canvas').offsetWidth / columns
                    p.createCanvas(columns * ppb, rows * ppb)

                    colorScheme['sand'] = p.color('#EFE7B4')

                    colorScheme['land'][1] = p.color('#58824D')
                    colorScheme['land'][2] = p.color('#65A755')
                    colorScheme['land'][3] = p.color('#83C963')

                    colorScheme['water'] = p.color('#9AD9CB')

                    colorScheme['default'] = p.color('#B7AB8D')

                    this.currentColor = colorScheme['default']

                    this.island = newIsland()
                }

                p.windowResized = () => {
                    ppb = document.getElementById('canvas').offsetWidth / columns
                    p.resizeCanvas(columns * ppb, rows * ppb)
                }

                p.draw = () => {
                    p.noStroke()
                    p.blendMode(p.BLEND)

                    for (let x = 0; x < columns; x++) {
                        for (let y = 0; y < rows; y++) {
                            switch (this.island.map[x][y].type) {
                                case 'water':
                                    p.fill(colorScheme.water)
                                    break
                                case 'land':
                                    p.fill(colorScheme.land[this.island.map[x][y].elevation])
                                    break
                                case 'sand':
                                    p.fill(colorScheme.sand)
                                    break
                                case 'path':
                                    p.fill(this.island.map[x][y].color || colorScheme.default)
                                    break
                                default:
                                    p.fill(colorScheme.default)
                            }
                            p.square(x * ppb, y * ppb, ppb)
                        }
                    }

                    if (this.mode == 'terrain') {
                        p.blendMode(p.OVERLAY)
                    }

                    for (let entity of this.island.entities) {
                        p.fill(entity.color || colorScheme.default)
                        p.rect(entity.x * ppb, entity.y * ppb, entity.width * ppb, entity.height * ppb, ppb)
                    }

                    p.blendMode(p.OVERLAY)

                    p.stroke(255, 127)
                    p.strokeWeight(1.5);

                    for (let x = acreBlocks; x < columns; x += acreBlocks) {
                        for (let y = ppb * lineDotLength; y < p.height; y += ppb * lineDotLength * 3) {
                            p.line(x * ppb, y, x * ppb, y + ppb * lineDotLength);
                        }
                    }

                    for (let y = acreBlocks; y < rows; y += acreBlocks) {
                        for (let x = ppb * lineDotLength; x < p.width; x += ppb * lineDotLength * 3) {
                            p.line(x, y * ppb, x + ppb * lineDotLength, y * ppb);
                        }
                    }
                }

                p.mousePressed = () => {
                    if (this.mode == 'entity') {
                        let x = p.floor(p.mouseX / ppb)
                        let y = p.floor(p.mouseY / ppb)

                        this.currentEntity = this.island.entities.find(entity => {
                            return x < entity.x + entity.width && x > entity.x && y < entity.y + entity.height && y > entity.y
                        })
                    }
                }

                p.mouseDragged = () => {
                    let x = p.floor(p.mouseX / ppb)
                    let y = p.floor(p.mouseY / ppb)
                    let prevX = p.floor(p.pmouseX / ppb)
                    let prevY = p.floor(p.pmouseY / ppb)

                    if ((prevX != x || prevY != y)) {
                        switch (this.mode) {
                            case 'terrain':
                                if (this.brushSize > 1) {
                                    for (let l = p.round(x - this.brushSize / 2); l < x + this.brushSize / 2; l++) {
                                        for (let m = p.round(y - this.brushSize / 2); m < y + this.brushSize / 2; m++) {
                                            if (!(l < 0 || l >= columns || m < 0 || m >= rows)) this.island.map[l][m] = this.currentOperation(this.island.map[l][m])
                                        }
                                    }
                                } else this.island.map[x][y] = this.currentOperation(this.island.map[x][y])
                                break
                            case 'entity':
                                if (this.currentEntity) {
                                    this.currentEntity.x = x - p.ceil(this.currentEntity.width / 2)
                                    this.currentEntity.y = y - p.ceil(this.currentEntity.height / 2)
                                }
                                break
                        }
                    }
                }

                p.mouseReleased = () => {
                    if (this.mode == 'entity' && this.currentEntity) {
                        this.currentEntity = null
                    }
                }
            }
        }
    },
    mounted() {
        const canvas = new p5(this.sketch, document.querySelector('#canvas'))
        this.currentOperation = this.brushes.land
    }
}
</script>

<style lang="scss" scoped>
    main {
        display: flex;
        flex-direction: column;
        align-items: center;

        #canvas {
            width: 50vw;
            overflow: hidden;
        }

        .toolbar {
            display: flex;
            a {
                display: flex;
                flex-direction: column;
                align-items: center;
            }
        }
    }
</style>
