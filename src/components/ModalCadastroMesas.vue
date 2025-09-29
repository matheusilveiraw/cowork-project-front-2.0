<template>
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
              <input 
                type="number" 
                class="form-control" 
                id="numeroMesa" 
                v-model="mesa.deskNumber"
              />
            </div>
            <div class="form-group">
              <label for="nomeMesa">Nome da Mesa</label>
              <input 
                type="text" 
                class="form-control" 
                id="nomeMesa" 
                v-model="mesa.deskName"
              />
            </div>
          </form>
        </div>

        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="$emit('close')">
            Cancelar
          </button>
          <button type="button" class="btn btn-primary" @click="salvarMesa">
            Salvar
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ModalCadastroMesas",
  props: {
    show: Boolean
  },
  data() {
    return {
      mesa: {
        deskNumber: "",
        deskName: ""
      }
    };
  },
  methods: {
    async salvarMesa() {
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
        } catch (error) {
            console.error('Erro completo:', error);
            
            // Teste se a URL está acessível
            try {
                const testResponse = await fetch("http://localhost/projetos/cowork-project-back/public/desks", {
                    method: 'GET'
                });
                console.log('Teste URL status:', testResponse.status);
            } catch (testError) {
                console.error('Teste URL falhou:', testError);
            }
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
  background: rgba(0,0,0,0.6);
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
