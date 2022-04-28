<template lang="">
    <div>
        
        <span v-for="(item, index) in deck" :key="index">{{item}} <br></span>
        <button @click="drawCard">Dobierz kartę</button><br>
        <span>Twoja ręka</span><br>
        <span>{{ players[0].hand }}</span>
        
        

    </div>
</template>
<script>
export default {
    data() {
        return {
            deck: [],
            players:[
                {
                    hand:[],
                    saldo:0
                }
            ]
        }
    },
    created(){
        this.fillDeck()
    },
    methods: {
        fillDeck(){
            for(let i=0; i<4; i++){
                let color;
                let symbol;
                switch (i){
                    case 0:
                        symbol = "karo"
                        color = "red"
                        break;
                    case 1:
                        symbol = "kier"
                        color = "red"
                        break;
                    case 2:
                        symbol = "trefl"
                        color = "black"
                        break;
                    case 3:
                        symbol = "pik"
                        color = "black"
                        break;
                    default:
                        break;
                }
                this.deck.push({
                    number:'Ace',
                    symbol: symbol,
                    color: color,
                })
                for(let j=2; j<=10; j++){
                    this.deck.push({
                        number: j,
                        symbol: symbol,
                        color: color
                    })
                }
            }
        },
        drawCard(){
            let cardNum = this.getRandomInt(0,this.deck.length)
            if(this.deck[cardNum]==null){
                console.log(cardNum)
            }
            this.players[0].hand.push(this.deck[cardNum])
            this.deck.splice(cardNum,1)
        },
        getRandomInt(min, max) {
            min = Math.ceil(min);
            max = Math.floor(max);
            return Math.floor(Math.random() * (max - min)) + min;
        }
    },
}
</script>
<style lang="">
    
</style>