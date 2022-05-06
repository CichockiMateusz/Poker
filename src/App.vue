<template>
    <div>
        <!-- clubs (♣), diamonds (♦), hearts (♥) and spades (♠) !-->
        <span>Ręka przeciwnika</span><br>
        <card v-if="players[1].handHidden==false" :deck="players[1].hand" />
        <card-hidden v-if="players[1].handHidden==true" :deck="players[1].hand" />
        <div class="grid grid-cols-2">
            <card v-if="debug" @some-event="(n) => addToHand(n)" :deck="deck"/>
        </div>
        <div>
            <div>
                <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="checkHandButtonEvent" v-if="!gameHasEnded">Sprawdź rękę</button>
                <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="swapCards" v-show="!players[0].wasHandSwapped && !gameHasEnded">Wymień rękę</button><br>
                <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="newGame" v-show="gameHasEnded">Nowa gra</button>
            </div>
            <span>Twoja ręka</span><br>
            <card @some-event="(n) => echo(n)" :deck="players[0].hand" />
        </div>
        <span>Łączna ilość wygranych </span>
        <span>{{ wins }}</span><br><br><br>
        <span v-if="players[0].estimation!=''">Gracz pierwszy ma: {{ players[0].estimation }}</span><br>
        <span v-if="players[1].estimation!=''">Gracz drugi ma: {{ players[1].estimation }}</span><br>
        <span class="text-lg">{{ message }}</span>
    </div>
</template>

<script>
import card from './components/Card.vue'
import cardHidden from './components/CardHidden.vue'
import axios from 'axios'
import VueCookies from 'vue-cookies'
export default {
    data() {
        return {
            deck: [],
            players:[
                {
                    hand:[],
                    wasHandSwapped: false,
                    power:{
                        ranking:-1,
                        card:-1
                    },
                    estimation: ""
                },
                {
                    hand:[],
                    wasHandSwapped: false,
                    handHidden: true,
                    power:{
                        ranking:-1,
                        card:-1
                    },
                    estimation: ""
                }
            ],
            debug: false,
            wins: 0,
            gameHasEnded: false,
            message: ""
        }
    },
    created(){
        this.fillDeck()
        this.apiCall()
        this.debugCheck()
        this.players.forEach((player, index) => {
            this.drawToFullHand(index)
        });
        this.checkIfCookiesExist()
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
        newGame(){
            this.deck = []
            this.players[0].hand = []
            this.players[1].hand = []
            this.fillDeck()
            this.players.forEach((player, index) => {
                this.drawToFullHand(index)
            });
            this.message = ""
            this.players[0].estimation = ""
            this.players[1].estimation = ""
            this.gameHasEnded = false
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
        drawCard(playerID = 0){
            let cardNum = this.getRandomInt(0,this.deck.length)
            if(this.deck[cardNum]==null){
                console.log(cardNum)
            }
            this.players[playerID].hand.push(this.deck[cardNum])
            this.deck.splice(cardNum,1)
        },
        drawToFullHand(playerID = 0){
            while(this.players[playerID].hand.length<5){
                this.drawCard(playerID)
            }
            this.sortHand(playerID)
        },
        checkHandButtonEvent(){
            this.estimateHand(null,0)
            this.estimateHand(null,1)
            this.confrontHands(0,1)
            this.players[1].handHidden = false
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
                this.players[playerID].estimation = "Royal flush"
                this.assignPower(playerID, 9, 14)
            }
            else if(isFlush&&isStright){
                this.players[playerID].estimation = "Straight flush"
                this.assignPower(playerID, 8, this.players[playerID].hand[4].value)
            }
            else if(isFourOfAKind){
                this.players[playerID].estimation = "Four of a kind"
                this.assignPower(playerID, 7, isFourOfAKind)
            }
            else if(isThreeOfAKind&&isAPair){
                this.players[playerID].estimation = "Full house"
                this.assignPower(playerID, 6, isThreeOfAKind)
            }
            else if(isFlush){
                this.players[playerID].estimation = "Flush"
                this.assignPower(playerID, 5, this.players[playerID].hand[4].value)
            }
            else if(isStright){
                this.players[playerID].estimation = "Straight"
                this.assignPower(playerID, 4, this.players[playerID].hand[4].value)
            }
            else if(isThreeOfAKind){
                this.players[playerID].estimation = "Three of a kind"
                this.assignPower(playerID, 3, isThreeOfAKind)
            }
            else if(isASecondPair){
                this.players[playerID].estimation = "Two pair"
                this.assignPower(playerID, 2, isASecondPair)
            }
            else if(isAPair){
                this.players[playerID].estimation = "Pair"
                this.assignPower(playerID, 1, isAPair)
            }
            else{
                this.players[playerID].estimation = "High card " + this.players[playerID].hand[4].character
                this.assignPower(playerID, 0, this.players[playerID].hand[4].value)
            }
            // console.log("DEBUG: Is hand four of a kind?:" + this.checkForFourOfAKind(values))
            // console.log("DEBUG: Is hand three of a kind?:" + isThreeOfAKind)
            // console.log("DEBUG: Is hand two of a kind?:" + this.checkForAPair(values))
            // console.log("DEBUG: Is hand two of a kind?:" + this.checkForAPair(values))
            // console.log("DEBUG: Is hand flush?:" + this.checkForFlush(symbols))
            // console.log("DEBUG: Is hand straight?:" + this.checkForStraight(values))
        },
        confrontHands(firstPlayer, secondPlayer){
            if(this.players[firstPlayer].power.ranking>this.players[secondPlayer].power.ranking){
                this.message = "Wygrał gracz pierwszy"
                this.addWinToCookies()
                this.gameHasEnded = true
            }
            else if(this.players[firstPlayer].power.ranking<this.players[secondPlayer].power.ranking){
                this.message = "Wygrał gracz drugi"
                this.gameHasEnded = true
            }
            else if(this.players[firstPlayer].power.ranking == this.players[secondPlayer].power.ranking){
                if(this.players[firstPlayer].power.card > this.players[secondPlayer].power.card){
                    this.message = "Wygrał gracz pierwszy"
                    this.addWinToCookies()
                    this.gameHasEnded = true
                }
                else if(this.players[firstPlayer].power.card < this.players[secondPlayer].power.card){
                    this.message = "Wygrał gracz drugi"
                    this.gameHasEnded = true
                }
                else for(let i=4; i>=0; i--){
                        if(this.players[firstPlayer].hand[i].value>this.players[secondPlayer].hand[i].value){
                            this.message = "Wygrał gracz pierwszy"
                            this.addWinToCookies()
                            this.gameHasEnded = true
                            i=-1
                        }
                        else if(this.players[firstPlayer].hand[i].value<this.players[secondPlayer].hand[i].value){
                            this.message = "Wygrał gracz drugi"
                            this.gameHasEnded = true
                            i=-1
                        }
                        if((i==0)&&(this.players[firstPlayer].hand[i].value == this.players[secondPlayer].hand[i].value)){
                            this.message = "Remis"
                            this.gameHasEnded = true
                            i=-1
                        }
                }
            } 
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
        },
        assignPower(playerID = 0, ranking, card){
            this.players[playerID].power.ranking = ranking
            this.players[playerID].power.card = card
        },
        checkIfCookiesExist(){
            if($cookies.get('wins')==null){
                $cookies.set('wins',0)
            }
            this.wins = $cookies.get('wins')
        },
        addWinToCookies(){
            $cookies.set('wins', parseInt($cookies.get('wins'))+1)
            this.wins = $cookies.get('wins')
        }
    },
    components:{
        card,
        cardHidden
    }
}
</script>
<style lang="">
    
</style>