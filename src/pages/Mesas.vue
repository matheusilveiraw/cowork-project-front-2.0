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
        <div v-if="loading" class="text-center">
          <p>Carregando mesas...</p>
        </div>

        <div v-else-if="mesas.length === 0" class="text-center">
          <p>Nenhuma mesa cadastrada. Clique em "Nova Mesa" para começar.</p>
        </div>

        <div v-else class="row">
          <div class="col-md-6 mb-3" v-for="mesa in mesas" :key="mesa.id">
            <div class="mesa-box p-3">
              <h1>
                <strong>
                  Mesa {{ mesa.deskNumber }} - {{ mesa.deskName || 'Mesa sem nome' }}
                </strong>
              </h1>
              <p>Cliente: nome do cliente</p>
              <p>Último dia de aluguel: 01/01/2026</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <ModalCadastroMesas 
      :show="abrirModal" 
      @close="abrirModal = false"
      @mesa-salva="buscarMesas"
    /> 
  </div>
</template>

<script>
import ModalCadastroMesas from "@/components/ModalCadastroMesas.vue";
import axios from "axios";

export default {
  name: "Mesas",
  components: {
    ModalCadastroMesas
  },
  data() {
    return {
      abrirModal: false,
      mesas: [],
      loading: false
    };
  },
  methods: {
    async buscarMesas() {
      this.loading = true;
      try {
        const response = await axios.get(
          "http://localhost/projetos/cowork-project-back/public/desks"
        );
        
        console.log('Mesas carregadas:', response.data);
        
        this.mesas = response.data.data || response.data || [];
        
      } catch (error) {
        console.error('Erro ao buscar mesas:', error);
        alert('Erro ao carregar mesas: ' + (error.response?.data?.message || error.message));
      } finally {
        this.loading = false;
      }
    }
  },
  mounted() {
    this.buscarMesas();
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