<template>
    <div>
        <div class="columns is-desktop" id="game-puzzle-swap">
            <div class="column">
                <div class="puzzle" :class="{'done':done}" :style="{width:width+'px',height:height+'px',backgroundImage:'url('+image+')',backgroundSize:width+'px '+height+'px'}">
                    <div class="row" v-for="(row,rowIdx) in pzls">
                        <div class="pzl" v-for="(col,colIdx) in row" @click="pzzlClick(rowIdx,colIdx)" :class="{'clicked':col.clicked}" :style="{width:(width/size)+'px',height:(height/size)+'px',backgroundImage:'url('+image+')',backgroundSize:width+'px '+height+'px',backgroundPosition:(col.col*-(width/size))+'px '+(col.row*-(height/size))+'px'}"></div>
                    </div>
                </div>
            </div>
            <div class="column">
                <img class="preview" :src="image" alt="" :style="{width:width+'px',height:height+'px'}">
            </div>
        </div>
        <div class="columns" id="game-puzzle">
            <div class="column has-text-centered timeColumn">
                <b>Time:</b>
                <br />
                {{ time }} s.
            </div>
            <transition name="fade">
                <div class="modal is-active" v-if="modal">
                    <div class="modal-background"></div>
                    <div class="modal-content">
                        <div class="box has-text-centered">
                            <p>Your time is {{ time }} s.</p><br />
                            <br />
                            <button type="button" class="button is-primary" @click="modal = false">Close</button>
                        </div>
                    </div>
                    <button class="modal-close is-large" aria-label="close" @click="modal = false"></button>
                </div>
            </transition>
        </div>
    </div>
</template>

<script>
export default {
    name: 'GamePuzzleSwap',
    data: function(){
        return{
            time: 0,
            timeInterval: null,
            pzls: 0,
            done: false,
            selected: 0,
            width:500,
            height:500,
            modal: false,
        }
    },
    props: {
        image: {
            type: String,
            required: true,
        },
        size: {
            default: 3,
        }
    },
    methods: {
        checkIsInOrder(a){
            return a.every((val, i, arr) => !i || (val >= arr[i - 1]));
        },
        shuffle(a){
            for (let i = a.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [a[i], a[j]] = [a[j], a[i]];
            }
            return a;
        },
        pzzlClick(x,y){
            if(this.selected != 0){ // gdy juz pierwszy wybrany
                // odkliknij
                this.pzls[this.selected.x][this.selected.y].clicked = false;
                // skopiuj do tymczasowego
                let targetPzl = {};
                targetPzl = this.pzls[x][y];
                // wgraj pierwszy w miejsce teraz klikniętego
                this.pzls[x][y] = this.selected.object;
                // wgraj teraz kliknięty w miejsce drugiego
                this.pzls[this.selected.x][this.selected.y] = targetPzl;
                // odznacz kliknięcie
                this.selected = 0;
            } else { // gdy jeszcze nie wybrany
                // zaznacz że kliknięty
                this.pzls[x][y].clicked = true;
                // zainicjuj obiekt
                this.selected = {};
                // współprzędne kafelka
                this.selected.x = x;
                this.selected.y = y;
                // skopiuj obiekt
                this.selected.object = this.pzls[x][y];
            }
            // zamień tablice wielowymiarow na jednowymiarow
            let tmpArr = [];
            for(let x = 0; x < this.size; x++){
                for(let y = 0; y < this.size; y++){
                    tmpArr.push(this.pzls[x][y].id);
                }
            }
            if(this.checkIsInOrder(tmpArr)){
                this.done = true;
                // zawibruj jesli telefon
                // window.navigator.vibrate(800);
                clearInterval(this.timeInterval);
                setTimeout(()=>{
                    this.modal = true;
                },1000);
            }
        },
        changeGameSize() {
            window.onresize = (event) => {
                let bodyWidth = window.innerWidth || document.body.clientWidth;
                if(bodyWidth < 768) {
                    this.width = 300;
                    this.height = 300;
                } else {
                    this.width = 500;
                    this.height = 500;
                }
            };
        }
    },
    created(){
        // zainicjuj tablicę
        this.pzls = [];
        // pętla wierszy
        for(let x = 0; x < this.size; x++){
            // zainicjuj tymczasow tablicę elementów wiersza
            let row = [];
            // pętla kolumn
            for(let y = 0; y < this.size; y++){
                // wypełnij tablicę wiersza danymi o obrazku
                row.push({
                    id: ((x*this.size)+y),
                    row: x,
                    col: y,
                    clicked : false,
                });
            }
            // dodaj tymczasowa listę wiersza do tablicy wszystkich bloków
            this.pzls.push(row);
        }
        // zmiksuj kawałki

        let tmpArr = [];
        // zamień tablice wielowymiarow na jednowymiarow
        for(let x = 0; x < this.size; x++){
            for(let y = 0; y < this.size; y++){
                tmpArr.push(this.pzls[x][y]);
            }
        }
        // mieszaj
        tmpArr = this.shuffle(tmpArr);
        // wyczysc glowna tablice
        this.pzls = [];
        // zamień tablice jednowymiarow na wielowymiarow
        for(let x = 0; x < this.size; x++){
            // zainicjuj tymczasow tablicę elementów wiersza
            let row = [];
            // pętla kolumn
            for(let y = 0; y < this.size; y++){
                row.push(tmpArr[(x*this.size)+y]);
            }
            // dodaj tymczasowa listę wiersza do tablicy wszystkich bloków
            this.pzls.push(row);
        }
        // rozpocznij liczenie czasu
        this.timeInterval = setInterval(()=>{
            this.time++;
        },1000);
        // set initial game width and height
        if((window.innerWidth || document.body.clientWidth) < 768) {
            this.width = 300;
            this.height = 300;
        }
        // changing the game widh and height on resize
        this.changeGameSize();
    }
}
</script>

<style lang="less">
.fade-enter-active, .fade-leave-active {
    transition: opacity .5s
}
.fade-enter, .fade-leave-to{
    opacity: 0
}
#game-puzzle-swap{
    text-align: center;
    .preview{
        display: inline-block;
        border:1px solid #000;
    }
    .puzzle{
        display: inline-block;
        background-position: -1000px -1000px;
        background-repeat: no-repeat;
        &.done{
            background-position:0 0;
            border: 1px solid #000;
            .row{
                opacity: 0;
            }
        }
        .row{
            display: table-row;
            transition: opacity 500ms;
            .pzl{
                border:1px solid #000;
                display: table-cell;
                transition: opacity 300ms;
                &.clicked{
                    opacity: 0.5;
                }
            }
        }
    }
}
.timeColumn {
    font-size: 30px;
    line-height: 35px;
}
</style>
