<template lang="">
    <div class="cell button-30" role="button" 
        :class="cellClassObject" 
        :style="{background: cellState.bgColor, color: cellFontColor }"
        @mousedown.left="mouseDown" 
        @mouseleave.left="mouseLeave" 
        @click="leftClick" 
        @contextmenu.prevent="rightClick" 
        > 
        {{cellValue}}
    </div>
</template>
<script>
import {computed, reactive, watch} from "vue";

export default {
    props:{
        gridRow:{
            type: Number,
            required: true
        },
        gridCol:{
            type: Number,
            required: true
        },
        value:{
            type: Number
        },
        cellFlipped:{
            type: Number
        },
        gameStatus:{
            type: Number
        }
    },
    emits: ["leftClick", "rightClick"],
    setup(props, {emit}){
        const cellState = reactive({
            mouseLeftClicked: false,
            mouseRightClicked: false,
            mouseLeftDown: false,
            bgColor: "",
        })

        const cellValue = computed(() => {
            if (!cellShow.value){
                return ""
            }else if (cellState.mouseRightClicked){
                return "🚩"
            }else if (props.value === -1){
                return "💣"
            }else if (props.value === 0){
                return ""
            }else{
                return props.value
            }
        })

        const cellFontColor = computed(() => {
            switch (props.value){
                case (1):
                   return "blue"
                case(2):
                   return "green"
                case(3):
                   return "red"
                case(4):
                   return "darkblue"
                case(5):
                   return "brown"
                case(6):
                   return "purple"
                default:
                    return ""
            }
        })

        const cellClassObject = computed(() => ({
            // To prevent if cell is clicked but mouseLeave make it pop back
            'cell-left-clicked': cellState.mouseLeftClicked || cellState.mouseLeftDown , 
        }))

        watch(() => props.gameStatus, (gameStatus) => {
            if (gameStatus === -1){
                // Game reset
                cellState.mouseLeftClicked = false
                cellState.mouseRightClicked = false
                cellState.mouseLeftDown = false
                cellState.bgColor = ""
            }else if(gameStatus >= 1){
                // Game end
                if (cellState.mouseRightClicked && props.value !== -1){
                    cellState.bgColor = "rgba(247, 135, 135, 0.53)"
                }
            }

        })

        const checkClickable = (fn) => {
            return function(){
                if((props.gameStatus >= 1) || props.cellFlipped || cellState.mouseRightClicked){
                    return false
                }
                fn()
            }
        }

        const mouseDown = checkClickable(() => {
            cellState.mouseLeftDown = true
        })

        const mouseLeave = checkClickable(() => {
            cellState.mouseLeftDown = false
        })

        const leftClick = checkClickable(() => {
            cellState.mouseLeftClicked = true
            const clickBomb = props.value === -1 ? true: false
            emit("leftClick", props.gridRow-1, props.gridCol-1, clickBomb)

            if (clickBomb){
                cellState.bgColor = "red"
            }
        })
        
        const rightClick = () => {
            // Disable any click
            if((props.gameStatus >= 1) || props.cellFlipped){
                return false
            }

            cellState.mouseRightClicked = !cellState.mouseRightClicked
            emit("rightClick", cellState.mouseRightClicked)
        }

        const cellShow = computed(() => {
            if (props.cellFlipped === 1){
                if (props.value !== -1){ 
                    // reveal when it is actually a number, no matter it is flagged or not
                    cellState.mouseLeftClicked = true
                    cellState.mouseRightClicked = false
                }else if (props.value === -1 && !cellState.mouseRightClicked){
                    // if it is bomb and not flagged, reveal bomb
                    cellState.mouseLeftClicked = true
                }
                // if it is bomb and flagged then remain the same
            }

            return props.cellFlipped || cellState.mouseLeftClicked || cellState.mouseRightClicked
        })

        return{
            leftClick,
            rightClick,
            cellState,
            cellFontColor,
            cellValue,
            cellClassObject,
            mouseDown,
            mouseLeave
        }
    }
}
</script>
<style lang="scss" scoped>
@import '@/assets/scss/button.scss';

.cell{
    grid-row: v-bind(gridRow) / calc(v-bind(gridRow) + 1);
    grid-column: v-bind(gridCol) / calc(v-bind(gridCol) + 1);
    font-weight: 900;

}

.cell-left-clicked {
  background-color: #fcfcfdaf;

  box-shadow: #b6b6b673 0 3px 7px inset;
  transform: translateY(2px);
}

</style>