<template>
    <v-container>
    <v-row class="text-center">

      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3">
          Moteur de recherche sur les matières à l'ESGI
        </h1>
      </v-col>

      <v-col class="mb-5" cols="12">
        <v-row>
            <v-col class="mb-4">
                <v-text-field
                    v-model="recherche"
                    label="Bar de recherche"
                ></v-text-field>
            </v-col>
            <v-col class="mb-4">
                <v-select
                    :items="categories"
                    v-model="categorie"
                    label="Catégories"
                    outlined
                ></v-select>
            </v-col>
            <v-col class="mb-4">
                <v-text-field
                    v-model="nbMin"
                    label="Nombre d'heure minimum"
                    type="number"
                ></v-text-field>
            </v-col>
            <v-col class="mb-4">
                <v-text-field
                    v-model="nbMax"
                    label="Nombre d'heure maximum"
                    type="number"
                ></v-text-field>
            </v-col>            
        </v-row>
        <v-row justify="center">
            <v-simple-table dark>
                <template v-slot:default>
                <thead>
                    <tr>
                    <th class="text-center">
                        Matiere
                    </th>
                    <th class="text-center">
                        Categories
                    </th>
                    <th class="text-center">
                        Coefficient
                    </th>
                    <th class="text-center">
                        ECTS
                    </th>
                    <th class="text-center">
                        Intervenant
                    </th>
                    <th class="text-center">
                        Nombre d'heure
                    </th>
                    <th class="text-center">
                        Date de debut
                    </th>
                    <th class="text-center">
                        Date de fin
                    </th>
                    <th class="text-center">
                        Description
                    </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in matieres" :key="item._id">
                        <td>{{ item._source.matiere }}</td>
                        <td>{{ item._source.categories.toString() }}</td>
                        <td>{{ item._source.coefficient }}</td>
                        <td>{{ item._source.ects }}</td>
                        <td>{{ item._source.intervenant }}</td>
                        <td>{{ item._source.nombre_heure }}</td>
                        <td>{{ item._source.date_first_cours }}</td>
                        <td>{{ item._source.date_last_cours }}</td>
                        <td>{{ item._source.description }}</td>
                    </tr>
                </tbody>
                </template>
            </v-simple-table>
        </v-row>
      </v-col>

      <v-col class="mb-5" cols="12">
        <v-row justify="center">
        </v-row>
      </v-col>

    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios'

export default {
    name: 'Elasticsearch',
     data () {
        return {
            matieres: [],
            categories: ['', 'DEV', 'LANGUE', 'IA', 'BIG DATA', 'RESEAU'],
            categorie: '',
            recherche: '',
            nbMin: null,
            nbMax: null
        }
    },
    created: function () {
        this.start()
    },
    methods: {
        start () {
            this.recherche = ''
            this.categorie = ''
            this.nbMin = null
            this.nbMax = null
            axios.get('http://localhost:9200/matiere_esgi/_search')
            .then((result) => {
                var dataHtpp = result.data
                this.matieres = dataHtpp.hits.hits
                console.log(this.matieres)
            })
            .catch(function () {
                console.log('FAILURE!!')
            })
        },
        barRecherche (value) {
            this.categorie = ''
            this.nbMin = null
            this.nbMax = null
            if (value == null) {
                this.start()
            } else {
                axios.get('http://localhost:9200/matiere_esgi/_search?q=*' + value + '*')
                .then((result) => {
                    var dataHtpp = result.data
                    this.matieres = dataHtpp.hits.hits
                    console.log(this.matieres)
                })
                .catch(function () {
                    console.log('FAILURE!!')
                })
            }
        },
        rechercheCategories (value) {
            this.recherche = ''
            this.nbMin = null
            this.nbMax = null
            if (value == '') {
                this.start()
            } else {
                var query =
                {
                    "query": {
                        "match": {
                            "categories": value
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
                    this.matieres = dataHtpp.hits.hits
                    console.log(this.matieres)
                }).catch(function () {
                    console.log('FAILURE!!')
                })
            }
        },
        rechercheMinMax (min, max) {
            this.categorie = ''
            this.recherche = ''
            var subQuery = { }
            if ((min == null || min == '') && (max == null || max == '')) {
                console.log("ici 1")
                this.start()
                return
            } else if ((min == null || min == '') && max != null) {
                console.log("ici 2")
                subQuery = {
                    "lte": max
                }
            } else if (min != null && (max == null || max == '')) {
                console.log("ici 3")
                subQuery = {
                    "gte": min
                }
            } else {
                subQuery = {
                    "gte": min,
                    "lte": max
                }
            }
            var query = {
                "query": {
                    "range": {
                        "nombre_heure": subQuery
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
                this.matieres = dataHtpp.hits.hits
                console.log(this.matieres)
            }).catch(function () {
                console.log('FAILURE!!')
            })
        }
    },
    watch: {
        recherche: function (newRep, oldRep) {
            console.log(newRep, oldRep)
            this.barRecherche(newRep)
        },
        categorie: function (newRep, oldRep) {
            console.log(newRep, oldRep)
            this.rechercheCategories(newRep)
        },
        nbMin: function (newRep, oldRep) {
            console.log(newRep, oldRep)
            console.log(this.nbMin, this.nbMax)
            this.rechercheMinMax(newRep, this.nbMax)
        },
        nbMax: function (newRep, oldRep) {
            console.log(newRep, oldRep)
            console.log(this.nbMin, this.nbMax)
            this.rechercheMinMax(this.nbMin, newRep)
        }
    }
    
}
</script>

<style>

</style>