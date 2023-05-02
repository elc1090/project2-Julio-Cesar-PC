<script>
    import axios from 'axios'
    import Veiculo from '../../models/veiculo.js'

    let tipoVeiculo
    let marcaVeiculo
    let modeloVeiculo
    let anoVeiculo

    export default {
        name: 'carCard',
        props: {
            tipo: String,
            marca: String,
            modelo: String,
            ano: String,
            tituloCard: String,
            nome: String
        },

        data() {
            return {
                loading: false,
                loaded: false,
                veiculoFinal: new Veiculo(),
            }
        },

        mounted() {
            document.getElementById("tipo").id = this.tipo;
            document.getElementById("marca").id = this.marca;
            document.getElementById("modelo").id = this.modelo;
            document.getElementById("ano").id = this.ano;
            document.getElementById("card").id = this.nome;
            document.getElementById("card-header").id = this.nome + "-header";
        },

        methods: {
            getMarcas() {
                tipoVeiculo = document.getElementById(this.tipo).value

                if (tipoVeiculo != "tipo") {
                    axios
                    .get(`https://parallelum.com.br/fipe/api/v2/${tipoVeiculo}/brands`)
                    .then(response => {
                        document.getElementById(this.marca).innerHTML = ""
                        document.getElementById(this.modelo).innerHTML = ""
                        document.getElementById(this.ano).innerHTML = ""
                        document.getElementById(this.marca).appendChild(new Option("Selecione o marca", "marca"))
                        document.getElementById(this.modelo).appendChild(new Option("Selecione o modelo", "modelo"))
                        document.getElementById(this.ano).appendChild(new Option("Selecione o ano", "ano"))

                        response.data.forEach(element => {
                            let option = document.createElement("option")
                            option.text = element.name
                            option.value = element.code
                            document.getElementById(this.marca).appendChild(option)
                        });
                    })
                }
            },

            getModelos() {
                marcaVeiculo = document.getElementById(this.marca).value

                if (marcaVeiculo != "marca") {
                    axios
                    .get(`https://parallelum.com.br/fipe/api/v2/${tipoVeiculo}/brands/${marcaVeiculo}/models`)
                    .then(response => {
                        document.getElementById(this.modelo).innerHTML = ""
                        document.getElementById(this.ano).innerHTML = ""
                        document.getElementById(this.modelo).appendChild(new Option("Selecione o modelo", "modelo"))
                        document.getElementById(this.ano).appendChild(new Option("Selecione o ano", "ano"))
                        response.data.forEach(element => {
                            let option = document.createElement("option")
                            option.text = element.name
                            option.value = element.code
                            document.getElementById(this.modelo).appendChild(option)
                        });
                    })
                }
            },

            getAnos() {
                modeloVeiculo = document.getElementById(this.modelo).value

                if (modeloVeiculo != "modelo") {
                    axios
                    .get(`https://parallelum.com.br/fipe/api/v2/${tipoVeiculo}/brands/${marcaVeiculo}/models/${modeloVeiculo}/years`)
                    .then(response => {
                        document.getElementById(this.ano).innerHTML = ""
                        document.getElementById(this.ano).appendChild(new Option("Selecione o ano", "ano"))
                        response.data.forEach(element => {
                            let option = document.createElement("option")
                            option.text = element.name
                            option.value = element.code
                            document.getElementById(this.ano).appendChild(option)
                        });
                    })
                }
            },

            getVeiculo() {
                anoVeiculo = document.getElementById(this.ano).value
                this.loading = true

                if (anoVeiculo != "ano") {
                    axios
                    .get(`https://parallelum.com.br/fipe/api/v2/${tipoVeiculo}/brands/${marcaVeiculo}/models/${modeloVeiculo}/years/${anoVeiculo}`)
                    .then(response => {
                        let veiculoFipe = response.data
                        axios
                        .get(`https://parallelum.com.br/fipe/api/v2/${tipoVeiculo}/${veiculoFipe.codeFipe}/years/${anoVeiculo}/history`)
                        .then(response => {
                            this.veiculoFinal = new Veiculo(tipoVeiculo, marcaVeiculo, modeloVeiculo, anoVeiculo, veiculoFipe.price, response.data)
                            this.loading = false
                            this.loaded = true
                            let preco = this.veiculoFinal.precoVeiculo
                            let precoMedio = this.calulaMediaPreco(this.parseFloatPreco(this.veiculoFinal.veiculo.priceHistory))
                            let precoMaximo = this.calculaPrecoMaximo(this.veiculoFinal.veiculo.priceHistory)
                            let precoMinimo = this.calculaPrecoMinimo(this.veiculoFinal.veiculo.priceHistory)
                            document.getElementById(this.nome + "-header").innerHTML = this.veiculoFinal.veiculo.model + " - " + anoVeiculo

                            // preenche preço no card
                            document.getElementById(this.nome).innerHTML = `
                            
                        <p class="card-text">Preço atual de mercado: <span class="fw-bold">${preco}</span></p>
                        <p class="card-text">Preço médio de mercado: <span class="fw-bold">R$ ${precoMedio.toLocaleString('pt-br', {minimumFractionDigits: 2})}</span></p>
                        <p class="card-text">Preço máximo de mercado: <span class="fw-bold">R$ ${precoMaximo.toLocaleString('pt-br', {minimumFractionDigits: 2})}</span></p>
                        <p class="card-text">Preço mínimo de mercado: <span class="fw-bold">R$ ${precoMinimo.toLocaleString('pt-br', {minimumFractionDigits: 2})}</span></p>
                            `
                        })
                    })   
                }
            },

            calulaMediaPreco(array) {
                let soma = 0
                array.forEach(element => {
                    soma += element.price
                });
                return soma / array.length
            },

            calculaPrecoMaximo(array) {
                let max = 0
                array.forEach(element => {
                    if (element.price > max) {
                        max = element.price
                    }
                });
                return max
            },

            calculaPrecoMinimo(array) {
                let min = array[0].price
                array.forEach(element => {
                    if (element.price < min) {
                        min = element.price
                    }
                });
                return min
            },

            parseFloatPreco(array) {
                array.forEach(element => {
                    element.price = parseFloat(element.price.replace("R$ ", "").replace(".", "").replace(",", ".").replace(" ", ""))
                });
                return array
            },
        }   
            
    }
</script>

<template>
    <div class="card w-100">
        <div class="card-body">
            <form class="d-flex flex-column align-items-center">
                <h3>{{ tituloCard }}</h3>
                <select class="form-select" name="tipo" id="tipo" v-on:change="getMarcas()">
                    <option value="tipo">Selecione o tipo</option>
                    <option value="cars">Carros</option>
                    <option value="motorcycles">Motos</option>
                    <option value="trucks">Caminhões</option>
                </select>

                <select class="form-select mt-2" name="marca" id="marca" v-on:change="getModelos()">
                    <option value="marca">Selecione a marca</option>
                </select>

                <select class="form-select mt-2" name="modelo" id="modelo" v-on:change="getAnos()">
                    <option value="modelo">Selecione o modelo</option>
                </select>

                <select class="form-select mt-2" name="ano" id="ano" >
                    <option value="ano">Selecione o ano</option>
                </select>

                <div class="d-grid gap-2 mt-2">
                    <button class="btn btn-primary" type="button" v-on:click="getVeiculo()">Enviar</button>
                </div>

                <div class="spinner-border mt-2" role="status" v-show="loading">
                    <span class="visually-hidden hidden">Loading...</span>
                </div>

                <hr class="hr" />

                <div class="card mt-2" v-show="loaded">
                    <h3 class="card-header" id="card-header"></h3>
                    <div class="card-body" id="card">

                    </div>
                </div>


            </form>
        </div>
    </div>
</template>