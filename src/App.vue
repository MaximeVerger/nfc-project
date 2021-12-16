<template>
  <button v-on:click="askPermission" id="nfc">Permission NFC</button>
  <p>
      <button v-on:click="readTag">Test NFC Read</button>
      <button v-on:click="writeTag">Test NFC Write</button>
  </p>
  <p>
    <label for='name'>Nom</label>
    <input id='name' v-model="message" placeholder="Nom animal">
  </p>
  <pre id="log"></pre>
</template>

<script>
export default {
  name: "App",
  data() {
    return {

    };
  },
  methods: {
    displayOutcome(type, outcome){
      return function() {
        var argList = [outcome, type].concat([].slice.call(arguments));
        switch(outcome) {
        case "success":
          console.info.apply(console, argList);
          break;
        case "error":
          console.error.apply(console, argList);
          break;
        default:
          console.log.apply(console, argList);
        }
        document.getElementById(type).classList.remove('success', 'error', 'default');
        document.getElementById(type).classList.add(outcome);
      };
    },
    consoleLog(data){
        var logElement = document.getElementById('log');
        logElement.innerHTML += data + '\n';
    },
    /*global NDEFReader*/
    async readTag(){
        if ("NDEFReader" in window) {
            const ndef = new NDEFReader();
            try {
            await ndef.scan();
            ndef.onreading = event => {
                const decoder = new TextDecoder();
                var json = " " ;
                var parsedJSON = " "; 
                for (const record of event.message.records) {
                  json = decoder.decode(record.data)
                  parsedJSON = JSON.parse(json)
                  this.consoleLog("=== data ===\n" + json);
                  this.consoleLog("=== data ===\n" + parsedJSON);
                  this.consoleLog("TOTOTITITATA")

                }
                
              }
            } catch(error) {
            this.consoleLog(error);
            }
        } else {
            this.consoleLog("Web NFC is not supported.");
        }
    },
    async writeTag(){
      if ("NDEFReader" in window) {
          const ndef = new NDEFReader();
          try {
            await ndef.write("{'id':12, 'name':'Felix'}");
            this.consoleLog("NDEF message written!");
          } catch(error) {
            this.consoleLog(error);
          }
      } else {
          this.consoleLog("Web NFC is not supported.");
      }
    },
    askPermission(){
        if ('NDEFReader' in window) {
            const reader = new NDEFReader();
            reader.scan()
            .then(() => {
            this.displayOutcome("nfc", "success")("Successfully started NFC scan");
            })
            .catch((err) => {
            this.displayOutcome("nfc", "error")(err);
            });
        } else {
            this.displayOutcome("nfc", "error")("NDEFReader is not available");
        }
    }
  }
}
</script>

<style lang="scss">

</style>
