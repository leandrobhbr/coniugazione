<template>
  <q-page>
    <div class="row q-px-xs">
      <div class="col-12 col-md-6 q-mt-md q-px-xs">
        <q-select
          outlined
          use-input
          input-debounce="0"
          v-model="verboScelto"
          label="500 Verbi in ordine alfabetico"
          :options="opzioniDelVerboAlfabetico"
          behavior="dialog"
          @filter="filterFn"
        >
          <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
               Mi dispiace, è stato scritto un errore o non ha trovato alcun risultato. Al momento abbiamo solo 500 verbi registrati. In futuro aumenteremo la quantità di verbi.
              </q-item-section>
            </q-item>
          </template>
          <template v-slot:prepend>
            <q-icon name="search" color="red" />
          </template>
        </q-select>
      </div>
      <div class="col-12 col-md-6 q-mt-md q-px-xs">
        <q-select
          outlined
          v-model="verboScelto"
          label="500 Verbi più cercati"
          :options="opzioniDelVerbo"
          behavior="menu"
        />
      </div>
      <div v-for="(data, index) in listaDeiVerbi" :key="index">
        <div v-if="data.verbi === verboScelto">
          <div class="q-gutter-sm q-pa-md">
            <b>Piu tempi verbali?</b>
            <q-radio v-model="piuTempiVerbale" val="si" label="sì" />
            <q-radio v-model="piuTempiVerbale" val="no" label="no" />
            <q-btn flat color="teal" icon="remove_red_eye" label="visualizzare le risposte" v-if="!vrisposte" @click="vrisposte = true" />
            <q-btn flat color="teal" icon="cloud_queue" label="nascondi le risposte" v-if="vrisposte" @click="vrisposte = false" />
          </div>
          <q-tab-panels v-model="piuTempiVerbale" animated class="shadow-2 rounded-borders q-mt-md q-ml-sm">
          <q-tab-panel name="si">
            <div class="row">
              <div class="col-12 col-md-12 q-mt-md q-px-xs" style="height: 100px; overflow: auto">
              <q-checkbox label="SELEZIONA TUTTI" v-model="chkTutti" color="orange" /><br/>
              <span v-for="(dataChk, indexChk) in data.coniugazione" :key="dataChk.id">
                <q-checkbox :id="'chk' + indexChk" v-model="chk" :val="dataChk.id" :label="'' + dataChk.modalita_verbale + ' - ' + dataChk.tempo_verbale + ''" /><br />
              </span>
              </div>
            </div>
          </q-tab-panel>
          </q-tab-panels>
          <div class="row">
            <div v-for="data1 in data.coniugazione" :key="data1.id">
              <!-- id verbi modalita_verbale tempo_verbale italiano portoghese -->
              <div class="q-pa-md" v-if="chkVerifica(data1.id)" style="min-width: 280px">
              <div class="q-pa-lg bg-grey-1 text-grey-9">{{ data1.modalita_verbale }} - {{ data1.tempo_verbale }}</div>
              <div v-for="(numeroFrase, index) in raccontareConiugazione(data1.italiano)" :key="index">
                <q-input
                  class="q-mt-md"
                  ref="input"
                  lazy-rules
                  v-model="input[data1.id + ritornaFrase(numeroFrase,data1.italiano)]"
                  stack-label
                  :hint="ritornaFrasePort(numeroFrase,data1.portoghese)"
                  :label="ritornaPersona(numeroFrase,data1.italiano)"
                  :rules="[val => val !== undefined && val.toLowerCase() === ritornaFrase(numeroFrase,data1.italiano) || ritornaFrase(numeroFrase,data1.italiano)]"
                />
                <div v-if="vrisposte === false && ritornaInput(input[data1.id + ritornaFrase(numeroFrase,data1.italiano)]) === ritornaFrase(numeroFrase,data1.italiano)">
                  <q-icon class="text-green q-mr-sm" name="thumb_up" /> <span class="text-green negrito" v-html="ritornaFraseConStile(numeroFrase,data1.italiano)"></span>
                </div>
                <div v-if="vrisposte">
                  <span class="text-green negrito" v-html="ritornaFraseConStile(numeroFrase,data1.italiano)"></span>
                </div>
              </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>
<style>
.negrito b { color: red; }
</style>
<script>
import verbi from 'assets/verbi.json'
export default {
  data () {
    return {
      chkTutti: false,
      listaDeiVerbi: verbi,
      verboScelto: null,
      coniugazioni: null,
      input: [],
      chk: [],
      stringOpzioni: this.opzioniInizioDelVerboOriginale(true),
      opzioniDelVerboAlfabetico: this.opzioniInizioDelVerboOriginale(true),
      opzioniDelVerbo: this.opzioniInizioDelVerboOriginale(),
      piuTempiVerbale: 'no',
      vrisposte: false
    }
  },
  watch: {
    verboScelto (newValue, oldValue) {
      this.input = []
    },
    chkTutti (newValue, oldValue) {
      if (newValue) {
        this.inizioChk(true)
        return
      }
      this.chk = []
    }
  },
  mounted () {
    this.inizioChk()
  },
  methods: {
    opzioniInizioDelVerboOriginale (alfabetico = false) {
      var opzioni2 = []
      Object.entries(verbi).forEach(([key, value]) => {
        opzioni2.push(`${value.verbi}`)
      })
      if (alfabetico) {
        opzioni2.sort(function (a, b) {
          return a.localeCompare(b)
        })
      }
      return opzioni2
    },
    raccontareConiugazione (value) {
      // console.log(value)
      // le coniugazioni sono separate da [BR]
      var account = (value.match(/\[br\]/g) || []).length
      // console.log(account)
      return account
    },
    inizioChk (tutti = false) {
      var chk1 = []
      Object.entries(this.listaDeiVerbi).forEach(([key, value]) => {
        Object.entries(value.coniugazione).forEach(([key1, value1]) => {
          if (tutti === true) {
            chk1.push(`${value1.id}`)
          } else if (value1.chk) {
            chk1.push(`${value1.id}`)
          }
        })
      })
      this.chk = chk1
    },
    ritornaFraseConStile (numeroFrase, frasi) {
      numeroFrase = numeroFrase - 1
      var res = frasi.split('[br]')
      var res1 = res[numeroFrase].replace('[b]', '<b>')
      var res2 = res1.replace('[|b]', '</b>')
      return res2
    },
    ritornaFrase (numeroFrase, frasi) {
      numeroFrase = numeroFrase - 1
      var res = frasi.split('[br]')
      var res1 = res[numeroFrase].replace('[b]', '')
      var res2 = res1.replace('[|b]', '')
      return res2
    },
    ritornaInput (valore) {
      if (valore !== undefined) {
        return valore.toLowerCase()
      }
      return valore
    },
    ritornaFrasePort (numeroFrase, frasi) {
      // console.log(frasi)
      if (frasi !== '') {
        return this.ritornaFrase(numeroFrase, frasi)
      }
      return null
    },
    ritornaPersona (numeroFrase, frasi) {
      numeroFrase = numeroFrase - 1
      var res = frasi.split('[br]')
      var res2 = res[numeroFrase].split(' ')
      if (res2[0] === 'che') { // che io
        return res2[0] + ' ' + res2[1]
      }
      if (res2[1] === undefined) { // coniugazione con una parola
        return ''
      }
      return res2[0]
    },
    chkVerifica (valore) {
      return this.chk.some(item => item === valore)
    },
    filterFn (val, update) {
      if (val === '') {
        update(() => {
          this.opzioniDelVerboAlfabetico = this.stringOpzioni
        })
        return
      }
      update(() => {
        const needle = val.toLowerCase()
        this.opzioniDelVerboAlfabetico = this.stringOpzioni.filter(v => v.toLowerCase().indexOf(needle) > -1)
      })
    }
  }
}
</script>
