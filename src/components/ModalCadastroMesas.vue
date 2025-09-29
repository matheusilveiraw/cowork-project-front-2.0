<template>
  <div>
    <Loading :show="loading" :message="loadingMessage" />

    <base-alert 
      v-if="showSuccessAlert" 
      type="info" 
      dismissible 
      with-icon
      @dismiss="showSuccessAlert = false"
    >
      <span data-notify="icon" class="tim-icons icon-bell-55"></span>
      <span data-notify="message">Mesa salva com sucesso!</span>
    </base-alert>

    <base-alert 
      v-if="showErrorAlert" 
      type="danger" 
      dismissible 
      with-icon
      @dismiss="showErrorAlert = false"
    >
      <span data-notify="icon" class="tim-icons icon-alert-circle-exc"></span>
      <span data-notify="message">{{ errorMessage }}</span>
    </base-alert>


    <div v-if="show" class="modal-backdrop">
      <div class="modal-dialog modal-xl">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Cadastrar Nova Mesa</h5>
            <button type="button" class="close" @click="$emit('close')">
              <span>&times;</span>
            </button>
          </div>

          <div class="modal-body">
            <form>
              <div class="form-group">
                <label for="numeroMesa">Número da Mesa</label>
                <input type="number" class="form-control" id="numeroMesa" v-model="mesa.deskNumber"
                  :disabled="loading" />
              </div>
              <div class="form-group">
                <label for="nomeMesa">Nome da Mesa</label>
                <input type="text" class="form-control" id="nomeMesa" v-model="mesa.deskName" :disabled="loading">
              </div>
            </form>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="$emit('close')" :disabled="loading">
              Cancelar
            </button>
            <button type="button" class="btn btn-primary" @click="salvarMesa" :disabled="loading">
              <span v-if="loading">Salvando...</span>
              <span v-else>Salvar</span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { BaseAlert } from "@/components";

export default {
  name: "ModalCadastroMesas",
    components: { 
    BaseAlert
  },
  props: {
    show: Boolean
  },
  data() {
    return {
      loading: false,
      loadingMessage: "Salvando mesa...",
      showSuccessAlert: false,
      showErrorAlert: false,
      errorMessage: "",
      mesa: {
        deskNumber: "",
        deskName: ""
      }
    };
  },
  methods: {
    async salvarMesa() {
      this.loading = true;
      this.loadingMessage = "Salvando mesa...";
      this.showSuccessAlert = false;
      this.showErrorAlert = false;

      try {
        const response = await axios.post(
          "http://localhost/projetos/cowork-project-back/public/desks", 
          this.mesa,
          {
            headers: {
              'Content-Type': 'application/json',
            },
            timeout: 10000
          }
        );
        
        console.log('Sucesso:', response.data);
        
        // Mostra alerta de sucesso
        this.showSuccessAlert = true;
        
        // Fecha o modal após 2 segundos
        setTimeout(() => {
          this.fecharModal();
          this.$emit('mesa-salva');
        }, 2000);
        
      } catch (error) {
        console.error('Erro completo:', error);
        
        if (error.response) {
          const status = error.response.status;
          const message = error.response.data.message || 'Erro desconhecido';
          
          if (status === 409) {
            this.errorMessage = 'Já existe uma mesa com este número. Escolha outro número.';
          } else if (status === 400) {
            this.errorMessage = 'Dados inválidos. Verifique os campos preenchidos.';
          } else {
            this.errorMessage = 'Erro ao salvar mesa: ' + message;
          }
        } else if (error.request) {
          this.errorMessage = 'Não foi possível conectar ao servidor. Verifique sua conexão.';
        } else {
          this.errorMessage = 'Erro: ' + error.message;
        }
        
        this.showErrorAlert = true;
        
      } finally {
        this.loading = false;
      }
    },
    
    fecharModal() {
      // Reseta os estados antes de fechar
      this.showSuccessAlert = false;
      this.showErrorAlert = false;
      this.errorMessage = "";
      this.mesa.deskNumber = "";
      this.mesa.deskName = "";
      this.$emit('close');
    }
  },
  
  watch: {
    show(newVal) {
      if (!newVal) {
        // Quando o modal fecha, reseta os alertas
        this.showSuccessAlert = false;
        this.showErrorAlert = false;
        this.errorMessage = "";
      }
    }
  }
};
</script>

<style>
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1050;
}

.modal-dialog {
  width: 80%;
}

.modal-content .form-control {
  color: #000 !important;
  background-color: #fff;
}
</style>
