<template>

    <div>
        <br><br><br>
        <div class="content col-md-6">
            <div class="voltar" v-if="show_classification">
                <b-link :to="{ name: 'Home'}">
                    <b-button size="sm" variant="primary">Voltar</b-button>
                </b-link>
            </div> 
            <div v-show="loading" id="loading">
                <i class="fa fa-spinner fa-pulse fa-3x fa-fw" ></i>
                <span>...coletando tweets</span>
            </div>            
            <!-- só exibira a div se uma pesquisa for realizada -->
            <div v-if="display && search != null" class="panel panel-info">
                                
                <div class="panel-heading">
                    <h4 class="media-heading">Busca: <strong>{{search}}</strong></h4>
                    <div v-if="show_classification">
                        <h5 class="media-heading">Total coletados: <strong style="color: blue">{{tweets_size}}</strong></h5>
                        <h5 class="media-heading">Classificações Positivas: <strong style="color: blue">{{cont_pos}}</strong> ({{porcent_pos}}%)</h5>
                        <h5 class="media-heading">Classificações Negativas: <strong style="color: red">{{cont_neg}}</strong> ({{porcent_neg}}%)</h5>
                    </div>
                    
                </div>

                <div v-for="(tweet, index) in tweets" :key="tweet.id" class="media-body">
                    <p>
                        <span v-if="tweet.classification == 1">
                            Classificação: <strong style="color: blue">Positiva</strong>
                        </span>
                        <span v-else>
                            Classificação: <strong style="color: red">Negativa</strong>
                        </span>
                    </p>
                    <span class="user">@{{tweet.user}}</span>
                    <div class="tweet">{{tweet.tweet_text}}</div>
                    <div class="correcao">
                        <span v-if="tweet.classification == 1">
                            Alterar classificação: <a @click="toNegative(index, tweet.id)" class="nav-link" href="javascript:void(0)" title="Alterar para negativo"><i class="fas fa-thumbs-down" style="color: red"></i></a>
                        </span>
                        <span v-else>
                            Alterar classificação:<a @click="toPositive(index, tweet.id)" class="nav-link" href="javascript:void(0)" title="Alterar para positivo"><i class="fas fa-thumbs-up" style="color: blue"></i></a>
                        </span>
                    </div>
                    
                </div>

            </div>

            <div v-else>
                <b-alert show variant="danger"><strong>Ops!</strong> É preciso realizar uma busca para coletar e classificar os tweets.</b-alert>
                <router-link :to="{ name: 'Home'}">
                    <button class="btn btn-primary" type="submit">Voltar</button>
                </router-link>                
            </div>

            <div v-if="error_twitter" class="twitter_error">
                <b-alert show variant="danger"><strong>Ops!</strong> Ocorreu uma falha de comunicação com o Twitter. Tente novamente.</b-alert>
                <router-link :to="{ name: 'Home'}">
                    <button class="btn btn-primary" type="submit">Voltar</button>
                </router-link>                
            </div>
            
        </div>
    </div>

</template>

<script>
    import axios from 'axios'

    export default {
        name: 'Classificados',
        created() {
            this.search = this.$route.params.search_id;
            if(this.$route.query.display && this.search != "") {           
                this.loading = true;
                axios.get(this.url + "?search=" + this.search).then(tweets => {
                    this.tweets = tweets.data;
                    this.tweets_size = this.tweets.length
                    
                    for(let tweet of this.tweets) {
                        if(tweet.classification == 0) {
                            this.cont_neg++
                        }
                        else {
                            this.cont_pos++
                        }
                    }
                    this.show_classification = true
                    this.loading = false
                    this.setPercent()
                }).catch(response => {
                    this.error_twitter = true
                    this.loading = false
                })
                this.display = this.$route.query.display;
            }           
        },
        data () {
            return {
                display : false,
                search: null,
                loading: false,
                tweets: [],
                /* url: "https://analise-sentimento.herokuapp.com/tweets/", */
                url: "http://localhost:8000/tweets/",
                error_twitter: false,
                show_classification: false,
                cont_pos: 0,
                cont_neg: 0,
                tweets_size: 0,
                porcent_pos: 0,
                porcent_neg: 0
        }
    },
        methods: {
            toPositive: function(index, id) {
                axios.patch(this.url + id + "/", {"classification": 1}).then(response => {
                    let change = this.tweets[index]
                    change.classification = "1"
                    this.tweets.splice(index, 1, change)
                    this.cont_neg--
                    this.cont_pos++
                    this.setPercent()
                }).catch(error => {
                    alert("Classificação não alterada. Tente novamente.")
                })
            },
            toNegative: function(index, id) {
                axios.patch(this.url + id + "/", {"classification": 0}).then(response => {
                    let change = this.tweets[index]
                    change.classification = "0"
                    this.tweets.splice(index, 1, change)
                    this.cont_neg++
                    this.cont_pos--
                    this.setPercent()
                }).catch(error => {
                    alert("Classificação não alterada. Tente novamente.")
                })
            },
            setPercent: function() {
                this.porcent_pos = ((this.cont_pos / this.tweets_size) * 100).toFixed(2)
                this.porcent_neg = ((this.cont_neg / this.tweets_size) * 100).toFixed(2)
            }
        }
    }
</script>

<style scoped>

    ul li a:link, a:visited {
        text-decoration: none;
    }
    #loading span {
        font-size: 35px;
    }

    #loading {
        text-align: center
    }
    .content {
       /*  margin-top: 70px; */
        margin-left: auto;
        margin-right: auto;
    }

    .media-body {
        background-color: white;
        padding-top: 10px;
        padding-left: 5px;
        padding-right: 5px;
    }

    .panel-info {
        border: 1px grey solid;
    }

    div.panel-heading{
        padding: 10px;
        background-color: #d9edf7
    }
    
    .tweet {
        font-style: italic;
    }

    .user {
        font-weight: bolder;
    }

    .media-body {
        border: 1px lightgrey solid;
    }

    .correcao {
        padding-top: 10px;
        padding-bottom: 10px;
    }
    .twitter_error {
        margin-top: 10px;
    }

    .voltar {
        margin-bottom: 10px;
    }
</style>
