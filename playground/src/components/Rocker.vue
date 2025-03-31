<template>
    <p class="metronome" style="position: absolute"></p>
    <span id="perfect" :class="{ 'pop-animation': perfect !== '' }">{{ perfect }}</span>
    <div>
      <button v-if="!gameStarted" v-on:click="startGame">You Play</button>
      <div v-if="gameStarted">
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
  
      <div class="life-bar">
        <div class="life" :style="{ width: life + '%' }"></div>
      </div>
    </div>
  </div>
  </template>
  
  <script>
  import song from '@/assets/parabens1.mp3';
  import wrongnote from '@/assets/Missed.mp3';
  import { Midi } from '@tonejs/midi';

  export default {
    data() {
      return {
        missed: new Audio(wrongnote),
        audio: new Audio(song),
        gameStarted: false,
        chart: [
        ],
        activeNotes: [],
        points: 0,
        life: 100,
        perfect: '',
        perfectHits: 0,
        key: {
          a: false,
          s: false,
          d: false,
          k: false,
          l: false
        }
      };
    },
    mounted() {
      window.addEventListener("keydown", this.handleKeydown);
    },
    beforeUnmount() {
      window.removeEventListener("keydown", this.handleKeydown);
    },
    methods: {
      startGame() {
  this.gameStarted = true;

  // Chamar loadMidi e aguardar o carregamento do arquivo MIDI antes de continuar
  this.loadMidi().then(() => {
    // Verificar se o chart está vazio após carregar o MIDI
    this.$nextTick(() => {
      console.log(this.chart);
      console.log('Chart data (cópia):', Array.from(this.chart)); // Verifique se a cópia do chart é válida

      if (this.chart.length > 0) {
        // Adicionar uma verificação para garantir que as notas sejam criadas no tempo certo
        Array.from(this.chart).forEach((note) => {
          const noteDelay = note.time;  // Certifique-se de que note.time está no formato correto (milissegundos)
          setTimeout(()=>{
          this.audio.play();
        }, 400);
          // Esperar o tempo correto e criar a nota
          setTimeout(() => {
            this.createNote(note);
          }, noteDelay + 400); // Ajustar a criação da nota com base no tempo
        });
      } else {
        console.error('Chart está vazio, verifique a lógica de carregamento do MIDI');
      }
    });
    

    // Começar o metrônomo e tocar o áudio
    setTimeout(() => {
      this.metronome(); 
    }, 400);  // Ajuste o tempo do metrônomo conforme necessário
    this.missed.play();
    this.missed.muted = true;
    this.missed.loop = true;
  }).catch((error) => {
    console.error('Erro ao carregar o arquivo MIDI:', error);
  });
},

// Atualize o método loadMidi para retornar uma Promise, para que possamos aguardar o carregamento
async loadMidi() {
  try {
    
    const response = await fetch('/parabens.mid');
    if (!response.ok) {
      throw new Error('Erro ao carregar o arquivo MIDI');
    }

    const midiData = await response.arrayBuffer();
    console.log('Arquivo MIDI carregado:', midiData);

    // Usando @tonejs/midi para processar o arquivo MIDI
    const midi = new Midi(midiData);
    const notes = this.convertMidiToNotes(midi);

    console.log("Notas convertidas: ", notes);
    this.chart = notes;
    console.log(this.chart[0]);

    // Retornar uma Promise resolvida quando os dados estiverem prontos
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
    const positions = { a: 8, s: 72, d: 135, k: 200, l: 265 };
    const colors = { a: "lightgreen", s: "crimson", d: "gold", k: "cornflowerblue", l: "tomato" };

    const bpm = midi.header.tempos[0]?.bpm || 120; // Pega o BPM do arquivo MIDI
    const tickDuration = (60 / bpm) * 1000; // Tempo por tick em ms
    const thresholdMs = 50; // Limite para considerar notas como próximas (em milissegundos)

    let prevNoteTimeInMs = null; // Para armazenar o tempo da última nota

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

      metronome() {
          var bpm = 150;

          var notebeat = 60 / bpm;

          var spacebeat = Math.round(notebeat * 1000); // Arredondamento para evitar problemas com valores de

          let count = 1; // Inicia em 1

          clearInterval(window.metronomeInterval); // Garante que não há intervalos duplicados antes de iniciar um novo

          window.metronomeInterval = setInterval(() => {
              document.querySelector(".metronome").innerHTML = count;

              count = count < 4 ? count + 1 : 1; // Reinicia para 1 após 4

          }, spacebeat);
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
            this.life -= 15;
            this.audio.muted = true;
            setTimeout(()=>{
              this.audio.muted = false;
            },1000);

            /*f (this.life <= 0) {
              setTimeout(() => {
                alert("Game Over!");
                location.reload();
              }, 1000);
            }*/
          }
        }, 15);
      },
      handleKeydown(event) {
        const keyPressed = event.key.toLowerCase();
        if (!["a", "s", "d", "k", "l"].includes(keyPressed)) return;
  
        if(this.perfectHits === 4 || this.perfectHits === 6){
          this.key[keyPressed] = true;
          this.perfectHits = 0; // Set the key as pressed
          setTimeout(() => {
            this.key[keyPressed] = false; // Reset after animation
          }, 800);
        }

        const note = this.activeNotes.find((n) => n.key === keyPressed && n.active);
  
        document.getElementById(keyPressed).style.transform = "scale(1.1)";
        setTimeout(() => {
          document.getElementById(keyPressed).style.transform = "scale(1)";
        }, 200);
  
        if (note) {
          if (note.top > 410 && note.top < 470) {
            console.log(this.top)
            this.points++;
            note.active = false;
            this.life = Math.min(this.life + 5, 100);
            this.activeNotes = this.activeNotes.filter((n) => n.id !== note.id);
            console.log(`Acertou! Pontuação: ${this.points}`);
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
                    console.log(this.perfectHits)
            if (note.top > 440 && note.top <450) {
                    this.perfect = 'You rock!';
                    
            } else if (note.top > 420 && note.top < 439 || note.top > 451 && note.top <470) {
                    this.perfect = 'good';
            }
          } else {
            console.log("Errou!");
            this.audio.muted = true;
            this.missed.muted = false;
            setTimeout(()=>{
              this.missed.muted = true;
              this.audio.muted = false;
            }, 300);
            this.activeNotes = this.activeNotes.filter((n) => n.id !== note.id);
            document.getElementById("game-container").style.border = "4px solid red";
            setTimeout(() => {
            this.perfect = '';
            }, 1000);
            setTimeout(() => {
              document.getElementById("game-container").style.border = "2px solid white";
            }, 200);
            this.life -= 5;
            /*if (this.life <= 0) {
              alert("Game Over!");
              location.reload();
            }*/
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
    -webkit-transform: perspective(250px) rotateX(15deg);    
    position: relative;
    bottom: 200px;
    justify-content: center;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
  }
  
  .note {
    position: absolute;
    width: 50px;
    height: 40px;
    border-radius: 50%;
    text-align: center;
    line-height: 60px;
    font-size: 24px;
    font-weight: bold;
    color: white;
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -8px 0px inset;
  }

  .notepath{
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
    position: absolute;
    color: yellow;
    font-size: 35px;;
    font-weight: normal;
    top: 150px;
    -webkit-text-stroke-width: 2px; /* largura da borda */
    -webkit-text-stroke-color: gold;
    filter: drop-shadow(2px 7px goldenrod);
    display: inline-block;
  }

.pop-animation {
  animation: popEffect 0.3s ease-out;
}
  </style>
  