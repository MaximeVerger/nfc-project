<template>

<header>
    <h1>Master finder</h1>
    <img src="" alt="Logo"/>
</header>

<main>
    <h2>Présentation</h2>
    <article>
        <p>
            Ce site a pour fonction de vous aider à retrouver le propriétaire d'un animale, 
            et de vous permettre d'entrer vos informations et celle de l'animale sur une puce NFC
            pour pouvoir vous joindre si quelqu'un d'autre le trouve.
        </p>
        <p>
            ATTENTION : Ce site ne fonctionne qu'avec le navigateur Chrome !
        </p>
    </article>

    <h2>Lire la puce d'un animale</h2>
    <article>
        <label>Il faut appuyer puis placer la puce derrière votre smartphone</label>
        <button v-on:click="readTag">Lire</button>
    </article>

    <h2>Ecrire sur la puce d'un animale</h2>
    <article>
        <label>Il faut appuyer puis placer la puce derrière votre smartphone</label>
        <label>Ne surtout pas écrire sur la puce d'un animale qui n'est pas le votre !</label>
        <form @submit.prevent="writeTag">
            <label for="name">Nom de l'animale : </label>
            <input id="name" required v-model="newName" />
            <label for="masterName">Nom de famille : </label>
            <input id="masterName" required v-model="masterName" />
            <label for="birthday">Date de naissance : </label>
            <input id="birthday" required v-model="birthday" />
            <label for="phone">Téléphone du propriétaire : </label>
            <input id="phone" required v-model="phone" />
            <button>Ecrire</button>
        </form>
        <!-- <button v-on:click="readTag">Ecrire</button> -->
    </article>

    <h2>Lecture de puce NFC</h2>
    <article>
        <label>Veuillez prendre en compte les faits suivants : </label>
        <ul>
            <li> Votre smartphone ne peut pas forcément lire les puces NFC </li>
            <li> Vous devez utiliser le navigateur Chrome </li>
            <ul> Pour activer la permission de votre téléphone en cas de problème
                <li>Essayez de l'activer manuellement sur votre téléphone</li>
                <li>Essayez d'appuyer sur le bouton "Permission NFC" ci dessous</li>
            </ul>
        </ul>
        <button v-on:click="askPermission" id="nfc">Permission NFC</button>
    </article>

    <!-- <p>
        <button v-on:click="writeTag">Test NFC Write</button>
    </p> -->

    <pre id="name"></pre>
    <pre id="lastname"></pre>
    <pre id="birthday"></pre>
    <pre id="phone"></pre>
    <pre id="log"></pre>
</main>

<footer>
</footer>

</template>

<script>
export default {
  name: "App",
  data() {
    return {
        newName: "",
        masterName: "",
        birthday: "",
        phone: ""
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
                var json = " ";
                for (const record of event.message.records) {
                  json = decoder.decode(record.data);
                  this.printData(json);
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
              records: [
                { recordType: "text", data: "name " + this.newName },
                { recordType: "text", data: "lastname " + this.masterName },
                { recordType: "text", data: "birthday " + this.birthday},
                { recordType: "text", data: "phone " + this.phone }
              ],
            });
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
    },
    printData(data){
      var parsed = data.split(" ");
      var Element = document.getElementById(parsed[0]);
      Element.innerHTML += parsed[1]
    }
  }
}
</script>

<style lang="scss">

</style>
