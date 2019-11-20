<template>
    <v-data-table :headers="headers" :items="desserts" :search="search" sort-by="calories" class="elevation-1">
        <template v-slot:top>
            <v-toolbar flat color="white">
                <v-toolbar-title>Crud Atma Pedia</v-toolbar-title>
                <v-divider class="mx-4" inset vertical></v-divider>
                 <v-text-field
                    v-model="search"
                    append-icon="mdi-search-web"
                    label="Search"
                    single-line
                    hide-details
                ></v-text-field>
                <v-spacer></v-spacer>

                <v-btn color="primary" dark class="mb-2" @click="deleteAll()">Delete All</v-btn>
                <v-divider class="mx-4" inset vertical></v-divider>

                <v-dialog v-model="dialog" max-width="500px">
                    <template v-slot:activator="{ on }">
                        <v-btn color="primary" dark class="mb-2" v-on="on">buat makanan</v-btn>
                    </template>
                    <v-card>
                        <v-card-title>
                            <span class="headline">{{ formTitle }}</span>
                        </v-card-title>
                    
                        <v-card-text>
                            <v-container>
                                <v-row>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.name" label="Dessert name"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.calories" label="Calories"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.fat" label="Fat (g)"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.carbs" label="Carbs (g)"></v-text-field>
                                    </v-col>
                                    <v-col cols="12" sm="6" md="4">
                                        <v-text-field v-model="editedItem.protein" label="Protein (g)"></v-text-field>
                                    </v-col>
                                </v-row>
                            </v-container>
                        </v-card-text>

                        <v-card-actions>

                            <v-spacer></v-spacer>

                            <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                            <v-btn color="blue darken-1" text @click="save">Save</v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </v-toolbar>
        </template>

       <template v-slot:item.multiDelete ="{ item }">
         <v-checkbox label="" color="indigo" :id="item['.key']" :value="item['.key']" v-model="selected"></v-checkbox>
      </template>

        <template v-slot:item.action="{ item }">
            <v-icon class="mr-1" @click="editItem(item)">mdi-silverware</v-icon>
            <v-icon class="mr-1" @click="deleteItem(item)">mdi-delete</v-icon>
        </template>

    </v-data-table>
</template>

<script>

import{ dessertsRef } from '../firebase';

    export default {
        data: () => ({
            cek : -1,
            search:'',
            dialog: false,

            selected: [],
            marked: -1,

            headers: [
                {
                    text: 'Dessert (100g serving)',
                    align: 'left',
                    sortable: false,
                    value: 'name',
                },
                {
                    text: 'Calories',
                    value: 'calories'
                },
                {
                    text: 'Fat (g)',
                    value: 'fat'
                },
                {
                    text: 'Carbs (g)',
                    value: 'carbs'
                },
                {
                    text: 'Protein (g)',
                    value: 'protein'
                },
                { 
                    text: "#", 
                    value:"multiDelete", 
                    sortable: false 
                },
                {
                    text: 'Actions',
                    value: 'action',
                    sortable: false
                },
            ],
            desserts: [],
            editedIndex: -1,
            editedItem: {
                name: '',
                calories: 0,
                fat: 0,
                carbs: 0,
                protein: 0,
            },
            defaultItem: {
                name: '',
                calories: 0,
                fat: 0,
                carbs: 0,
                protein: 0,
            },
        }),
        
        firebase: {
            desserts : dessertsRef
        },
        computed: {
            formTitle() {
                return this.cek === -1 ? 'New Item' : 'Edit Item'
            },
        },

        watch: {
            dialog(val) {
                val || this.close()
            },
        },

        methods: {
            close() {
                this.dialog = false
            },
            // methods untuk save
            save() {
                if (this.cek > -1) {
                    dessertsRef.child(this.editedIndex).set({
                        name : this.editedItem.name,
                        calories : this.editedItem.calories, 
                        fat : this.editedItem.fat,
                        carbs: this.editedItem.carbs,
                        protein: this.editedItem.protein})
                        this.clear()
                } else {
                    dessertsRef.push({
                        name: this.editedItem.name,
                        calories: this.editedItem.calories,
                        fat: this.editedItem.fat,
                        carbs: this.editedItem.carbs,
                        protein: this.editedItem.protein})
                        this.clear()
                }
                this.close()
            },
            editItem (item) {
                this.editedIndex = item['.key']
                this.cek = 0
                this.editedItem = Object.assign({}, item)
                this.dialog = true
            },
            deleteItem(item) {
                confirm('Are you sure you want to delete this item?') && dessertsRef.child((item['.key'])).remove()
            },
            clear(){
                this.editedItem={}
                this.editedItem.name= ''
                this.editedItem.calories= 0
                this.editedItem.fat= 0
                this.editedItem.carbs= 0
                this.editedItem.protein= 0
                this.cek=-1
            },
            deleteAll(){
                console.log(this.selected)
                for(this.marked=0;this.marked<this.selected.length;this.marked++){
                console.log(this.marked)
                dessertsRef.child(this.selected[this.marked]).remove()
                }
                this.selected=[]
                console.log(this.selected)
            }
        },
    }
</script>