<template>
    <div>
        <table class="table table-hover">
            <thead>
                <tr>
                    <th scope="col" v-for="t, key in titulos" :key="key">{{t.titulo}}</th>
                    <th v-if="visualizar.visivel || atualizar.visivel || remover.visivel"></th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="obj, chave in dadosFiltrados" :key="chave">
                    <td v-for="valor, chaveValor in obj" :key="chaveValor">
                        <span v-if="titulos[chaveValor].tipo == 'text'">{{valor}}</span>
                        <span v-if="titulos[chaveValor].tipo == 'data'">
                            {{ formataDataTempo(valor) }}
                        </span>
                        <span v-if="titulos[chaveValor].tipo == 'imagem'">
                            <img :src="'/storage/'+valor" width="30" height="30">
                        </span>
                    </td>
                    <td v-if="visualizar.visivel || atualizar.visivel || remover.visivel">
                        <Button v-if="visualizar.visivel" class="btn btn-outline-primary btn-sm" :data-bs-toggle="visualizar.dataToggle" :data-bs-target="visualizar.dataTarget" @click="setStore(obj)" >Visualizar</Button>
                        <Button v-if="atualizar.visivel" class="btn btn-outline-primary btn-sm" :data-bs-toggle="atualizar.dataToggle" :data-bs-target="atualizar.dataTarget" @click="setStore(obj)">Atualizar</Button>
                        <Button v-if="remover.visivel" class="btn btn-outline-danger btn-sm" :data-bs-toggle="remover.dataToggle" :data-bs-target="remover.dataTarget" @click="setStore(obj)">Remover</Button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
    export default {
        props: ['dados', 'titulos', 'atualizar', 'visualizar', 'remover'],
        methods:{
            formataDataTempo(data){
                if(!data) return ''

                data = data.split('T')

                let dataFormatada = data[0]
                dataFormatada = dataFormatada.split('-');
                dataFormatada = dataFormatada[2] + '/' + dataFormatada[1] + '/' + dataFormatada[0];
                
                let horaFormatada = data[1]
                horaFormatada = horaFormatada.split('.')
                horaFormatada = horaFormatada[0]

                let dataFinal = dataFormatada + ' ' + horaFormatada;

                console.log(horaFormatada)
                return dataFinal;

            },
            setStore(obj){
                this.$store.state.transacao.status = ''
                this.$store.state.transacao.mensagem = ''
                this.$store.state.transacao.dados = ''
                this.$store.state.item = obj
            }
        },
        computed: {
            dadosFiltrados() {
                
                let campos = Object.keys(this.titulos)
                let dadosFiltrados = []

                this.dados.map((item, chave) => {

                    let itemFiltrado = {}
                    campos.forEach(campo => {
                        
                        itemFiltrado[campo] = item[campo] //utilizar a sintaxe de array para atribuir valores a objetos
                    })
                    dadosFiltrados.push(itemFiltrado)
                })

                return dadosFiltrados //retorne um array de objetos 
            }
        }
    }
</script>