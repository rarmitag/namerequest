<template>
  <MainContainer id="quick-search-results-container">
    <template v-slot:container-header>
      <v-col cols="auto" class="h6 py-0 mt-1">
        You are searching for a name for a
        {{ entityText === ' BC Corporation' && location.text === ' BC' ? '' : ' ' + location.text }}
        {{ entityText }}
      </v-col>
    </template>
    <template v-slot:content>
      <v-row no-gutters class="pt-3">
        <NameInput/>
      </v-row>
      <v-row justify="center" class="ma-0 pa-0">
        <v-col cols="12" class="quick-search-title">
          <b>Quick Search Results</b>
        </v-col>
      </v-row>
      <v-row v-if="namesList.length" justify="center" class="ma-0 pa-0">
        <v-col cols="12" class="quick-search-info">
          We found <b>{{namesList.length}}</b> corporations using these, or similar, words.
        </v-col>
        <v-col cols="12" class="quick-search-info" v-if="location.value==='BC'">
          Research your name here to see if other businesses are using a similar name within
          the same business category. Click continue below to submit up to 3 names.
        </v-col>
        <v-col cols="12" class="quick-search-info" v-else>
          You must use your legal business name in your home jurisdiction, unless your name is too
          similar to an existing BC corporation then you may need an assumed name to operate in BC.
          Click continue below and you will have an option to add 2 assumed names to your submission.
        </v-col>
        <v-col cols="12" class="names-list">
          <v-tooltip right transition="fade-transition" content-class="tooltip">
            <template v-slot:activator="scope">
              <div v-on="scope.on">
                <QuickSearchNames/>
              </div>
            </template>
            <span>
              These results show corporations only and do not include sole proprietorships,
              DBA's or partnerships. This is solely for information purposes, and can help
              you choose a more unique name to submit. You should also research your choices
              by searching the internet, social media and relevant domain names if that is
              important to you.
            </span>
          </v-tooltip>
        </v-col>
      </v-row>
      <v-row v-else justify="center">
        <v-col cols="12" class="quick-search-info">
          We did not find any names using these exact or similar words within the same business category.
        </v-col>
      </v-row>
      <v-row justify="center">
        <v-col cols="auto" class="pb-0 pr-1">
          <v-btn id="search-again-button"
                 :disabled="nameChanged"
                 @click="searchAgain()">Search Again</v-btn>
        </v-col>
        <v-col cols="auto" class="pb-0 pr-1">
          <v-btn id="continue-search-button"
                 color="primary"
                 outlined
                 @click="detailedSearch()"
                 :loading="isLoadingAnalysis">Continue to Detailed Analysis</v-btn>
        </v-col>
      </v-row>
    </template>
  </MainContainer>
</template>

<script lang="ts">
import MainContainer from '@/components/new-request/main-container.vue'
import NameInput from '@/components/new-request/name-input.vue'
import QuickSearchNames from '@/components/new-request/quick-search-names.vue'
import allDesignations, { allDesignationsList } from '@/store/list-data/designations'
import newReqModule from '@/store/new-request-module'
import { Component, Vue, Watch } from 'vue-property-decorator'

@Component({
  components: { MainContainer, NameInput, QuickSearchNames }
})
export default class QuickSearchResults extends Vue {
  config = {
    modules: {
      toolbar: false
    },
    placeholder: '',
    scrollingContainer: false
  }
  contents: string = ''
  originalName: string = ''
  private isLoadingAnalysis: boolean = false

  get entity_type_cd () {
    return newReqModule.entity_type_cd
  }
  get entityText () {
    return newReqModule.entityTextFromValue
  }
  get location () {
    let value = newReqModule.location
    let options = newReqModule.locationOptions
    return options.find((opt: any) => opt.value === value)
  }
  get namesList () {
    return newReqModule.quickSearchNames
  }
  get searchName () {
    return newReqModule.name
  }
  get nameChanged () {
    return this.originalName === this.searchName
  }
  mounted () {
    this.originalName = this.searchName
    this.$nextTick(() => {
      // add classname to button text (for more detail in Sentry breadcrumbs)
      const searchAgainBtn = this.$el.querySelector("#search-again-button > span")
      if (searchAgainBtn) searchAgainBtn.classList.add("search-again-btn")
      const continueDetailedAnalysisBtn = this.$el.querySelector("#continue-search-button > span")
      if (continueDetailedAnalysisBtn) continueDetailedAnalysisBtn.classList.add("continue-detailed-analysis-btn")
    })
  }
  async searchAgain () {
    newReqModule.mutateQuickSearch(true)
    await newReqModule.startAnalyzeName()
  }
  async detailedSearch () {
    this.isLoadingAnalysis = true
    newReqModule.mutateNoCorpNum(true)
    newReqModule.mutateQuickSearch(false)
    await newReqModule.startAnalyzeName()
    this.isLoadingAnalysis = false
  }
}
</script>

<style scoped lang="scss">
@import '@/assets/scss/theme.scss';
#search-again-button.v-btn[disabled] {
  opacity: 0.4 !important;
  color: white !important;
  background-color: #1669bb !important;
}
.quick-search-title {
  font-size: 1rem;
  color: $gray9;
  text-align: center;
  padding: 0;
  margin: 0;
}
.quick-search-info {
  font-size: 0.875rem;
  color: $gray7;
  text-align: center;
  padding: 0;
  margin: 0;
  padding-top: 0.3125rem;
}
.names-list {
  padding-top: 0.625rem;
}
#continue-search-button {
  background-color: #fff !important;
}
</style>
