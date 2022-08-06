<template>
    <h1 class="m-3 d-flex justify-content-center text-center">
        Abertura de Chamados
    </h1>
    <div class="card d-flex p-2 m-5 bd-highlight">
        <form @submit.prevent="abrirChamado">
            <div class="row">
                <div class="col">
                    <label for="contrato" class="form-label">Contrato:</label>
                    <v-select :options="contratos" @option:selected="splitEvent" v-model="contrato" class="form-control" id="contrato"></v-select>
                </div>
                <div class="col">
                    <label for="cliente" class="form-label">Cliente:</label>
                    <input type="text" v-model="cliente" @change="splitTitulo" class="form-control" id="cliente" >
                </div>
                <div class="col">
                    <label for="date" class="form-label">Data e Hora:</label>
                    <Datepicker v-model="date" />
                </div>
                
            </div>
            <div class="row mb-3 mt-3">
                <div class="col">
                <label for="titulo" class="form-label">Titulo:</label>
                <input type="text" class="form-control" v-model="titulo" id="titulo" aria-describedby="tituloHelp">
                <div id="tituloHelp" class="form-text">A tag de abertura é gerada automaticamente no prenchimento do
                    cliente.</div>
                </div>
                <div class="col">
                    <label for="categoria" class="form-label">Categoria:</label>
                    <v-select :options="categorias" v-model="categoria" class="form-control" id="categoria"></v-select>
                </div>
            </div>
            <div class="mb-3">
                <label for="descricao" class="form-label">Descrição:</label>
                <textarea class="form-control" v-model="descricao" id="descricao" rows="5" cols="33"></textarea>
            </div>
            <button type="submit" class="btn btn-primary">Abrir chamado</button> - App Token: {{ app_token }} User Token: {{ user_token }} Session: {{ session }}
        </form>
    </div>
</template>

<script>
import vSelect from 'vue-select'
import 'vue-select/dist/vue-select.css';

import Datepicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'

import axios from 'axios'

export default {
    name: 'TicketMaker',
    data() {
        return {
            contratos: [],
            contrato: '',
            cliente: '',
            titulo: '',
            descricao: '',
            categorias: [],
            categoria: '',
            app_token: '',
            user_token: '',
            session: '',
            date: new Date().toJSON(),
            tag: '',
        }
    },
    methods: {
        splitEvent(){
            var lue = this.contrato.label.split(" ")
            this.tag = lue[0];
            this.titulo = '[' + lue[0] + '] - ' 
        },
        splitTitulo(){
            this.titulo = this.titulo + this.cliente + ' - '
        },
        updateClients() {
            //puxa clientes da api
        },
        initSession() {
            const config = {
            headers:{
                'Content-Type': 'application/json',
                'Authorization': 'user_token ' + this.user_token,
                'App-Token': this.app_token,
            }
            };
            const url = 'https://glpi.setupti.com/glpi/apirest.php/initSession';


            const apicall = axios.create({
                baseURL: url,
            });

            return apicall.get(url, config)
            .then((res) => {
                console.log(res);
                this.session = res.data.session_token
            }).catch((err) => {
                console.log(err);
            });
            
        },
        updateCategories(){

            if(this.session != ''){
                const config = {
                headers:{
                    'Content-Type': 'application/json',
                    'Session-Token': this.session,
                    'App-Token': this.app_token,
                }
                };
                const url = 'https://glpi.setupti.com/glpi/apirest.php/itilcategory';


                const apicall = axios.create({
                    baseURL: url,
                });

                apicall.get(url, config)
                .then((res) => {
                    //console.log(res);
                    for(var i = 0; i < res.data.length ; i++){
                        var id = res.data[i].id
                        var label = res.data[i].name
                        var cat = {}
                        cat.id = id
                        cat.label = label

                        this.categorias.push(cat) 
                    }
                }).catch((err) => {
                    console.log(err);
                })
            }
            
        },
        updateLocations(){

            if(this.session != ''){
                
                const config = {
                headers:{
                    'Content-Type': 'application/json',
                    'Session-Token': this.session,
                    'App-Token': this.app_token,
                }
                };
                const url = 'https://glpi.setupti.com/glpi/apirest.php/location';


                const apicall = axios.create({
                    baseURL: url,
                });

                apicall.get(url, config)
                .then((res) => {
                    //console.log(res);
                    for(var i = 0; i < res.data.length ; i++){
                        var id = res.data[i].id
                        var label = res.data[i].name
                        var cat = {}
                        cat.id = id
                        cat.label = label

                        this.contratos.push(cat) 
                    }
                }).catch((err) => {
                    console.log(err);
                })
            }
        },
        addChamado(){
            const config = {
            headers:{
                'Content-Type': 'application/json',
                'Session-Token': this.session,
                'App-Token': this.app_token,
            }
            };
            const url = 'https://glpi.setupti.com/glpi/apirest.php/Ticket/';
            
            const data = {
                "input": {
                    "type":1,
                    "entities_id":1,
                    "itilcategories_id": this.categoria.id,
                    //"requesttypes_id":6,
                    //"urgency":5,
                    //"impact":5,
                    //"priority":5,
                    //"date": this.date,    need fix data format
                    "name": this.titulo,
                    "content": this.descricao,
                    "locations_id": this.contrato.id,
                    "_users_id_assign": "9",
                    "_users_id_requester": "9",

                }
            };

            const apicall = axios.create({
                baseURL: url,
            });

            apicall.post(url, data, config)
            .then((res) => {
                console.log(res);
                this.session = res.data.session_token
            }).catch((err) => {
                console.log(err);
            })
        },
        async abrirChamado() {
            
            await this.initSession() 
            this.addChamado()

            this.contratos= []
            this.contrato= ''
            this.cliente= ''
            this.titulo= ''
            this.descricao= '',
            this.categorias= []
            this.categoria= ''
            this.app_token= ''
            this.user_token= ''
            this.session= ''
            this.date= new Date().toJSON()
            this.tag= ''
  
            this.updateCategories()
            this.updateLocations()
        },
    },
    components: {
        vSelect,
        Datepicker,
    },
    async beforeMount(){
        
        await this.initSession()    
        this.updateCategories()
        this.updateLocations()
        document.title = 'GLPI - Abertura de chamados';
    },
}
</script>

<style>
</style>
