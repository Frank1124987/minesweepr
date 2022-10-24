<template>
    <div id="main">
        <Game/>
    </div>
</template>
<script>
import Game from "@/components/Game.vue";

export default {
    components: {
        Game
    },


}
</script>
<style lang="scss">

#main{
    background: rgb(203, 203, 203);
    margin: auto;
    display: inline-block;
}
</style>