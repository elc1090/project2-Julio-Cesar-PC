
<script>
import CarCard from '../components/carCard.vue'

export default {
    name: 'fipeForm',
    components: {
        CarCard,
    }
}
</script>

<template>
    <div class="">
        <div class="container d-flex justify-content-center mt-5 gap-5">
            
            <div class="w-50">
                <CarCard class="mx-auto w-100" tituloCard="Veiculo 1" tipo="tipo-1" marca="marca-1" modelo="modelo-1" ano="ano-1" nome="veiculo1" />
            </div>
            
            <div class="w-50">
                <CarCard class="mx-auto w-100" tituloCard="Veiculo 2" tipo="tipo-2" marca="marca-2" modelo="modelo-2" ano="ano-2" nome="veiculo2" />
            </div>

        </div>
    </div>
</template>


