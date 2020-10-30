<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="6">
        <v-text-field label="Laanebeløp: " v-model = "laanebelop" ></v-text-field>
        <v-text-field label="Rente: "></v-text-field>
      </v-col>
      <v-col cols="6">
        <v-text-field label="Startdato: "></v-text-field>
        <v-text-field label="Innfrielsesdato: "></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col align="center">
        <v-label>{{heading}}</v-label><br>
        <v-btn color="base" type="submit" @click="makePostRequest()">Press me</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class HelloWorld extends Vue {
  private the_head = "Synne";
  private laanebelop = "2000000";
  private nominellRente = 3;
  private terminGebyr = 30; 
  private utlopsDato = "2045-01-01";
  private saldoDato = "2020-01-01";
  private datoForsteInnbetaling = "2020-02-01";
  private ukjentVerdi = "TERMINBELOP";

  get heading() {
    return this.the_head;
  }

  setHead() {
    this.the_head = "Steinar";
  }

  async makePostRequest() {

    let req = {
      laanebelop: this.laanebelop,
      nominellRente: this.nominellRente,
      terminGebyr: this.terminGebyr,
      utlopsDato: this.utlopsDato,
      saldoDato: this.saldoDato,
      datoForsteInnbetaling: this.datoForsteInnbetaling,
      ukjentVerdi: this.ukjentVerdi
    }

    let res = await fetch("https://visningsrom.stacc.com/dd_server_laaneberegning/rest/laaneberegning/v1/nedbetalingsplan", {
      method: "POST",
      headers: {
				'Content-Type': 'application/json'
			},
      body: JSON.stringify(req)
    });
    if (res.ok) {
      let resObject = await res.json();
      console.log(resObject);

    }
    
    }
}
</script>
