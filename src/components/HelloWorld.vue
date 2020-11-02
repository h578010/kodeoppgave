<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="6">
        <v-text-field label="Lånebeløp: " v-model="laanebelop" :rules="[required, numbers, positive]" ></v-text-field>
        <v-text-field label="Løpetid: " v-model="lopetid" :rules="[required, numbers, positive]" ></v-text-field>
        <v-menu v-model="fromDateMenu" :close-on-content-click="false">
          <template v-slot:activator="{ on }">
            <v-text-field label="Startdato: " readonly v-model="saldoDato" v-on="on" ></v-text-field>
          </template>
          <v-date-picker v-model="saldoDato" elevation="15" @input="fromDateMenu = false" ></v-date-picker>
        </v-menu>
      </v-col>
      <v-col cols="6">
        <v-text-field label="Termingebyr: " v-model="terminGebyr" :rules="[required, numbers, positive]" ></v-text-field>
        <v-text-field label="Nominell rente: " v-model="nominellRente" :rules="[required, numbers, positive]"></v-text-field>
      </v-col>
    </v-row>
    <v-row>
      <v-col align="center">
        <v-btn color="success" type="submit" @click="makePostRequest()">Regn ut</v-btn><br /><br />
        <v-tabs v-model="tab">
          <v-tabs-slider color="success"></v-tabs-slider>
          <v-tab>Månedlige avdrag</v-tab>
          <v-tab>Årlige avdrag</v-tab>
        </v-tabs>
        <v-tabs-items v-model="tab">
          <v-tab-item>
            <v-simple-table v-if="innbetalinger.length > 0">
              <template v-slot: default>
                <thead>
                  <tr>
                    <th class="text-left">Dato</th>
                    <th class="text-left">Restgjeld</th>
                    <th class="text-left">Innbetaling</th>
                    <th class="text-left">Gebyr</th>
                    <th class="text-left">Renter</th>
                    <th class="text-left">Total</th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="innbetaling in getInnbetalinger()"
                    :key="innbetaling.dato"
                  >
                    <td>{{ innbetaling.dato }}</td>
                    <td>{{ Math.round(innbetaling.restgjeld) }}</td>
                    <td>{{ Math.round(innbetaling.innbetaling) }}</td>
                    <td>{{ innbetaling.gebyr }}</td>
                    <td>{{ Math.round(innbetaling.renter) }}</td>
                    <td>{{ Math.round(innbetaling.total) }}</td>
                  </tr>
                </tbody>
              </template>
            </v-simple-table>
          </v-tab-item>
          <v-tab-item>
            <v-simple-table v-if = "innbetalinger.length>0">
            <template v-slot: default>
              <thead>
                <tr>
                  <th class="text-left">Dato</th>
                  <th class="text-left">Restgjeld</th>
                  <th class="text-left">Innbetaling</th>
                  <th class="text-left">Gebyr</th>
                  <th class="text-left">Renter</th>
                  <th class="text-left">Total</th>
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for = "oppsummering in getOppsummering()"
                  :key = "oppsummering.dato">
                  <td>{{oppsummering.dato.slice(0,4)}}</td>
                  <td>{{Math.round(oppsummering.restgjeld)}}</td>
                  <td>{{Math.round(oppsummering.innbetaling)}}</td>
                  <td>{{oppsummering.gebyr}}</td>
                  <td>{{Math.round(oppsummering.renter)}}</td>
                  <td>{{Math.round(oppsummering.total)}}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
          </v-tab-item>
        </v-tabs-items>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

interface Innbetaling {
  restgjeld: number;
  dato: string;
  innbetaling: number;
  gebyr: number;
  renter: number;
  total: number;
}

@Component
export default class HelloWorld extends Vue {
  private tab = 0;
  private innbetalinger: Innbetaling[] = [];
  private oppsummering: Innbetaling[] = [];
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
  getOppsummering() {
    return this.oppsummering;
  }

  // Rules:
  required(value: string) {
    return value.length > 0 || "Vennligst oppgi en verdi.";
  }
  positive(value: string) {
    return parseInt(value) > 0 || "Vennligst oppgi en verdi høyere enn null.";
  }
  numbers(value: string) {
    return !isNaN(parseInt(value)) || "Vennligst oppgi en numerisk verdi.";
  }

  async makePostRequest() {
    let parts = this.saldoDato.split("-");
    let date = new Date(
      parseInt(parts[0]),
      parseInt(parts[1]) - 1,
      parseInt(parts[2])
    );
    let year = date.getFullYear();
    let utlopsDatoStr = new Date(
      year + parseInt(this.lopetid),
      date.getMonth(),
      date.getDate() + 1
    );
    let utlopsDato = utlopsDatoStr.toISOString().slice(0, 10);
    let datoForsteInnbetalingStr = new Date(
      year,
      date.getMonth() + 1,
      date.getDate() + 1
    );
    let datoForsteInnbetaling = datoForsteInnbetalingStr
      .toISOString()
      .slice(0, 10);

    let req = {
      laanebelop: parseInt(this.laanebelop),
      nominellRente: parseInt(this.nominellRente.replace(",", ".")),
      terminGebyr: parseInt(this.terminGebyr),
      utlopsDato,
      saldoDato: this.saldoDato,
      datoForsteInnbetaling,
      ukjentVerdi: this.ukjentVerdi,
    };

    let res = await fetch("https://visningsrom.stacc.com/dd_server_laaneberegning/rest/laaneberegning/v1/nedbetalingsplan", {
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
      let hashmap = new Map<string, Innbetaling>();
      this.innbetalinger.forEach((inn: Innbetaling) => {
        let year = inn.dato.slice(0, 4);
        let current = hashmap.get(year);
        if (current == undefined) {
          hashmap.set(year, {...inn});
        } else {
          current.innbetaling += inn.innbetaling;
          current.gebyr += inn.gebyr;
          current.renter += inn.renter;
          current.total += inn.total;
          current.restgjeld = inn.restgjeld; // Forutsetter at APIet leverer innbetalingene i kronologisk rekkefølge
        }
      });
      this.oppsummering = [];
      hashmap.forEach((opps) => {
        this.oppsummering.push(opps);
      })
    } else {
      alert("Det er problemer med å oppnå kontakt med serveren. Vennligst prøv igjen senere.");
    }
  }
}
</script>
