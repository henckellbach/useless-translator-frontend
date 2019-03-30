<template>
  <main class="main">
    <div class="input-strip">
      <div class="input-strip__inner">
        <h1 class="input-strip__heading">Useless Translator</h1>
        <b-form-textarea
          v-model="inputText"
          placeholder="Enter your text"
          rows="6"
          max-rows="10"
          :disabled="isProcessing"
          :state="inputState"
          @input="resetInputState"
        />
        <div class="input-strip__controls">
          <b-button
            variant="primary"
            @click="translate"
            :disabled="isProcessing"
          >Translate</b-button>
          <span
            class="input-strip__help"
            v-b-modal.helpModal
          >
            ?
          </span>
        </div>
      </div>
    </div>
    <b-modal
      id="helpModal"
      title="About this service"
      ok-only="true"
    >
      <p>This is a useless translation service. It takes your English input and translates it back and forth to 3 random languages.</p>
    </b-modal>
    <div class="translations">
      <div class="translations__loader" v-if="isProcessing">
        <div class="loader" />
      </div>
      <ul class="translations__list">
        <li
          class="translations__item"
          v-for="(val, key) in translations"
          :key="key"
        >
          <div class="translations__meta">From {{ val.from.name }} to {{ val.to.name }}</div>
          <div class="translations__text">
            <span class="translations__text-original">{{ val.text }}</span>
            <span class="translations__text-transliterated">{{ transliterate(val.text) }}</span>
          </div>
        </li>
      </ul>
    </div>
  </main>
</template>

<script>
import { transliterate } from 'transliteration'

export default {
  data () {
    return {
      inputText: '',
      translations: [],
      isProcessing: false,
      inputState: null // null = default, true = invalid, false = valid
    }
  },
  methods: {
    translate () {
      if (!this.inputText.trim()) {
        this.inputState = false
        return
      }

      this.translations = []
      this.isProcessing = true

      const text = encodeURI(this.inputText.trim())
      const lang = 'en'
      const apiUrl = process.env.NODE_ENV === 'production' ? process.env.VUE_APP_API_URL : process.env.VUE_APP_API_LOCAL_URL
      const url = `${apiUrl}/?lang=${lang}&text=${text}`

      fetch(url)
        .then((response) => {
          response.json().then(data => {
            this.translations = data
            this.isProcessing = false
          })
        })
    },
    resetInputState () {
      this.inputState = null
    },
    transliterate (text) {
      const tr = transliterate(text)
      return tr === text ? null : tr
    }
  },
  mounted () {
    document.title = 'Useless Translator'
  }
}
</script>

<style lang="scss">
@import "../assets/scss/components/input-strip";
@import "../assets/scss/components/loader";
@import "../assets/scss/components/translations";
</style>
