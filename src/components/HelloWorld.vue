<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="6">
        <v-text-field label="Lånebeløp: " v-model="laanebelop"></v-text-field>
        <v-text-field label="Løpetid: " v-model="lopetid" ></v-text-field>

         <v-menu v-model="fromDateMenu" :close-on-content-click="false">
          <template v-slot:activator="{ on }">
            <v-text-field label="Startdato: " v-model="saldoDato" v-on="on"></v-text-field>
          </template>
          <v-date-picker v-model="saldoDato" elevation="15" @input="fromDateMenu = false" ></v-date-picker>
        </v-menu>
      </v-col>
      <v-col cols="6">
        <v-text-field label="Termingebyr: " v-model="terminGebyr" ></v-text-field>
        <v-text-field label="Nominell rente: " v-model="nominellRente" ></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col align="center">
        <v-btn color="base" type="submit" @click="makePostRequest()">Regn ut</v-btn><br><br>
        <v-simple-table v-if = "innbetalinger.length>0">
          <template v-slot: default>
            <thead>
              <tr>
                <th class="text-left">
                  Restgjeld
                </th>
                <th class="text-left">
                  Dato
                </th>
                <th class="text-left">
                  Innbetaling
                </th>
                <th class="text-left">
                  Gebyr
                </th>
                <th class="text-left">
                  Renter
                </th>
                <th class="text-left">
                  Total
                </th>
              </tr>
            </thead>
            <tbody>
              <tr
                v-for = "innbetaling in getInnbetalinger()"
                :key = "innbetaling.dato">
                <td>{{Math.round(innbetaling.restgjeld)}}</td>
                <td>{{innbetaling.dato}}</td>
                <td>{{Math.round(innbetaling.innbetaling)}}</td>
                <td>{{innbetaling.gebyr}}</td>
                <td>{{Math.round(innbetaling.renter)}}</td>
                <td>{{Math.round(innbetaling.total)}}</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class HelloWorld extends Vue {
  private innbetalinger = [];
  private laanebelop = "2000000";
  private nominellRente = "1.5";
  private terminGebyr = "30";
  private saldoDato = new Date().toISOString().slice(0, 10);
  private ukjentVerdi = "TERMINBELOP";
  private lopetid = "25";
  private fromDateMenu = false;

  getInnbetalinger() {
    return this.innbetalinger;
  }

  async makePostRequest() {
    let parts = this.saldoDato.split('-');
    let date = new Date(parseInt(parts[0]), parseInt(parts[1]) - 1, parseInt(parts[2]));
    let year = date.getFullYear();
    let utlopsDatoStr= new Date(year + parseInt(this.lopetid), date.getMonth(), date.getDate() + 1);
    let utlopsDato = utlopsDatoStr.toISOString().slice(0, 10);
    let datoForsteInnbetalingStr = new Date(year, date.getMonth() + 1, date.getDate() + 1);
    let datoForsteInnbetaling = datoForsteInnbetalingStr.toISOString().slice(0, 10);

    let req = {
      laanebelop: this.laanebelop,
      nominellRente: this.nominellRente.replace(',', '.'),
      terminGebyr: this.terminGebyr,
      utlopsDato,
      saldoDato: this.saldoDato,
      datoForsteInnbetaling,
      ukjentVerdi: this.ukjentVerdi,
    };

    let res = await fetch("https://visningsrom.stacc.com/dd_server_laaneberegning/rest/laaneberegning/v1/nedbetalingsplan",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(req),
      }
    );
    if (res.ok) {
      let resObject = await res.json();
      this.innbetalinger = resObject.nedbetalingsplan.innbetalinger;
    }
  }
}
</script>
