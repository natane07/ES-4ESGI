<template>
    <v-row>
        <v-col class="mb-4">
            <v-input messages="Nombre d'heure de cours">{{ sum_nb_heure }}</v-input>
        </v-col>
        <v-col class="mb-4">
            <v-input messages="Nombre de matiére">{{ nb_matiere }}</v-input>
        </v-col>
        <v-col class="mb-4">
            <v-input messages="Moyenne d'heure par matiére">{{ moyene_heure }}</v-input>
        </v-col>
        <v-col class="mb-4">
            <v-input messages="Nombre d'intervenant distinct">{{ inter_distinct }}</v-input>
        </v-col>
        <v-col class="mb-4">
            <v-input messages="Fatal error">Input</v-input>
        </v-col>       
    </v-row>
</template>

<script>
import axios from 'axios'
export default {
    name: 'Statistique',
    data () {
        return {
            sum_nb_heure: 0,
            nb_matiere: 0,
            moyene_heure: 0,
            inter_distinct: 0,
        }
    },
    created: function () {
        this.calculate_stat()
    },
    methods: {
        calculate_stat() {
            this.nb_heure_sum()
            this.nb_matiere_es()
            this.nb_heure_moyenne()
            this.nb_inter_distinct()
        },
        nb_heure_sum() {
            var query = {
                "aggs": {
                    "sum_nb_heure": {
                        "sum": {
                            "field": "nombre_heure"
                        }
                    }
                }
            }
            axios.get('http://localhost:9200/matiere_esgi/_search', {
                params: {
                    source: JSON.stringify(query),
                    source_content_type: 'application/json'
                }
            }).then((result) => {
                var dataHtpp = result.data
                this.sum_nb_heure = dataHtpp.aggregations.sum_nb_heure.value
            }).catch(function () {
                console.log('FAILURE!!')
            })
        },
        nb_matiere_es() {
            axios.get('http://localhost:9200/matiere_esgi/_count')
            .then((result) => {
                var dataHtpp = result.data
                this.nb_matiere = dataHtpp.count
            }).catch(function () {
                console.log('FAILURE!!')
            })
        },
        nb_heure_moyenne() {
            var query = {
                "aggs": {
                    "moyenne_nb_heure": {
                        "avg": {
                            "field": "nombre_heure"
                        }
                    }
                }
            }
            axios.get('http://localhost:9200/matiere_esgi/_search', {
                params: {
                    source: JSON.stringify(query),
                    source_content_type: 'application/json'
                }
            }).then((result) => {
                var dataHtpp = result.data
                this.moyene_heure = dataHtpp.aggregations.moyenne_nb_heure.value
            }).catch(function () {
                console.log('FAILURE!!')
            })
        },
        nb_inter_distinct() {
            var query = {
                "aggs": {
                    "nb_inter_dinstinct": {
                        "cardinality": {
                            "field": "intervenant.raw"
                        }
                    }
                }
            }
            axios.get('http://localhost:9200/matiere_esgi/_search', {
                params: {
                    source: JSON.stringify(query),
                    source_content_type: 'application/json'
                }
            }).then((result) => {
                var dataHtpp = result.data
                this.inter_distinct = dataHtpp.aggregations.nb_inter_dinstinct.value
            }).catch(function () {
                console.log('FAILURE!!')
            })
        }
    }
}
</script>

<style>

</style>