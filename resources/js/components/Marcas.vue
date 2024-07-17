<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">


                <!-- início do card de busca -->
                <card-component titulo="Busca de marcas">
                    <template v-slot:conteudo>
                        <div class="form-row">
                            <div class="col mb-3">
                                <input-container-component titulo="ID" id="inputId" id-help="idHelp" texto-ajuda="Opcional. Informe o ID da marca">
                                    <input type="number" class="form-control" id="inputId" aria-describedby="idHelp" placeholder="ID" v-model="busca.id">
                                </input-container-component>
                            </div>
                            <div class="col mb-3">
                                <input-container-component titulo="Nome da marca" id="inputNome" id-help="nomeHelp" texto-ajuda="Opcional. Informe o nome da marca">
                                    <input type="text" class="form-control" id="inputNome" aria-describedby="nomeHelp" placeholder="Nome da marca" v-model="busca.nome">
                                </input-container-component>
                            </div>
                        </div>
                    </template>

                    <template v-slot:rodape>
                        <button type="submit" class="btn btn-primary btn-sm float-right" @click="pesquisar()">Pesquisar</button>
                    </template>
                </card-component>
                <!-- fim do card de busca -->


                <!-- início do card de listagem de marcas -->
                <card-component titulo="Relação de marcas">
                    <template v-slot:conteudo>
                        <table-component v-if="marcas.data"
                            :dados="marcas.data"
                            :visualizar="{visivel: true, dataToggle: 'modal', dataTarget:'#modalMarcaVisualizar'}"
                            :atualizar="{visivel: true, dataToggle: 'modal', dataTarget: '#modalMarcaAtualizar'}"
                            :remover="{visivel: true, dataToggle: 'modal', dataTarget: '#modalMarcaRemover'}"
                            :titulos="{
                                id: {titulo: 'ID', tipo:'text'},
                                nome:{titulo: 'Nome', tipo:'text'},
                                imagem:{titulo:'Imagem', tipo:'imagem'},
                                created_at:{titulo: 'Data de criação', tipo: 'data'}
                            }"
                        ></table-component>
                    </template>

                    <template v-slot:rodape>
                        <div class="row">
                            <div class="col-10 me-auto">
                                <paginate-component>
                                    <li v-for="l, key in marcas.links" :key="key" 
                                    :class="l.active ? 'page-item active' : 'page-item'" 
                                    @click="paginacao(l)">
                                        <a class="page-link" v-html="l.label"></a>
                                    </li>
                                </paginate-component>
                            </div>
                            <div class="col d-flex justify-content-end">
                                <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#modalMarca">Adicionar</button>
                            </div>
                        </div>
                    </template>
                </card-component>
                <!-- fim do card de listagem de marcas -->
            </div>
        </div>


        <!--Inicio do modal de inclusão de marca-->
        <modal-component id="modalMarca" titulo="Adicionar marca">

            <template v-slot:alerta>
                <alert-component tipo="success" :detalhes="transacaoDetalhes" titulo="Cadastro realizado com sucesso" v-if="transacaoStatus == 'adicionado'"></alert-component>
                <alert-component tipo="danger" :detalhes="transacaoDetalhes" titulo="Erro ao tentar cadastrar a marca" v-if="transacaoStatus == 'erro'"></alert-component>
            </template>

            <template v-slot:conteudo>
                <div class="form-group">
                    <input-container-component titulo="Nome da marca" id="novoNome" id-help="novoNomeHelp" texto-ajuda="Informe o nome da marca">
                        <input type="text" class="form-control" id="novoNome" aria-describedby="novoNomeHelp" placeholder="Nome da marca" v-model="nomeMarca">
                    </input-container-component>
                </div>

                <div class="form-group">
                    <input-container-component titulo="Imagem" id="novoImagem" id-help="novoImagemHelp" texto-ajuda="Selecione uma imagem no formato PNG">
                        <input type="file" class="form-control" id="novoImagem" aria-describedby="novoImagemHelp" placeholder="Selecione uma imagem" @change="carregarImagem($event)">
                    </input-container-component>
                </div>
            </template>
            <template v-slot:rodape>
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Fechar</button>
                <button type="button" class="btn btn-primary" @click="salvar()">Salvar</button>
            </template>
        </modal-component>
        <!--Fim do modal de inclusão de marca-->

        <!--inicio do modal de visualização de marca-->
        <modal-component id="modalMarcaVisualizar" titulo="Visualizar marca">
            <template v-slot:alerta>
            </template>
            <template v-slot:conteudo>
                <input-container-component titulo="ID">
                    <input type="text" class="form-control" :value="$store.state.item.id" disabled>
                </input-container-component>
                <input-container-component titulo="Nome da Marca">
                    <input type="text" class="form-control" :value="$store.state.item.nome" disabled>
                </input-container-component>
                <input-container-component titulo="Imagem da Marca">
                <img :src="'/storage/'+$store.state.item.imagem" width="50%" height="50%" v-if="$store.state.item.imagem">
                </input-container-component>
                <input-container-component titulo="Data de Criação">
                    <input type="text" class="form-control" :value="$store.state.item.created_at" disabled>
                </input-container-component>
            </template>
            <template v-slot:rodape>
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Fechar</button>
            </template>
        </modal-component>
        <!--Fim do modal de vizualização de marca-->

        <!--inicio do modal de remoção de marca-->
        <modal-component id="modalMarcaRemover" titulo="Remover marca">
            <template v-slot:alerta>
                <alert-component tipo="success" titulo="Transação realizada com sucesso" :detalhes="$store.state.transacao.status" v-if="$store.state.transacao.status == 'sucesso'">
                </alert-component>
                <alert-component tipo="danger" titulo="Erro na transação" :detalhes="$store.state.transacao.status" v-if="$store.state.transacao.status == 'erro'">
                </alert-component>
            </template>
            <template v-slot:conteudo v-if="$store.state.transacao.status != 'sucesso'">
                <input-container-component titulo="ID">
                    <input type="text" class="form-control" :value="$store.state.item.id" disabled>
                </input-container-component>
                <input-container-component titulo="Nome da Marca">
                    <input type="text" class="form-control" :value="$store.state.item.nome" disabled>
            </input-container-component>
            </template>
            <template v-slot:rodape>
                <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Fechar</button>
                <button type="button" class="btn btn-danger" @click="remover()" v-if="$store.state.transacao.status != 'sucesso'" >Remover</button>
            </template>
        </modal-component>
        <!--Fim do modal de remoção de marca-->

        <!--Inicio do modal de atualização de marca-->
        <modal-component id="modalMarcaAtualizar" titulo="Atualizar marca">

            <template v-slot:alerta>
                <alert-component tipo="success" titulo="Transação realizada com sucesso" :detalhes="$store.state.transacao.status" v-if="$store.state.transacao.status == 'sucesso'">
                </alert-component>
                <alert-component tipo="danger" titulo="Erro na transação" :detalhes="$store.state.transacao.status" v-if="$store.state.transacao.status == 'erro'">
                </alert-component>
            </template>

        <template v-slot:conteudo>
            <div class="form-group">
                <input-container-component titulo="Nome da marca" id="atualizaNome" id-help="atualizaNomeHelp" texto-ajuda="Nome da marca">
                    <input type="text" class="form-control" id="atualizaNome" aria-describedby="atualizaNomeHelp" v-model="$store.state.item.nome">
                </input-container-component>
            </div>

            <div class="form-group">
                <input-container-component titulo="Imagem" id="atualizaImagem" id-help="atualizaImagemHelp" texto-ajuda="Selecione uma imagem no formato PNG">
                    <input type="file" class="form-control" id="atualizaImagem" aria-describedby="atualizaImagemHelp" placeholder="Selecione uma imagem" @change="carregarImagem($event)">
                </input-container-component>
            </div>
        </template>
        <template v-slot:rodape>
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Fechar</button>
            <button type="button" class="btn btn-primary" @click="atualizar()">Atualizar</button>
        </template>
        </modal-component>
        <!--Fim do modal de inclusão de marca-->



    </div>
</template>

<script>
import Paginate from './Paginate.vue'
    export default {
        components:{ Paginate},
        data() {
            return {
                urlBase: 'http://localhost:8000/api/v1/marca',
                urlPaginacao: '',
                urlFiltro:'',
                nomeMarca: '',
                arquivoImagem: [],
                transacaoStatus: '',
                transacaoDetalhes: {},
                marcas: { data: []},
                busca: {id:'', nome:''}
            }
        },
        methods: {
            atualizar() {

            let formData = new FormData();
            formData.append('_method', 'patch')
            formData.append('nome', this.$store.state.item.nome)

            if(this.arquivoImagem[0]){
            formData.append('imagem', this.arquivoImagem[0])
            }

            let url = this.urlBase + '/' + this.$store.state.item.id

            let config = {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            }

            axios.post(url, formData, config)
                .then(response => {
                    console.log('Atualizado', response)
                    atualizaImagem.value = ''
                    this.$store.state.transacao.status = 'sucesso'
                    this.$store.state.transacao.mensagem = 'Registro de marca atualizado com sucesso'
                    this.carregarLista()
                })
                .catch(errors => {
                    this.$store.state.transacao.data = ''
                    console.log('Erro de atualização', errors.response)
                    this.$store.state.transacao.status = 'erro'
                    this.$store.state.transacao.mensagem = errors.response.data.errors
                })
            },
            remover(){
                let confirmacao = confirm('Tem certeza que deseja remover esse registro?')
                
                if(!confirmacao){
                    return false;
                } 

                let url = this.urlBase + "/" + this.$store.state.item.id;

                let formData = new FormData();
                formData.append('_method', 'delete')

                
                axios.post(url, formData)
                .then(response =>{
                    console.log('Registro removido com sucesso')
                    this.$store.state.transacao.status = 'sucesso'
                    this.$store.state.transacao.mensagem = response.data.msg
                    this.carregarLista();
                })
                .catch(errors => {
                    this.$store.state.transacao.status = 'erro'
                    this.$store.state.transacao.mensagem = errors.response.data.erro
                    console.log('Houve um erro na tentativa de remoção do registro')
                })
            },

            pesquisar(){
                 let filtro = ''
                 
                 for(let chave in this.busca){

                {
                        filtro += ";"
                    }
                    filtro += chave + ':like:' + this.busca[chave]
                    }
                if(filtro != ''){
                this.urlPaginacao = 'page=1'
                this.urlFiltro= '&filtro='+filtro
                } else{
                    this.urlFiltro = ''
                }
                this.carregarLista()
            },
            paginacao(l){
                if(l.url){
                this.urlPaginacao = l.url.split('?')[1]
                this.carregarLista()
                }
            },
            carregarLista() {

                let url = this.urlBase + '?' + this.urlPaginacao + this.urlFiltro

                axios.get(url)
                    .then(response => {
                        this.marcas = response.data
                        console.log(this.marcas)
                    })
                    .catch(errors => {
                        console.log(errors)
                    })
            },
            carregarImagem(e) {
                this.arquivoImagem = e.target.files
            },
            salvar() {
                console.log(this.nomeMarca, this.arquivoImagem[0])

                let formData = new FormData();
                formData.append('nome', this.nomeMarca)
                formData.append('imagem', this.arquivoImagem[0])

                let config = {
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    }
                }

                axios.post(this.urlBase, formData, config)
                    .then(response => {
                        this.transacaoStatus = 'adicionado'
                        this.transacaoDetalhes = {
                            mensagem: 'ID do registro: ' + response.data.id
                        }

                        console.log(response)
                    })
                    .catch(errors => {
                        this.transacaoStatus = 'erro'
                        this.transacaoDetalhes = {
                            mensagem: errors.response.data.message,
                            dados: errors.response.data.errors
                        }
                        //errors.response.data.message
                    })
            }
        },
        mounted() {
            this.carregarLista()
        }
    }
</script>