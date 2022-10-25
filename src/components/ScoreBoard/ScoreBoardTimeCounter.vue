<template >
    <div id="time-counter">
        {{time}}
    </div>
</template>
<script>
import { onUnmounted, ref, watch } from '@vue/runtime-core'
export default {
    props:{
        gameStatus:{
            type: Number
        }
    },
    setup(props){
        const time = ref(0)
        let timer
        const activate = () => {
            if (!timer){
                // !PROBLEM: Not sure why the first second always delays alot
                timer = window.setInterval(() => {
                    time.value++
                }, 1000)
            }
        }

        watch(() => props.gameStatus, (newStatus) => {
            if (newStatus === 0){
                // Game state 0: The game is now playable and the user start playing
                // => start the timer
                activate()
            }else if (newStatus === -1){
                // Game state -1: The game is now playable, but not yet been started
                time.value = 0
                window.clearInterval(timer)
                timer = null
            }else{
                // Game state 1, 2: The game is stopped
                window.clearInterval(timer)
                timer = null
            }
        })

        onUnmounted(() => {
            window.clearInterval(timer)
        })


        return{
            time
        }
    }
}
</script>
<style lang="scss">
#time-counter{
    position: absolute;
    right: 10px;
    top: 50%;
    transform: translate(0, -50%);
    
    width: 100px;
    height: 50px;

    border-radius: 4px;
    border-width: 0;
    background-color: #FCFCFD;

    font-size: 40px;
    box-shadow: rgba(0, 0, 0, 0.18) 0px 2px 4px;
}
</style>