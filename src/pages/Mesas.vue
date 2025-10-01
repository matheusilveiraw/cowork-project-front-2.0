<template>
  <div class="content">
    <div class="card">
      <div class="card-header">
        <h4 class="title">Mesas</h4>
      </div>

      <div class="menu-rentals">
        <div class="d-flex mb-3 ml-3">
          <button class="btn btn-primary mr-2" @click="abrirModalCadastro">
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
              <div class="d-flex justify-content-between align-items-start">
                <h1 class="mb-0">
                  <strong>
                    Mesa {{ mesa.deskNumber }} - {{ mesa.deskName || 'Mesa sem nome' }}
                  </strong>
                </h1>
                <div>
                  <button class="btn btn-sm btn-outline-primary mr-1" @click="abrirModalEdicao(mesa)" title="Editar mesa">
                    <i class="tim-icons icon-pencil"></i>
                  </button>
                  <button class="btn btn-sm btn-outline-danger" @click="confirmarExclusao(mesa)" title="Excluir mesa">
                    <i class="tim-icons icon-simple-remove"></i>
                  </button>
                </div>
              </div>
              <p class="mt-2 mb-1">Cliente: nome do cliente</p>
              <p class="mb-0">Último dia de aluguel: 01/01/2026</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <ModalCadastroMesas 
      :show="abrirModal" 
      :mesa-editando="mesaEditando" 
      @close="fecharModal"
      @mesa-salva="buscarMesas" 
    />
  </div>
</template>

<script>
import ModalCadastroMesas from "@/components/ModalCadastroMesas.vue";
import axios from "axios";
import Swal from 'sweetalert2';

export default {
  name: "Mesas",
  components: {
    ModalCadastroMesas
  },
  data() {
    return {
      abrirModal: false,
      mesaEditando: null,
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

        this.mesas = response.data.data || response.data || [];

      } catch (error) {
        console.error('Erro ao buscar mesas:', error);
        this.mostrarErro('Erro ao carregar mesas: ' + (error.response?.data?.message || error.message));
      } finally {
        this.loading = false;
      }
    },

    abrirModalCadastro() {
      this.mesaEditando = null;
      this.abrirModal = true;
    },

    abrirModalEdicao(mesa) {
      this.mesaEditando = {
        idDesk: mesa.idDesk,
        deskNumber: mesa.deskNumber,
        deskName: mesa.deskName
      };
      this.abrirModal = true;
    },

    async confirmarExclusao(mesa) {
      const result = await Swal.fire({
        title: 'Confirmar Exclusão',
        html: `Tem certeza que deseja excluir a mesa <strong>"Mesa ${mesa.deskNumber} - ${mesa.deskName || 'Mesa sem nome'}"</strong>?`,
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
        confirmButtonText: 'Sim, excluir!',
        cancelButtonText: 'Cancelar',
        reverseButtons: true
      });

      if (result.isConfirmed) {
        await this.excluirMesa(mesa);
      }
    },

    async excluirMesa(mesa) {
      try {
        await axios.delete(
          `http://localhost/projetos/cowork-project-back/public/desks/${mesa.idDesk}`
        );

        Swal.fire({
          title: 'Excluído!',
          text: 'Mesa excluída com sucesso.',
          icon: 'success',
          timer: 2000,
          showConfirmButton: false
        });

        this.buscarMesas();

      } catch (error) {
        console.error('Erro ao excluir mesa:', error);
        Swal.fire({
          title: 'Erro!',
          text: 'Erro ao excluir mesa: ' + (error.response?.data?.message || error.message),
          icon: 'error',
          confirmButtonText: 'OK'
        });
      }
    },

    fecharModal() {
      this.abrirModal = false;
      this.mesaEditando = null;
    },

    mostrarErro(mensagem) {
      this.$notify({
        type: 'danger',
        message: mensagem,
        icon: 'tim-icons icon-alert-circle-exc',
        horizontalAlign: 'right',
        verticalAlign: 'top'
      });
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

.mesa-locked {
  border-color: red !important;
}
</style>