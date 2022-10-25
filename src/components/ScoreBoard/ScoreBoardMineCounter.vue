<template lang="">
    <div id="mine-counter">
        {{mineCount}}
    </div>
</template>
<script>
export default {
    props:{
        mineCount: {
            type: Number
        }
    },
    setup(){

    }
}
</script>
<style lang="scss">
#mine-counter{
    position: absolute;
    left: 10px;
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