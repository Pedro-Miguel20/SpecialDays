<script setup>
import Bongo from "./Bongo.vue";
</script>

<template>
    
    <div>
      <div class="piano-key">
        <button v-if="!gameStarted && endgame === false" v-on:click="startGame" class="white-key">You Play</button>
      </div>
      
      <div v-if="gameStarted && endgame === false" class="allwrapper">
      <Bongo></Bongo>
      <div id="game-container">
        <div class="notepath">
            <div class="hit-zone" id="a" style="background-color: lightgreen;"><img v-if="key['a']" src="../assets/thunder.gif" style="position: absolute;     z-index: 100;
    bottom: -37px;"><div class="insidehit-zone" id="akey"></div></div></div>
        <div class="notepath">
            <div class="hit-zone" id="s" style="background-color: crimson;"><img v-if="key['s']" src="../assets/thunder.gif" style="position: absolute;     z-index: 100;
    bottom: -37px;"><div class="insidehit-zone" id="skey"></div></div></div>
        <div class="notepath">
            <div class="hit-zone" id="d" style="background-color: gold;"><img v-if="key['d']" src="../assets/thunder.gif" style="position: absolute;     z-index: 100;
    bottom: -37px;"><div class="insidehit-zone" id="dkey"></div></div></div>
        <div class="notepath">
            <div class="hit-zone" id="k" style="background-color: cornflowerblue;"><img v-if="key['k']" src="../assets/thunder.gif" style="position: absolute;     z-index: 100;
    bottom: -37px;"><div class="insidehit-zone" id="kkey"></div></div></div>
        <div class="notepath">
            <div class="hit-zone" id="l" style="background-color: tomato;"><img v-if="key['l']" src="../assets/thunder.gif" style="position: absolute;     z-index: 100;
    bottom: -37px;"><div class="insidehit-zone" id="lkey"></div></div></div>
        
        <div
          v-for="note in activeNotes"
          :key="note.id"
          class="note"
          :style="{ backgroundColor: note.color, left: note.position + 'px', top: note.top + 'px' }"
        ></div>
      </div>
      <span id="perfect" :class="{ 'pop-animation': perfect !== '' }">{{ perfect }}</span>
      <img src="https://i.pinimg.com/originals/11/0e/7c/110e7c1e1c8c8953e787b56fdff866ed.gif" width="250px">

      <!--<div class="life-bar">
        <div class="life" :style="{ width: life + '%' }"></div>
      </div>-->
      </div>
      <div v-if="endgame" class="wrapstats">
        <div class="wrappall">
          <h4 class="titlestats">{{ achieve }}</h4>
          <span></span><p>{{ points }} points</p>
          
          <button class="balloon-button" v-if="points > count/2 && !showGift" @click="openGift">Abra seu presente</button>

    <!-- Mostrar o GIF do presente -->
    <div v-if="showGift">
      <img src="../assets/gift.gif" id="gift" />
    </div>

    <!-- Mostrar o conteúdo surpresa depois do tempo -->
    <div v-if="showSurprise" style="position: absolute; bottom: 220px;">
      <span style="color:darkslategray;">KKKKKKK ABRE AEPOXA</span>
    </div>

    <a href="/"><span style="color:darkslategray;">🔙Voltar</span></a>

        </div>
    </div>
  </div>
  </template>
  
  <script>
  import song from '@/assets/parabens.mp3';
  import wrongnote from '@/assets/missedpiano.mp3';
  import { Midi } from '@tonejs/midi';
  import { watch, ref } from "vue";

  export default {
    emits: ['update'],
    data() {
      return {
        missed: new Audio(wrongnote),
        audio: new Audio(song),
        gameStarted: false,
        endgame: false,
        showGift: false,
        showSurprise: false,
        achieve: '',
        chart: [],
        activeNotes: [],
        points: 0,
        count: 1,
        life: 100,
        perfect: '',
        perfectHits: 0,
        key: (
          {a: false, status: 3, hand: 'l', number: 1},
          {s: false, status: 2, hand: 'l', number: 2},
          {d: false, status: 1, hand: 'l', number: 3},
          {k: false, status: 2, hand: 'r', number: 4},
          {l: false, status: 1, hand: 'r', number: 5}
        ),
        current: {
          right: 'idl',
          left: 'idl'
        },
        currentkey: {
          right: '',
          left: ''
        },
        colorMap: {
          a: '#6cff39',
          s: '#f70039',
          d: '#fcde2d',
          k: '#4bfef8',
          l: '#f8410f'
        }
      };
    },
    mounted() {
      window.addEventListener("keydown", this.handleKeydown);
      this.$emit('update', this.gameStarted);
    },
    beforeUnmount() {
      window.removeEventListener("keydown", this.handleKeydown);
    },
    methods: {
      startGame() {
        this.gameStarted = true;

        this.$emit('update', this.gameStarted);
        
        // Chamar loadMidi e aguardar o carregamento do arquivo MIDI antes de continuar
        this.loadMidi().then(() => {
          // Verificar se o chart está vazio após carregar o MIDI
          this.$nextTick(() => {

            if (this.chart.length > 0) {
              // Adicionar uma verificação para garantir que as notas sejam criadas no tempo certo
              Array.from(this.chart).forEach((note) => {
                const noteDelay = note.time;  // Certifique-se de que note.time está no formato correto (milissegundos)
                setTimeout(() => {
                  this.audio.play();
                }, 2700);

                // Esperar o tempo correto e criar a nota
                setTimeout(() => {
                  this.createNote(note);
                }, noteDelay); // Ajustar a criação da nota com base no tempo
              });
            } else {
              console.error('Chart está vazio, verifique a lógica de carregamento do MIDI');
            }
          });

          this.missed.play();
          this.missed.muted = true;
          this.missed.loop = true;
        }).catch((error) => {
          console.error('Erro ao carregar o arquivo MIDI:', error);
        });
      },
      openGift() {
      this.showGift = true;

      // Tempo em milissegundos (ajuste conforme o tempo do seu gif)
      setTimeout(() => {
        this.showSurprise = true;
      }, 3700); // 2 segundos
      
    },
      animatedConfetti() {
      const end = Date.now() + 15 * 1000;

// go Buckeyes!
      const colors = ["#a864fd", "#29cdff", "#78ff44", "#ff718d", "#fdff6a"];

      (function frame() {
        confetti({
          particleCount: 2,
          angle: 60,
          spread: 55,
          origin: { x: 0 },
          colors: colors,
        });

        confetti({
          particleCount: 2,
          angle: 120,
          spread: 55,
          origin: { x: 1 },
          colors: colors,
        });

        if (Date.now() < end) {
          requestAnimationFrame(frame);
        }
      })();
    },

    loserConfetti() {
      const end = Date.now() + 15 * 1000;

// go Buckeyes!

      (function frame() {
        confetti({
          particleCount: 1,
          angle: 60,
          spread: 25,
          scalar: 10,
          origin: { x: 0 },
          shapes: ["image"],
          shapeOptions: {
          image: [{
              src: "https://png.pngtree.com/png-clipart/20230804/original/pngtree-loser-emoticon-design-making-icon-vector-picture-image_9535316.png",
              width: 62,
              height: 62,
            },
          ]}
        });

        confetti({
          particleCount: 1,
          angle: 120,
          spread: 25,
          scalar: 10,
          origin: { x: 1 },
          shapes: ["image"],
          shapeOptions: {
          image: [{
              src: "https://png.pngtree.com/png-clipart/20230804/original/pngtree-loser-emoticon-design-making-icon-vector-picture-image_9535316.png",
              width: 62,
              height: 62,
            },
          ]}
        });

        if (Date.now() < end) {
          requestAnimationFrame(frame);
        }
      })();
    },

      async loadMidi() {
        try {
          const response = await fetch('/parabens.mid');
          if (!response.ok) {
            throw new Error('Erro ao carregar o arquivo MIDI');
          }

          const midiData = await response.arrayBuffer();

          // Usando @tonejs/midi para processar o arquivo MIDI
          const midi = new Midi(midiData);
          const notes = this.convertMidiToNotes(midi);
          this.chart = notes;

          return Promise.resolve();

        } catch (error) {
          console.error('Erro ao carregar ou processar o arquivo MIDI:', error);
          return Promise.reject(error);  // Retorne uma Promise rejeitada se houver erro
        }
      },

      convertMidiToNotes(midi) {
        const notes = [];
        const keyMap = {
            60: "a", 61: "a", 
            63: "s", 
            66: "d", 67: "d",
            69: "k", 71: "k", 
            72: "l", 74: "l"
        };
        const positions = { a: 6, s: 69, d: 132, k: 196, l: 257};
        const colors = { a: "lightgreen", s: "crimson", d: "gold", k: "cornflowerblue", l: "tomato" };

        const bpm = midi.header.tempos[0]?.bpm || 120; // Pega o BPM do arquivo MIDI
        const tickDuration = (60 / bpm) * 1000; // Tempo por tick em ms
        const thresholdMs = 50; // Limite para considerar notas como próximas (em milissegundos)

        let prevNoteTimeInMs = null;

        midi.tracks.forEach(track => {
            track.notes.forEach(note => {
                const key = keyMap[note.midi];
                if (!key) return; // Ignora notas que não estão no keyMap

                const noteTimeInTicks = note.ticks; // Tempo da nota em ticks
                const noteTimeInMs = noteTimeInTicks * tickDuration / midi.header.ppq; // Converte para milissegundos

                // Se a diferença entre o tempo da nota atual e a anterior for menor que o limiar, desconsidere a nota atual
                if (prevNoteTimeInMs && Math.abs(noteTimeInMs - prevNoteTimeInMs) < thresholdMs) {
                    return; // Ignora a nota se a diferença de tempo for muito pequena
                }

                const newNote = { 
                    key, 
                    color: colors[key], 
                    time: noteTimeInMs, 
                    position: positions[key], 
                };

                notes.push(newNote);
                prevNoteTimeInMs = noteTimeInMs; // Atualiza o tempo da última nota
            });
        });

        return notes;
      },
      createNote(note) {
        const noteObj = {
          id: Date.now() + Math.random(), // ID único para o Vue controlar
          key: note.key,
          color: note.color,
          position: note.position,
          top: -60,
          active: true,
        };

        // Verifique se o jogo deve acabar
        this.count++;
        this.activeNotes.push(noteObj);
        this.animateNote(noteObj);
        
        
      },

      animateNote(noteObj) {
    const interval = setInterval(() => {
        const foundNote = this.activeNotes.find((n) => n.id === noteObj.id);
        if (!foundNote) {
            clearInterval(interval);
            return;
        }

        
        foundNote.top += 4;
        if (foundNote.top >= 530) {
            clearInterval(interval);
            this.activeNotes = this.activeNotes.filter((n) => n.id !== noteObj.id);
            this.life -= 15;  // Diminui a vida quando a nota ultrapassa a linha

            this.audio.muted = true;
            setTimeout(() => {
                this.audio.muted = false;
            }, 1000);
        }
    }, 15);
    if(this.count > this.chart.length) {
      setTimeout(() => {
        this.endGame(this.endgame); // Chama o método endGame
      }, 4000);
    };
},

      endGame(){
        this.endgame = true;
        const victory = this.count / 2;
        if(victory < this.points){
          console.log("You rock!: " + this.points + " points");
          this.achieve = 'YOU ROCK!';
          this.animatedConfetti();
        } else {
          console.log("You failed: " + this.points + " points");
          this.achieve = 'DETUNED:(';
          this.loserConfetti();
        }
      },

      changeHand(hand, status, key) {
        const handname = hand[0] + 'hand' + status;
        const currname = hand[0] + 'hand' + this.current[hand]; // Corrigido

        const currEl = document.getElementById(currname);
        if (currEl) currEl.style.display = 'none';

        if (this.current[hand] !== 'idl') {
          const currAnim = document.getElementById(currname + '_');
          if (currAnim) currAnim.style.display = 'none';
        }

        const newEl = document.getElementById(handname);
        if (newEl) newEl.style.display = 'initial';

        if (status !== 'idl') {
          const newAnim = document.getElementById(handname + '_');
          if (newAnim) newAnim.style.display = 'initial';
        }

        this.current[hand] = status;
      },

      handleKeydown(event) {
        const keyPressed = event.key.toLowerCase();
        // Verificar fim de jogo quando todas as notas forem tocadas

        if (!["a", "s", "d", "k", "l"].includes(keyPressed)) return;

        const note = this.activeNotes.find((n) => n.key === keyPressed && n.active);

        let hand = '';
        let status = '';

        if (["a", "s", "d"].includes(keyPressed)) {
          hand = 'left';
        } else if (["k", "l"].includes(keyPressed)) {
          hand = 'right';
        }

        if (keyPressed === 'd' || keyPressed === 'k') status = '3';
        else if (keyPressed === 'a' || keyPressed === 'l') status = '1';
        else status = '2';

        this.changeHand(hand, status, keyPressed);

        setTimeout(() => {
          this.changeHand(hand, 'idl');
        }, 300);
        
        if(this.perfectHits === 4 || this.perfectHits === 6){
          this.key[keyPressed] = true;
          this.perfectHits = 0; // Set the key as pressed
          setTimeout(() => {
            this.key[keyPressed] = false; // Reset after animation
          }, 800);
        }

        document.getElementById(keyPressed).style.transform = "scale(1.1)";
        setTimeout(() => {
          document.getElementById(keyPressed).style.transform = "scale(1)";
        }, 200);

        if (note) {
          if (note.top > 410 && note.top < 470) {
            this.points++;
            note.active = false;
            this.life = Math.min(this.life + 5, 100);
            this.activeNotes = this.activeNotes.filter((n) => n.id !== note.id);
            //console.log(Acertou! Pontuação: ${this.points});//
            document.getElementById("game-container").style.border = "4px solid #80eF80";
            document.getElementById(keyPressed + "key").style.boxShadow = "0 0 20px 15px #ffa700";
            setTimeout(() => {
            this.perfect = '';
            }, 500);
            setTimeout(() => {
              document.getElementById("game-container").style.border = "2px solid white";
              document.getElementById(keyPressed + "key").style.boxShadow = "none";
            }, 200);
            this.perfectHits += 1;
            if (note.top > 440 && note.top <450) {
                    this.perfect = 'You rock!';
                    
            } else if (note.top > 420 && note.top < 439 || note.top > 451 && note.top <470) {
                    this.perfect = 'good';
            }
          } else {
            this.audio.muted = true;
            this.missed.muted = false;
            setTimeout(() => {
              this.missed.muted = true;
              this.audio.muted = false;
            }, 250);
            this.activeNotes = this.activeNotes.filter((n) => n.id !== note.id);
            document.getElementById("game-container").style.border = "4px solid red";
            setTimeout(() => {
              this.perfect = '';
            }, 1000);
            setTimeout(() => {
              document.getElementById("game-container").style.border = "2px solid white";
            }, 200);
            this.life -= 5;
          }
        }
      }
    }
  };
</script>

<style scoped>
  @font-face {
    font-family: 'daydream';
    src: url('../Fonts/Daydream.ttf');
  }

  .note {
				width: 100px;
				height: 100px;
				border: black;
    		border-style: solid;
			}

  #game-container {
    position: relative;
    width: max-content;
    height: 500px;
    background: rgb(92,84,84);
background: linear-gradient(0deg, rgba(92,84,84,1) 0%, rgba(152,150,150,1) 100%);
    border: 2px solid white;
    display: flex;
    border-radius: 10px;
    overflow: hidden;
    transform-style: preserve-3d;
    -webkit-transform: perspective(250px) rotateX(25deg);    
    position: relative;
    bottom: 160px;
    justify-content: center;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
  }
  
  .note {
    position: absolute;
    width: 13%;
    height: 6%;
    border-radius: 50%;
    text-align: center;
    line-height: 60px;
    font-size: 24px;
    font-weight: bold;
    color: white;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -8px 0px inset;
  }

  .allwrapper{
    display: flex;
    flex-direction: row;
    align-items: center;
    gap: 150px;
  }

  .notepath{
    position: relative;
    background-color: transparent; 
    width: max-content; 
    height: 100%;
    border: solid 2px white; 
    display: flex;
    align-items: flex-end;
    padding: 5px;
  }
  
  .hit-zone {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 50px;
    height: 40px;
    position: relative;
    bottom: 20px;
    background-color: cornflowerblue;
    bottom: 20px;
    border-radius: 50%;
    transform-style: preserve-3d;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
  }

  .life-bar {
    width: 100px;
    height: 20px;
    border: 2px solid white;
  }
  
  .life {
    background: red;
    width: 100%;
    height: 100%;
  }

  .insidehit-zone{
    background-color: gray; 
    width: 30px; 
    height: 22px; 
    border-radius: 50%; 
    position: relative; 
    bottom: 3px;
    transform-style: preserve-3d;
    will-change: transform;
  backface-visibility: hidden;
    box-shadow: rgba(50, 50, 93, 0.25) 0px 30px 60px -12px inset, rgba(0, 0, 0, 0.3) 0px 18px 36px -18px inset;
  }

  @keyframes popEffect {
  0% {
    transform: scale(1);
    opacity: 0.8;
  }
  50% {
    transform: scale(1.3);
    opacity: 1;
  }
  100% {
    transform: scale(1);
    opacity: 0.8;
  }
}

#perfect{
    font-family: 'daydream';
    color: yellow;
    font-size: 35px;;
    font-weight: normal;
    z-index: 100;
    -webkit-text-stroke-width: 2px; /* largura da borda */
    -webkit-text-stroke-color: gold;
    filter: drop-shadow(2px 7px goldenrod);
    display: inline-block;
  }

.pop-animation {
  animation: popEffect 0.3s ease-out;
}

.titlestats, p{
  color: darkslategray;
  margin: 10px;
}

.wrapstats{
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  margin: 30px auto;
}

.wrappall{
  display: flex;
  flex-direction: column;
  align-items: center;
  font-size: 40px;
}

.piano-key {
  position: relative;
  display: inline-block;
  margin: 20px;
}

.white-key {
  width: 180px;
  height: 60px;
  background: linear-gradient(to right, #ffffff, #eeeeee);
  border: 4px solid #000;
  border-radius: 10px;
  font-size: 1.5rem;
  font-weight: bold;
  color: #333;
  box-shadow: 0 6px #555;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s;
}

.white-key:active {
  transform: translateY(4px);
  box-shadow: 0 2px #555;
}

#canvas3d .spline-watermark{
  display: none !important;
}

.balloon-button {
    margin: 20px;
      background: linear-gradient(to top, #ff009d, #ff9999);
      border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
      width: 120px;
      height: 150px;
      position: relative;
      color: white;
      font-family: "Patrick Hand", cursive;
      font-weight: bold;
      font-size:26px;
      cursor: pointer;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2);
      padding:20px;
    }

    .balloon-button:hover {
        transform: scale(1.1);
    }

    .balloon-button::after {
      content: '';
      position: absolute;
      bottom: -15px;
      left: 50%;
      transform: translateX(-50%);
      width: 20px;
      height: 20px;
      background: #ff009d;
      clip-path: polygon(50% 0%, 0% 100%, 100% 100%);
    }
</style>
  
