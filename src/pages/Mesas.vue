<template>
  <div class="content">
    <div class="card">
      <div class="card-header">
        <h4 class="title">Mesas</h4>
      </div>

      <div class="menu-rentals">
        <div class="d-flex mb-3">
          <button class="btn btn-primary mr-2" @click="abrirModal = true">
            Nova Mesa
          </button>

          <button class="btn btn-secondary">
            Alterar Visualização mesa
          </button>
        </div>
      </div>

      <div class="card-body">
        <div class="row">
          <div class="col-md-6 mb-3" v-for="n in 6" :key="n">
            <div class="mesa-box p-3">
              <h1><strong>Mesa {{ n }} - nome da mesa</strong></h1>
              <p>Cliente: nome do cliente</p>
              <p>Último dia de aluguel: 01/01/2026</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <ModalCadastroMesas :show="abrirModal" @close="abrirModal = false"/> 
    <!-- Isso é como se fosse o modal no código, fica escondido até ser chamadi -->
  </div>
</template>

<script>
import ModalCadastroMesas from "@/components/ModalCadastroMesas.vue"; //importando o modal para ficar disponivel

export default {
  name: "Mesas",
  components: {
    ModalCadastroMesas
  },
  data() {
    return {
      abrirModal: false
    };
  }
};
</script>


<style scoped>
.mesa-box {
  border: 1px solid #ccc;
  border-radius: 6px;
  min-height: 150px;
  position: relative;
}
.mesa-box strong {
  position: absolute;
  top: 8px;
  left: 12px;
  font-weight: 600;
}
.mesa-locked {
  border-color: red !important;
}
</style>
