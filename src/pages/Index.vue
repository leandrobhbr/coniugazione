<template>
  <q-page>
    <div class="row q-px-xs">
      <div class="col-12 col-md-12 q-mt-md q-px-xs">
        <q-select
          outlined
          v-model="verboScelto"
          label="Verbi più cercati"
          :options="opzioniDelVerbo"
          behavior="menu"
        />
      </div>
      <div v-for="(data, index) in listaDeiVerbi" :key="index">
        <div v-if="data.verbi === verboScelto">
          <div class="row">
            <div v-for="(data1, index1) in data.coniugazione" :key="index1" class="col-12 col-md-4 q-mt-md q-px-xs">
              <!-- id verbi modalita_verbale tempo_verbale italiano portoghese -->
              <div class="q-pa-md">
              <div class="q-pa-lg bg-grey-1 text-grey-9">{{ data1.modalita_verbale }} - {{ data1.tempo_verbale }}</div>
              <div v-for="(numeroFrase, index2) in raccontareConiugazione(data1.italiano)" :key="index2">
                <q-input
                  class="q-mt-md"
                  ref="input"
                  lazy-rules
                  v-model="input[numeroFrase + index1 + index2 + ritornaPersona(numeroFrase,data1.italiano)]"
                  stack-label
                  :hint="ritornaFrasePort(numeroFrase,data1.portoghese)"
                  :label=ritornaPersona(numeroFrase,data1.italiano)
                  :rules="[val => val === ritornaFrase(numeroFrase,data1.italiano) || ritornaFrase(numeroFrase,data1.italiano)]"
                />
              </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import verbi from 'assets/verbi.json'
export default {
  data () {
    return {
      listaDeiVerbi: verbi,
      verboScelto: null,
      coniugazioni: null,
      input: []
    }
  },
  computed: {
    opzioniDelVerbo () {
      var opzioni = []
      Object.entries(this.listaDeiVerbi).forEach(([key, value]) => {
        // console.log(`${key} ${value.id} ${value.verbi}`)
        opzioni.push(`${value.verbi}`)
      })
      opzioni.sort(function (a, b) {
        return a.localeCompare(b)
      })
      return opzioni
    }
  },
  methods: {
    raccontareConiugazione (value) {
      // console.log(value)
      // le coniugazioni sono separate da [BR]
      var account = (value.match(/\[br\]/g) || []).length
      // console.log(account)
      return account
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
    ritornaFrasePort (numeroFrase, frasi) {
      console.log(frasi)
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
    }
  }
}
</script>
