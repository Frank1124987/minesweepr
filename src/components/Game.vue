<template>
    <div>
        <div id="score-board">
            <ScoreBoardMineCounter
                :mineCount = "gameState.mineCount"
            />
            <ScoreBoardReset
                @reset="reset"
                :gameStatus="gameState.status"
            />
            <ScoreBoardTimeCounter
                :gameStatus="gameState.status"
            />
        </div>
        <div id="game">
            <!-- 16 * 16 -->
            <template v-for="(x, key1) in gameSetting.cellRow" >
                <GameCell v-for="(y, key2) in gameSetting.cellCol" :key="key1 +'-'+ key2"
                    :gridRow="x"
                    :gridCol="y"
                    :value="gameState.board[x-1][y-1]"
                    @leftClick="leftClick"
                    @rightClick="rightClick"
                    :cellFlipped="gameState.boardState[x-1][y-1]"
                    :gameStatus = "gameState.status"
                />
            </template>
        </div>
    </div>  
</template>
<script>
import ScoreBoardMineCounter from "@/components/scoreBoard/ScoreBoardMineCounter.vue";
import ScoreBoardTimeCounter from "@/components/scoreBoard/ScoreBoardTimeCounter.vue";
import ScoreBoardReset from "@/components/scoreBoard/ScoreBoardReset.vue";
import GameCell from "@/components/GameCell.vue";
import {ref, reactive} from "vue";

export default {
    components:{
        GameCell,
        ScoreBoardMineCounter,
        ScoreBoardTimeCounter,
        ScoreBoardReset
    },
    setup(){
        const gameSetting = reactive({
            cellSize: 25,
            cellRow: 16,
            cellCol: 16,
            mineNum: 40,
        })

        const cellSizeCss = ref(gameSetting.cellSize + "px")
        const gameWidth = ref(`${gameSetting.cellSize * gameSetting.cellCol}px`)
        const gameHeight = ref(`${gameSetting.cellSize * gameSetting.cellRow}px`)

        const gameState = reactive({
            // -1: mine, 0,1,2... : number of mine around the cell
            board: Array.from(Array(gameSetting.cellRow), _ => Array(gameSetting.cellCol).fill(0)),

            // 0: not flipped, 1: flipped
            boardState: Array.from(Array(gameSetting.cellRow), _ => Array(gameSetting.cellCol).fill(0)),

            // Record all mines' position
            minePos: [],

            // -1 : The game is now playable, but not yet been started
            //  0 : The game is now playable and the user start playing
            //  1 : The game is stopped, you lost
            //  2 : The game is stopped, you win
            status: -1,

            cellOpened: 0,
            mineCount: 40
        })

        const reset = () =>{
            gameState.board.map(row => {row.fill(0)})
            gameState.boardState.map(row => {row.fill(0)})
            gameState.minePos.length = 0
            gameState.status = -1
            gameState.cellOpened = 0
            gameState.mineCount = 40
        }

        const rightClick = (flagged) => {
            // Generate the game board when first clicked
            if (gameState.status === -1){
                generateGameBoard(x, y)
                gameState.status = 0
            }
            gameState.mineCount = flagged ? gameState.mineCount - 1 : gameState.mineCount + 1
        }

        const leftClick = (x, y, clickBomb) => {
            // Generate the game board when first clicked
            if (gameState.status === -1){
                generateGameBoard(x, y)
                gameState.status = 0
            }

            // Detect if game ended
            // 1. Click bomb
            if (clickBomb){
                gameState.minePos.forEach(([x, y]) => {
                    gameState.boardState[x][y] = 1
                })

                gameState.status = 1
                return
            }
            // 2. All cell other than bomb is opened
            cellReveal(x, y)
            gameState.cellOpened = gameState.boardState.map(row => row.filter(e => e===0).length).reduce((accumulator, value) => accumulator + value)
            if (gameState.cellOpened === gameSetting.mineNum){
                
                gameState.status = 2
                return
            }
        }

        // Random Generate 40 Bomb
        const generateGameBoard = (click_x, click_y) => {
            for(let i = 0 ; i < gameSetting.mineNum ; i++){
                const pos_x = Math.floor(Math.random() * gameSetting.cellRow)
                const pos_y = Math.floor(Math.random() * gameSetting.cellCol)
                
                // Make sure no repeated bomb or bomb at first click cell
                if (gameState.board[pos_x][pos_y] === -1 || (pos_x === click_x && pos_y === click_y)){
                    i--
                    continue
                }
                gameState.board[pos_x][pos_y] = -1
                gameState.minePos.push([pos_x, pos_y])
            }

            // calculate every cell's value, search bomb and add bomb near value
            gameBoardCalcVal()
        }


        const gameBoardCalcVal = () => {
            gameState.minePos.forEach(([x, y]) => {
                for(let i = -1 ; i <= 1 ; i++){
                    for(let j = -1 ; j<=1 ; j++){
                        // Don't consider the cell itself
                        if( i===0 && j===0){
                            continue
                        }

                        let temp_x = x + i
                        let temp_y = y + j
                        // If the surrounding cell is bomb or out of bound
                        if(!(temp_x >= 0 && temp_x < gameSetting.cellRow && temp_y >= 0 && temp_y < gameSetting.cellCol ) || gameState.board[temp_x][temp_y] === -1){
                            continue
                        }
                        gameState.board[temp_x][temp_y]++
                    }
                }
            })
        }

        const cellReveal = (x, y) => {
            // BFS
            //  check if value is 0 (no near by mine) 
            //      if false, then simply reveal the cell
            //      if true, reveal and then push neighbor to stack so that it will keep searching
            //          push cell only if it is not flipped
            //          reveal the one that is flagged

            const stack = [[x, y]]
            while(stack.length){
                const current = stack.shift()

                gameState.boardState[current[0]][current[1]] = 1
                if (gameState.board[current[0]][current[1]] === 0){

                    for(let i = -1 ; i <= 1 ; i++){
                        for(let j = -1 ; j<=1 ; j++){
                            // Don't consider the cell itself
                            if( i===0 && j===0){
                                continue
                            }

                            let temp_x = current[0] + i
                            let temp_y = current[1] + j
                            // If the surrounding cell is out of bound or already flipped
                            if(!(temp_x >= 0 && temp_x < gameSetting.cellRow && temp_y >= 0 && temp_y < gameSetting.cellCol ) || gameState.boardState[temp_x][temp_y] === 1){
                                continue
                            }
                            
                            stack.push([temp_x, temp_y])
                        }
                    }
                }
            }
        }

        return {
            gameWidth,
            gameHeight,
            gameSetting,
            gameState,
            cellSizeCss,
            leftClick,
            rightClick,
            reset
        }
    }
    
}
</script>
<style lang="scss">
#game{
    margin: 10px;
    width: v-bind(gameWidth);
    height: v-bind(gameHeight);

    display: grid;
    grid-template-columns: repeat(v-bind("gameSetting.cellCol"), v-bind(cellSizeCss) [col]);
    grid-template-rows: repeat(v-bind("gameSetting.cellRow"), v-bind(cellSizeCss) [row]);

    box-shadow: rgba(0, 0, 0, 0.15) 0px 2px 8px;
}

#score-board{
    position: relative;
    height: 70px;
    margin: 10px;
    box-shadow: rgba(0, 0, 0, 0.15) 0px 2px 8px;
}
</style>