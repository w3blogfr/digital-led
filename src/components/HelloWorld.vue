<template>
  <div class="form-group"> 
    <label>Ligne</label>
    <select v-model="row">
      <template v-for="r in ['1','2','3','4','5','6','7','8','9','10','A','B','C','D','E','F','G','H','I','J','K']" :key="r">
        <option :value="1">{{ r }}</option>
      </template>
    </select>
  </div>
  
  <div class="form-group"> 
    <label>Luminosité</label>
    <select v-model="lightness">
      <option :value="1">Faible</option>
      <option :value="2">Moyen</option>
      <option :value="3">Fort</option>
    </select>
  </div>

  
  <div class="form-group"> 
    <label>Couleur arrière plan</label>
    <select v-model="backgroundColor">
      <option :value="null"></option>
      <template v-for="c in colors" :key="c">
        <option :value="c.value">{{ c.name }}</option>
      </template>
    </select>
  </div>

  <template v-for="(sequence,index) in sequences" :key="sequence">
    <fieldset>
      <legend>Sequence {{ index+1 }}</legend>
      <div class="form-group"> 
        <label>Type</label>
        <select v-model="sequence.type">
          <option>TEXT</option>
          <option>CHRONO</option>
        </select>
      </div>
      <div class="form-group" v-if="sequence.type=='CHRONO'"> 
        <label>Date</label>
        <input type="datetime-local" v-model="sequence.date" step="1"/>
      </div>
      <div class="form-group"> 
        <label>Text</label>
        <input type="text" v-model="sequence.text"/>
      </div>
      <div class="form-group"> 
        <label>Couleur</label>
        <select v-model="sequence.color">
          <option :value="null"></option>
          <template v-for="c in colors" :key="c">
            <option :value="c.value">{{ c.name }}</option>
          </template>
        </select>
      </div>
      <div class="form-group"> 
        <label>Delay</label>
        <input type="text" v-model="sequence.delay"/> ms (0 si infini)
      </div>
      <div class="form-group"> 
        <label>Flash</label>
        <input type="checkbox" v-model="sequence.flash"/> 
      </div>
      <div class="form-group" v-if="sequence.flash"> 
        <label>Nombre de Flashs</label>
        <input type="number" v-model="sequence.flashCounter" min="0" max="9"/> 
      </div>
      <div class="form-group" v-if="sequence.flash"> 
        <label>Vitesse du Flashs</label>
        <input type="number" v-model="sequence.flashSpeed" min="0" max="3"/> 
      </div>
      <button v-on:click="removeSequence(index)">Supprimer</button>    
    </fieldset>
  </template>

  <button v-on:click="addSequence">Ajouter une sequence</button>
  <br/>
  <br/>
  <button v-on:click="run">Envoyer</button>
</template>

<script>

var port = null;

async function openPort(){
  if(port==null){
          port = await navigator.serial.requestPort();
          await port.open({ baudRate: 9600, dataBits: 8, parity: "none", stopBits: 1, bufferSize:255 });
  }
}

async function sendCommand (command) {
      console.log(command);

      // Création de la commande à envoyer    

      var bytes = [];
      bytes.push(2); //<STX>
      let utf8Encode = new TextEncoder();
      bytes=bytes.concat(Array.from(utf8Encode.encode(command)));
      bytes.push(10); //<LF>

      console.log(bytes);
      const data = new Uint8Array(bytes);
        
      console.log(data);
      try {

        if(port==null){
          port = await navigator.serial.requestPort();
          await port.open({ baudRate: 9600, dataBits: 8, parity: "none", stopBits: 1, bufferSize:255 });
        }
        

        const writer = port.writable.getWriter();
        await writer.write(data);
        writer.releaseLock();

        /*
        const chunkSize = 4;
        for (let i = 0; i < bytes.length; i += chunkSize) {
            const chunk = bytes.slice(i, i + chunkSize);
            console.log('chunk',chunk);
           var  writer = port.writable.getWriter();

            await writer.write(new Uint8Array(chunk));
            writer.releaseLock();
          await new Promise(resolve => setTimeout(resolve, 20));
        }
        
        */

        //await port.close();
      
      } catch (error) {
        console.log(error);
        alert(error);
      }
    }

export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      timer: null,
      timerChrono: null,
      colors : [
        {
          'value': 1,
          'name': 'ROUGE'
        },
        {
          'value': 2,
          'name': 'VERT'
        },
        {
          'value': 3,
          'name': 'BLEU'
        },
        {
          'value': 4,
          'name': 'JAUNE'
        },
        {
          'value': 5,
          'name': 'VIOLET'
        },
        {
          'value': 6,
          'name': 'CYAN'
        },
        {
          'value': 7,
          'name': 'BLANC'
        },
        {
          'value': 8,
          'name': 'ORANGE'
        },
        {
          'value': 9,
          'name': 'ROSE'
        },
      ],
      row: '1',
      lightness:2,
      color: '',
      backgroundColor: '',
      sequences: [],
      repeatSequence: true,
    }
  },
  methods: {
    addSequence() {
      this.sequences.push({
        type: 'TEXT',
        date: null,
        text: "",
        color: '',
        flash: false,
        flashCounter: 0,
        flashSpeed: 0,
        delay: 0
      });
    },
    removeSequence(index) {
      this.sequences.splice(index, 1);
    },
    computeChrono(startDate){
      const currentDate = new Date();
      const timeDifference = currentDate - startDate;

      const days = Math.floor(timeDifference / (1000 * 60 * 60 * 24));
      const hours = Math.floor((timeDifference % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      const minutes = Math.floor((timeDifference % (1000 * 60 * 60)) / (1000 * 60));
      const seconds = Math.floor((timeDifference % (1000 * 60)) / 1000);

      var chrono=hours+':'+minutes.toString().padStart(2, '0')+':'+seconds.toString().padStart(2, '0');
      if(days>0){
        chrono = days+'d '+chrono;
      }
      return chrono;
    },
    createCommandFromSequence(sequence){
      var command="";
      command+=this.row;
      command+=this.lightness;
      if(sequence.flash){
        command+='^fs '+sequence.flashCounter+' '+sequence.flashSpeed+'^';
      }
      
      if(sequence.color){
        command+='^cs '+sequence.color+'^';
      }
      var text=sequence.text;
      if(sequence.type=='CHRONO'){
          text+=' '+this.computeChrono(new Date(sequence.date));
      }

      const chunkSize = 8;
      for (let i = 0; i < text.length; i += chunkSize) {
        const subText = text.substring(i, i + chunkSize);
        command+='^cs '+sequence.color+'^';
        command+=subText;
      }
      if(sequence.flash){
        command+='^fe^';
      }
      return command;
    },
    execute() {
      if(this.timer){
        clearTimeout(this.timer);
      }
      this.executeSequence(0);
    },
    executeSequence(index, isChrono){
      clearTimeout(this.timerChrono);
      if(index>=this.sequences.length && this.repeatSequence){
        index=0;
      }

      var sequence=this.sequences[index];
      if(index<this.sequences.length){
        var command=this.createCommandFromSequence(sequence);
        sendCommand(command);
        if(sequence.delay>0 && !isChrono){
          this.timer = setTimeout(() => {
            this.executeSequence(index+1,false);
          }, sequence.delay)
        }
      }
      if(sequence.type=='CHRONO'){
          this.timerChrono = setTimeout(() => {
            this.executeSequence(index,true);
          }, 1000)
      }
    },
    async run ()  {
      if (navigator.serial) {
        await openPort();
        this.execute();
      } else {
        alert('Web Serial API not supported.');
      }
    }
  },
  created () {
    this.addSequence();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.form-group{
  margin-top:5px;
  margin-bottom:5px;
}

.form-group label{
  width:200px;
  display:inline-block;
  text-align:right;
  padding-right:20px;
}

select {
  height:30px
}
</style>
