<template>
  <body>
    <div class="main-app" id="main">
      <header>Guess The Word</header>
      <div class="main-container">
        <div class="secondary-container">
          <section class="trials" id="trials">
          </section>

          <footer>
            <div class="check-word" id="check-word" @click="checkWord()">Check Word</div>
            <div class="hint" id="hint" @click="useHint">{{ hints }} Hint</div>
          </footer>
        </div>

        <section class="color-hints">
          <div class="no-style">Key Colors</div>
          <div class="right-in-place">Letter is Right And in Place</div>
          <div class="right-not-in-place">Letter is Right But Not in Place</div>
          <div class="wrong">Letter is Wrong And Not Exists in Word</div>
        </section>
      </div>

    </div>
    <div class="next-level" id="nextLevel">
      <lord-icon
          src="https://cdn.lordicon.com/cqofjexf.json"
          trigger="loop"
          delay="1000"
          style="width:50px;height:50px">
      </lord-icon>
      <p class="congrats">Congratulations!</p>
      <p class="word">Word : {{ word.toLowerCase() }}</p>
      <div>
        <i class="bi bi-play-btn-fill" title="Next Level" @click="nextLevel()"></i>
      </div>
      
    </div>
  </body>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      hintPlaces: [],
      currentHintPlaces: [],
      hints : 3,
      word : "console",
      idTrial : 10,
      currentInputIndex : 0,
      correctAnswerFlag : 0
    }
  },
  mounted(){
    window.addEventListener('keypress', this.assignLetters)
    window.addEventListener('keydown', this.removeLetters)
    this.newTrial()
    this.getWord()
  },
  methods:{
    nextLevel(){
      this.hints += 1
      this.correctAnswerFlag = 0
      this.idTrial = 10
      document.getElementById('nextLevel').style.display = "none"
      document.getElementById('main').style.opacity = 1
      this.getWord()
      document.getElementById("trials").innerHTML = ""
      this.newTrial()
      document.getElementById('check-word').style.pointerEvents = "all"
      document.getElementById('hint').style.pointerEvents = "all"
    },
    removeLetters(event){
      if(event.key == 'Backspace' && this.correctAnswerFlag == 0){
        if(this.currentInputIndex > 0){
          this.currentInputIndex -=1
          while(this.currentInputIndex >= 0){
            if(this.currentHintPlaces[this.currentInputIndex] == 0){
              document.getElementById(String(this.idTrial)).childNodes[this.currentInputIndex+2].innerHTML = "" 
              break
            }else {
              this.currentInputIndex -= 1
            }
          }
        }
      }
    },
    async getWord(){
      const apiUrl = 'https://random-word-api.herokuapp.com/word?length=7'
      await fetch(apiUrl)
        .then(response => {
          if (!response.ok) {
            throw new Error('Network response was not ok');
          }
          return response.json();
        })
        .then(data => {
          this.word = data[0]
          this.hintPlaces = new Array(this.word.length).fill(0);
          this.currentHintPlaces = new Array(this.word.length).fill(0);
        })
        .catch(error => {
          console.error('Error:', error);
        });
    },
    newTrial(){
      let trialText = ""
      trialText += `<div id="${this.idTrial}">
              <div class="try">Try ${this.idTrial-9}</div>`
      for(let i = 0 ; i < this.word.length ; i++){
        trialText += `<div class="input" id="${i}"></div>`
      }
      trialText += `</div>`
      document.getElementById("trials").innerHTML += trialText
    },
    useHint(){
      if(this.hints > 0){
        //get random no. instead
        var Chance = require('chance');
        // Instantiate Chance so it can be used
        var chance = new Chance();

        let hintPlace = chance.integer({ min: 0, max: this.word.length-1 })
        while(this.hintPlaces[hintPlace] == 1){
          hintPlace = chance.integer({ min: 0, max: this.word.length-1 })
        }
        this.hintPlaces[hintPlace] = 1
        this.currentHintPlaces[hintPlace] = 1
        this.word = this.word.toUpperCase()
        let place = document.getElementById(String(this.idTrial)).childNodes[hintPlace+2]
        place.innerHTML = this.word[hintPlace]
        place.style.backgroundColor = "#ff9a09"
        place.style.color = "#ffffff"
        place.style.border = "none"
        place.style.opacity = "0.5"
        this.hints --
      }
        
      if(this.hints == 0)
        this.disableHints()

    },
    disableHints(){
      document.getElementById("hint").style.pointerEvents = "none"
    },
    assignLetters(event){
      if(event === undefined){
      } else if (this.correctAnswerFlag == 0){
        if(event.keyCode == 13 && document.getElementById("check-word") != document.activeElement){
          this.checkWord()
        } else if (event.keyCode >= 48 && event.keyCode <= 57){
          //do nothing
        }else {
          if(this.currentInputIndex < this.word.length){
            while(this.currentInputIndex < this.word.length){
              if(this.currentHintPlaces[this.currentInputIndex] == 0){
                document.getElementById(String(this.idTrial)).childNodes[this.currentInputIndex+2].innerHTML = event.key.toUpperCase()
                this.currentInputIndex +=1
                break
              }else{
                this.currentInputIndex += 1
              }
            }
          }
          
        }
      } else if (event.keyCode == 13) {
        this.nextLevel()
      }
    },
    checkWord(){
      this.word = this.word.toUpperCase()
      let wrongFlag = 0
      for(let i = 0 ;i < this.word.length; i++){
        let place = document.getElementById(String(this.idTrial)).childNodes[i+2]
        if(place.innerHTML == String(this.word[i])){
          place.style.backgroundColor = "#ff9a09"
          place.style.color = "#ffffff"
          place.style.border = "none"
          place.style.opacity = "0.5"
        } else {
          for(let j = 0 ; j < this.word.length; j++){
            if(place.innerHTML == String(this.word[j])){
              place.style.backgroundColor = "#06aa88"
              place.style.color = "#ffffff"
              place.style.border = "none"
              place.style.opacity = "0.5"
              break;
            } else if(j == this.word.length-1) {
              place.style.backgroundColor = "#293045"
              place.style.color = "#ffffff"
              place.style.border = "none"
              place.style.opacity = "0.5"
            }
          }
          wrongFlag = 1
        }
      }
      this.currentInputIndex = 0
      this.currentHintPlaces = new Array(this.word.length).fill(0);
      if(wrongFlag){
        document.getElementById(String(this.idTrial)).childNodes[1].style.opacity = "0.5"
        this.idTrial += 1
        this.newTrial()
        var elem = document.getElementById('trials');
        elem.scrollTop = elem.scrollHeight;
      } else {
        this.correctAnswerFlag = 1
        document.getElementById('nextLevel').style.display = "block"
        document.getElementById('main').style.opacity = 0.6
        document.getElementById('check-word').style.pointerEvents = "none"
        document.getElementById('hint').style.pointerEvents = "none"
      }
        
    }
  }
}
</script>

<style>
@import url("https://cdn.jsdelivr.net/npm/bootstrap-icons@1.3.0/font/bootstrap-icons.css");
html,body {
  margin-top: 70px;
  background-color: #1a202a;
}
.main-app {
  padding: 10px;
  background-color: #ebebeb;
  margin: auto;
  max-width: 800px;
  min-height: 400px;
  display: flex;
  flex-direction: column;
  position: relative;
}
* {
  user-select: none;
  font-family: 'Cairo', sans-serif;
  text-align: center;
  font-weight: bold;
}
header {
  background-color: #ffffff;
  color: #1a202a;
  padding: 10px 0;
  font-weight: bold;
  font-size: 20px;
}
.main-container {
  display: flex;
  justify-content: space-evenly;
  position: absolute;
  height: 80%;
  bottom: 20px;
}
.secondary-container {
  padding: 10px;
  width: 50%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.color-hints {
  width: 45%;
  padding: 10px;
}
footer {
  height: 15%;
  width: 100%;
  display: flex;
  justify-content: space-evenly;
}
.check-word {
  font-size: 15px;
  width: 70%;
  color: white;
  background-color: #e84b2d;
  border-radius: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.check-word:hover {
  background-color: #c03c22;
  color: #ddd;
  cursor: pointer;
}
.hint {
  font-size: 15px;
  width: 20%;
  color: white;
  background-color: #007678;
  border-radius: 3px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.hint:hover {
  color: #ddd;
  background-color: #016264;
  cursor: pointer;
}
button {
  padding: 5px;
  border-radius: 3px;
  border: none;
}
.color-hints div {
  margin: 28px 10px;
  padding: 0px 5px;
  background-color: #ffffff;
  text-align: left;
  outline: 10px solid #ffffff;
  border-left: 30px solid black;
}
.color-hints .wrong {
  border-color: #293045;
}
.color-hints .right-not-in-place {
  border-color: #06aa88;
}
.color-hints .right-in-place {
  border-color: #ff9a09;
}
.color-hints .no-style {
  text-align: center;
  background: none;
  outline: none;
  border: none;
  font-size: 20px;
}
.trials {
  padding: 10px;
  height: 90%;
  overflow-y: scroll;
  overflow-y:auto;
}
.trials div {
  display: flex;
  align-items: center;
  margin: 3px 0;
}
.trials div div {
  background-color: #ffffff;
  width: 40px;
  height: 40px;
  border-bottom: 3px solid black;
  margin: 3px;
  font-size: 35px;
  display: flex;
  justify-content: center;
}
.trials div .try {
  background: none;
  border: none;
  font-size: 18px;
  min-width: 60px;
}
.trials::-webkit-scrollbar {
    display: block;
    width: 5px;
}
.trials::-webkit-scrollbar-track {
    background: transparent;
}  
.trials::-webkit-scrollbar-thumb {
    background-color: #293045;
    border-right: none;
    border-left: none;
}
.trials::-webkit-scrollbar-track-piece:end {
    background: transparent;
    margin-bottom: 10px; 
}
.trials::-webkit-scrollbar-track-piece:start {
    background: transparent;
    margin-top: 10px;
}
.trials .input:hover{
  background-color: #ddd;
}
.next-level {
  background-color: white;
  width: 400px;
  height: 250px;
  position: absolute;
  border-radius: 10px;
  box-shadow: rgba(0, 0, 0, 0.35) 0px 10px 20px;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  border: 5px solid #ff9a09;
  outline: 10px solid white;
  display: none;
}
.next-level lord-icon {
  margin-top: 10px;
}
.next-level .congrats{
  font-size: 40px;
  margin: -5px 0 0;
}
.next-level .word{
  font-size: 20px;
  margin: 0;
}
.next-level div {
  margin-top: -5%;
}
i {
  font-size: 40px;
  transition-duration: 100ms;
  color: #ff9a09;
}
i:hover {
  color: #c97c0f;
}
</style>