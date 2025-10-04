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
          <button class="btn btn-secondary mr-2" @click="abrirModalAluguelGeral">
            Alugar Mesa
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
          <div class="col-md-6 mb-3" v-for="mesa in mesas" :key="mesa.idDesk">
            <div class="mesa-box p-3">
              <div class="d-flex justify-content-between align-items-start">
                <h1 class="mb-0">
                  <strong>
                    Mesa {{ mesa.deskNumber }} - {{ mesa.deskName || 'Mesa sem nome' }}
                  </strong>
                </h1>
                <div>
                  <button class="btn btn-sm btn-outline-success mr-1" @click="abrirModalAluguel(mesa)" title="Alugar mesa">
                    <i class="tim-icons icon-tag"></i>
                  </button>
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
      :show="abrirModalCadastroMesa" 
      :mesa-editando="mesaEditando" 
      @close="fecharModalCadastro"
      @mesa-salva="buscarMesas" 
    />

    <ModalAluguelMesa
      :show="abrirModalAluguelMesa"
      :mesa-pre-selecionada="mesaSelecionadaAluguel"
      :mesas="mesas"
      @close="fecharModalAluguel"
      @aluguel-salvo="onAluguelSalvo"
    />
  </div>
</template>

<script>
import ModalCadastroMesas from "@/components/ModalCadastroMesas.vue";
import ModalAluguelMesa from "@/components/ModalAluguelMesa.vue";
import axios from "axios";
import Swal from 'sweetalert2';

export default {
  name: "Mesas",
  components: {
    ModalCadastroMesas,
    ModalAluguelMesa
  },
  data() {
    return {
      abrirModalCadastroMesa: false,
      abrirModalAluguelMesa: false,
      mesaEditando: null,
      mesaSelecionadaAluguel: null,
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
      this.abrirModalCadastroMesa = true;
    },

    abrirModalEdicao(mesa) {
      this.mesaEditando = {
        idDesk: mesa.idDesk,
        deskNumber: mesa.deskNumber,
        deskName: mesa.deskName
      };
      this.abrirModalCadastroMesa = true;
    },

    // Abre modal de aluguel COM mesa específica pré-selecionada
    abrirModalAluguel(mesa) {
      this.mesaSelecionadaAluguel = {
        idDesk: mesa.idDesk,
        deskNumber: mesa.deskNumber,
        deskName: mesa.deskName
      };
      this.abrirModalAluguelMesa = true;
    },

    // Abre modal de aluguel SEM mesa pré-selecionada
    abrirModalAluguelGeral() {
      this.mesaSelecionadaAluguel = null;
      this.abrirModalAluguelMesa = true;
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

    fecharModalCadastro() {
      this.abrirModalCadastroMesa = false;
      this.mesaEditando = null;
    },

    fecharModalAluguel() {
      this.abrirModalAluguelMesa = false;
      this.mesaSelecionadaAluguel = null;
    },

  onAluguelSalvo(dadosAluguel) {
    
    this.buscarMesas();
    
    Swal.fire({
      title: 'Sucesso!',
      text: 'Mesa alugada com sucesso.',
      icon: 'success',
      timer: 3000,
      showConfirmButton: false
    });
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