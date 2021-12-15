<template>
  <button v-on:click="askPermission" id="nfc">Permission NFC</button>
  <p>
      <button v-on:click="readTag">Test NFC Read</button>
      <button v-on:click="writeTag">Test NFC Write</button>
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
                for (const record of event.message.records) {
                this.consoleLog("Record type:  " + record.recordType);
                this.consoleLog("MIME type:    " + record.mediaType);
                this.consoleLog("=== data ===\n" + decoder.decode(record.data));
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
            await ndef.write({
              records: [{ recordType: "url", data: "http://example.com/" }]
            });
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
