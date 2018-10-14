<template>
  <div class="content">
    <br><br><br>
    
    <b-table :items="searchs" :fields="fields" v-if="show_table">
      <template slot="show_details" slot-scope="tweet">
        <b-button size="sm" class="mr-2" @click="getTweets(tweet.item.id)">
          Mostrar Tweets
        </b-button>
      </template>
    </b-table>

    <div v-show="loading_table" id="loading_table">
      <i class="fa fa-spinner fa-pulse fa-3x fa-fw" ></i>
      <span>...carregando</span>
    </div>

    <div v-if="!show_table">
      <div v-show="!loading">
        <b-button size="sm" class="" @click="backTable" variant="primary">Voltar</b-button>
      </div>
      
      <div class="header" v-show="!loading">
        <h5 class="media-heading">Total de tweets coletados: <strong>{{total}}</strong></h5>
        <h5 class="media-heading">Total de classificações alteradas: <strong>{{total_changes}}</strong></h5>
        <h5 class="media-heading">Classificações Positivas: <strong style="color: blue">{{cont_pos}}</strong> ({{porcent_pos}}%)</h5>
        <h5 class="media-heading">Classificações Negativas: <strong style="color: red">{{cont_neg}}</strong> ({{porcent_neg}}%)</h5>
      </div>

      <div v-show="loading" id="loading">
          <i class="fa fa-spinner fa-pulse fa-3x fa-fw" ></i>
          <span>...carregando</span>
      </div>

      <b-card v-for="t in tweet_data" :key="t.id">
          <b-row class="mb-2">
            <b-col sm="3" class="text-sm-right"><b>Usuário:</b></b-col>
            <b-col>@{{ t.user }}</b-col>
          </b-row>
          <b-row class="mb-2">
            <b-col sm="3" class="text-sm-right"><b>Tweet:</b></b-col>
            <b-col>{{ t.tweet_text }}</b-col>
          </b-row>
          <b-row class="mb-2">
            <b-col sm="3" class="text-sm-right"><b>Classificação:</b></b-col>
            <b-col v-if="t.classification == 1">
              <i class="fas fa-thumbs-up" style="color: blue"></i>
            </b-col>
            <b-col v-else>
              <i class="fas fa-thumbs-down" style="color: red"></i>
            </b-col>
          </b-row>
          <b-row class="mb-2">
            <b-col sm="3" class="text-sm-right"><b>Alterada:</b></b-col>
            <b-col>
              <div v-if="t.change == true">Sim</div>
              <div v-else>Não</div>
            </b-col>
          </b-row>
        </b-card>
    </div>

  </div>
</template>

<script>
  import axios from 'axios'
  import moment from 'moment'
  export default {
    created() {
      axios.get(this.url + "buscas/").then(response => {
        this.searchs = response.data;
        this.loading_table = false
      })
    },
    data () {
      return {
        fields: {
            expression: {
              sortable: false,
              label: 'Expressão'
            },
            search_date: {
              sortable: true,
              label: 'Data da Busca',
              formatter: (value, key, item) => {
                return moment(String(value)).format('DD/MM/YYYY HH:mm')
              }
            },
            show_details: {
              label: 'Tweets'
            }
          },
        searchs: [],
        url: "https://analise-sentimento.herokuapp.com/",
        /* url: "http://localhost:8000/", */
        tweet_data: [],
        show_table: true,
        total: 0,
        cont_pos: 0,
        cont_neg: 0,
        porcent_pos: 0,
        porcent_neg: 0,
        total_changes: 0,
        loading: true,
        loading_table: true
      }
    },
    methods: {
      getTweets: function(id) {
        this.show_table = false
        axios.get(this.url + "tweets/?search_id=" + id).then(response => {
          this.tweet_data = response.data
          for(let tweet of this.tweet_data) {
            if(tweet.classification == 0) {
                this.cont_neg++
            }
            else {
                this.cont_pos++
            }
            if(tweet.change) {
              this.total_changes++
            }
          }
          this.total = this.tweet_data.length
          this.setPercent()
          this.loading = false
        })
      },
      backTable: function() {
        this.total = 0
        this.cont_pos = 0
        this.cont_neg = 0
        this.porcent_pos = 0
        this.porcent_neg = 0
        this.total_changes = 0
        this.show_table = true
      },
      setPercent: function() {
        this.porcent_pos = ((this.cont_pos / this.total) * 100).toFixed(2)
        this.porcent_neg = ((this.cont_neg / this.total) * 100).toFixed(2)
      }
    }
  }
</script>

<style>
  .content {
    width: 60%;
    /* margin-top: 70px; */
    margin-left: auto;
    margin-right: auto;
  }
  .positivo {
    color: blue;
  }
  .negativo {
    color: red;
  }
  h3 {
    text-align: center;
  }
  .header {
    margin-top: 15px;
    margin-bottom: 15px;
    padding-top: 5px;
    padding-left: 10px;
    border: 1px solid
  }
  #loading {
    margin-bottom: 10px
  }
</style>
