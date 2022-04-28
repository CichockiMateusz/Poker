<template>
    <div>
        <!-- clubs (♣), diamonds (♦), hearts (♥) and spades (♠) !-->
        <div class="grid grid-cols-2">
            <card v-if="debug" @some-event="(n) => addToHand(n)" :deck="deck"/>
        </div>
        <div>
            <div>
                <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="drawToFullHand">Dobierz do 5 kart</button>
                <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="estimateHand">Sprawdź rękę</button>
                <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="swapCards">Wymień rękę</button><br>
            </div>
            <span>Twoja ręka</span><br>
            <card @some-event="(n) => echo(n)" :deck="players[0].hand" />
        </div>
    </div>
</template>

<script>
import card from './components/Card.vue'
import axios from 'axios'
export default {
    data() {
        return {
            deck: [],
            players:[
                {
                    hand:[],
                    saldo:0,
                    wasHandSwapped: false
                }
            ],
            debug: false
        }
    },
    created(){
        this.fillDeck()
        this.apiCall()
        this.debugCheck()
    },
    methods: {
        echo(z){
            if((this.players[0].hand[z].selected == undefined)||(this.players[0].hand[z].selected == false)) this.players[0].hand[z].selected = true
            else this.players[0].hand[z].selected = false
        },
        addToHand(z){
            if(this.players[0].hand.length<5){
                this.players[0].hand.push(this.deck[z])
                this.deck.splice(z,1)
            }
            this.sortHand(0)
        },
        swapCards(){
            if(this.players[0].wasHandSwapped==false){
                for(let i=0; i<=4; i++){
                    if(this.players[0].hand[i].selected == true){
                        this.players[0].hand.splice(i,1)
                        this.drawCard()
                        i--
                    }
                }
                this.players[0].wasHandSwapped=true
                this.sortHand(0)
            }
            
        },
        fillDeck(){
            for(let i=0; i<4; i++){
                let isRed;
                let symbol;
                switch (i){
                    case 0:
                        symbol = "♦"
                        isRed = true
                        break;
                    case 1:
                        symbol = "♥"
                        isRed = true
                        break;
                    case 2:
                        symbol = "♣"
                        isRed = false
                        break;
                    case 3:
                        symbol = "♠"
                        isRed = false
                        break;
                    default:
                        break;
                }
                for(let j=2; j<=10; j++){
                    this.deck.push({
                        character: j,
                        value: j,
                        symbol: symbol,
                        isRed: isRed,
                    })
                }
                this.deck.push({
                    character:'J',
                    value: 11,
                    symbol: symbol,
                    isRed: isRed,
                })
                this.deck.push({
                    character:'Q',
                    value: 12,
                    symbol: symbol,
                    isRed: isRed,
                })
                this.deck.push({
                    character:'K',
                    value: 13,
                    symbol: symbol,
                    isRed: isRed,
                })
                this.deck.push({
                    character:'A',
                    value: 14,
                    symbol: symbol,
                    isRed: isRed,
                })
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
        drawToFullHand(){
            while(this.players[0].hand.length<5){
                this.drawCard()
            }
            this.sortHand(0)
        },
        estimateHand(event, playerID = 0){
            let values = []
            let symbols = []
            this.players[playerID].hand.forEach(element => {
                values.push(element.value)
                symbols.push(element.symbol)
            });
            values.sort(function(a, b){return a - b})
            symbols.sort(function(a, b){return a - b})
            console.log(values)
            console.log(symbols)

            let isFlush = this.checkForFlush(symbols)
            let isStright = this.checkForStraight(values)
            let isFourOfAKind = this.checkForFourOfAKind(values)
            let isThreeOfAKind = null
            let isAPair = null
            let isASecondPair = null
            if(!isFourOfAKind){
                isThreeOfAKind = this.checkForThreeOfAKind(values)
                if(isThreeOfAKind){
                    isAPair = this.checkForAPair(values,isThreeOfAKind)
                }
                else{
                    isAPair = this.checkForAPair(values)
                    if(isAPair){
                        isASecondPair = this.checkForAPair(values,isAPair)
                    }
                }
            }

            if(isFlush&&isStright&&values[4]==14){
                alert("Royal flush")
            }
            else if(isFlush&&isStright){
                alert("Straight flush")
            }
            else if(isFourOfAKind){
                alert("Four of a kind")
            }
            else if(isThreeOfAKind&&isAPair){
                alert("Full house")
            }
            else if(isFlush){
                alert("Flush")
            }
            else if(isStright){
                alert("Straight")
            }
            else if(isThreeOfAKind){
                alert("Three of a kind")
            }
            else if(isASecondPair){
                alert("Two pair")
            }
            else if(isAPair){
                alert("Pair")
            }
            else{
                alert("High card " + this.players[0].hand[4].character)
            }
            // console.log("DEBUG: Is hand four of a kind?:" + this.checkForFourOfAKind(values))
            // console.log("DEBUG: Is hand three of a kind?:" + isThreeOfAKind)
            // console.log("DEBUG: Is hand two of a kind?:" + this.checkForAPair(values))
            // console.log("DEBUG: Is hand two of a kind?:" + this.checkForAPair(values))
            // console.log("DEBUG: Is hand flush?:" + this.checkForFlush(symbols))
            // console.log("DEBUG: Is hand straight?:" + this.checkForStraight(values))
        },
        checkForFlush(symbols){
            let firstCard = symbols[0]
            for(let i=1; i<5; i++){
                if(symbols[i] != firstCard){
                    return false
                }
            }
            return true
        },
        checkForStraight(values){
            for(let i=1; i<5; i++){
                if(values[0]+i != values[i]){
                    return false
                }
            }
            return true
        },
        checkForFourOfAKind(values){
            if((values[0] == values[1]) && (values[1] == values[2]) && (values[2] == values[3])) return values[0]
            if((values[1] == values[2]) && (values[2] == values[3]) && (values[3] == values[4])) return values[1]
            return false
        },
        checkForThreeOfAKind(values){
            if((values[0] == values[1]) && (values[1] == values[2])) return values[0]
            if((values[1] == values[2]) && (values[2] == values[3])) return values[1]
            if((values[2] == values[3]) && (values[3] == values[4])) return values[2]
            return false
        },
        checkForAPair(values,ignored = 0){
            for(let i=0; i<=3; i++){
                if((values[i] == values[i+1]) && (values[i] != ignored)) return values[i]
            }
            return false
        },
        getRandomInt(min, max) {
            min = Math.ceil(min);
            max = Math.floor(max);
            return Math.floor(Math.random() * (max - min)) + min;
        },
        apiCall(){
            axios
                .get('https://api.coindesk.com/v1/bpi/currentprice.json')
                .then(response => (console.log(response)))
        },
        debugCheck(){
            let uri = window.location.search.substring(1); 
            let params = new URLSearchParams(uri);
            if(params.get("debug")=='true'){
                this.debug=true
            }
            console.log(params.get("debug"));
        },
        sortHand(playerId){
            this.players[playerId].hand.sort((a,b) =>{
                return a.value - b.value
            })
        }
    },
    components:{
        card
    }
}
</script>
<style lang="">
    
</style>