<template >
    <div id="reset" class="button-30" role="button" @click="$emit('reset')">
        {{emoji}}
    </div>
</template>
<script>
import {computed} from "vue";
export default {
    props:{
        gameStatus: {
            type: Number
        }
    },
    emits:["reset"],
    setup(props){
        const emoji = computed(() => {
            if (props.gameStatus === 1){
                return "💀"
            }else if(props.gameStatus === 2){
                return "🤩"
            }else{
                return "😃"
            }
        })


        return {
            emoji
        }
    }
}
</script>
<style lang="scss" scoped>
@import '@/assets/scss/button.scss';
#reset{
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    width: 50px;
    height: 50px;

    font-size: 40px;
}

.button-30:hover {
  box-shadow: rgba(45, 35, 66, 0.4) 0 4px 8px, rgba(45, 35, 66, 0.3) 0 7px 13px -3px, #D6D6E7 0 -3px 0 inset;
  transform: translateY(-2px);
}

.button-30:active {
  box-shadow: #D6D6E7 0 3px 7px inset;
  transform: translateY(2px);
}
    
</style>