<template>
  <div>
    <Loading :show="loading" :message="loadingMessage" />

    <base-alert v-if="showErrorAlert" type="danger" dismissible with-icon class="modal-alert"
      @dismiss="showErrorAlert = false">
      <span data-notify="icon" class="tim-icons icon-alert-circle-exc"></span>
      <span data-notify="message">{{ errorMessage }}</span>
    </base-alert>

    <div v-if="show" class="modal-backdrop">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Alugar Mesa</h5>
            <button type="button" class="close" @click="fecharModal">
              <span>&times;</span>
            </button>
          </div>

          <div class="modal-body">
            <form>
              <div class="form-group">
                <label for="mesaSelect">Selecionar Mesa *</label>
                <select 
                  class="form-control" 
                  id="mesaSelect" 
                  v-model="aluguel.idDesk"
                  :disabled="loading"
                >
                  <option value="">Selecione uma mesa</option>
                  <option 
                    v-for="mesa in mesas" 
                    :key="mesa.idDesk" 
                    :value="mesa.idDesk"
                  >
                    Mesa {{ mesa.deskNumber }} - {{ mesa.deskName || 'Mesa sem nome' }}
                  </option>
                </select>
              </div>

              <div class="form-group">
                <label for="clienteSelect">Selecionar Cliente *</label>
                <select 
                  class="form-control" 
                  id="clienteSelect" 
                  v-model="aluguel.idCustomer"
                  :disabled="loading || carregandoClientes"
                >
                  <option value="">Selecione um cliente</option>
                  <option 
                    v-for="cliente in clientes" 
                    :key="cliente.idCustomer" 
                    :value="cliente.idCustomer"
                  >
                    {{ cliente.nameCustomer }} - {{ cliente.emailCustomer }}
                  </option>
                </select>
                <small v-if="carregandoClientes" class="text-muted">Carregando clientes...</small>
              </div>

              <div class="form-group">
                <label for="planoSelect">Selecionar Plano *</label>
                <select 
                  class="form-control" 
                  id="planoSelect" 
                  v-model="aluguel.idPlan"
                  :disabled="loading || carregandoPlanos"
                >
                  <option value="">Selecione um plano</option>
                  <option 
                    v-for="plano in planos" 
                    :key="plano.idPlan" 
                    :value="plano.idPlan"
                  >
                    {{ plano.planName }} - R$ {{ plano.price }}
                  </option>
                </select>
                <small v-if="carregandoPlanos" class="text-muted">Carregando planos...</small>
              </div>

              <div class="form-group">
                <label for="dataInicio">Data de Início do Aluguel *</label>
                <input 
                  type="date" 
                  class="form-control" 
                  id="dataInicio" 
                  v-model="aluguel.dataInicio"
                  :min="dataMinima"
                  :disabled="loading"
                >
                <small class="text-muted">Selecione a data em que o aluguel começará</small>
              </div>
            </form>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="fecharModal" :disabled="loading">
              Cancelar
            </button>
            <button 
              type="button" 
              class="btn btn-primary" 
              @click="salvarAluguel" 
              :disabled="loading || !formValido"
            >
              <span v-if="loading">Salvando...</span>
              <span v-else>Confirmar Aluguel</span>
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
  name: "ModalAluguelMesa",
  components: {
    BaseAlert
  },
  props: {
    show: Boolean,
    mesaPreSelecionada: {
      type: Object,
      default: null
    },
    mesas: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      loading: false,
      loadingMessage: "",
      showErrorAlert: false,
      errorMessage: "",
      
      aluguel: {
        idDesk: "",
        idCustomer: "",
        idPlan: "",
        dataInicio: ""
      },
      
      clientes: [],
      planos: [],
      carregandoClientes: false,
      carregandoPlanos: false
    };
  },
  computed: {
    formValido() {
      return this.aluguel.idDesk && this.aluguel.idCustomer && this.aluguel.idPlan && this.aluguel.dataInicio;
    },
    dataMinima() {
      return new Date().toISOString().split('T')[0];
    }
  },
  watch: {
    show(newVal) {
      if (newVal) {
        this.inicializarMesa();
        this.carregarDados();
      } else {
        this.limparFormulario();
      }
    }
  },
  methods: {
    inicializarMesa() {
      if (this.mesaPreSelecionada) {
        this.aluguel.idDesk = this.mesaPreSelecionada.idDesk;
      } else {
        this.aluguel.idDesk = "";
      }
    },

    async carregarDados() {
      await Promise.all([
        this.buscarClientes(),
        this.buscarPlanos()
      ]);
    },

    async buscarClientes() {
      this.carregandoClientes = true;
      try {
        const response = await axios.get(
          "http://localhost/projetos/cowork-project-back/public/customers"
        );
        this.clientes = response.data.data || response.data || [];
      } catch (error) {
        this.mostrarErro('Erro ao carregar clientes');
      } finally {
        this.carregandoClientes = false;
      }
    },

    async buscarPlanos() {
      this.carregandoPlanos = true;
      try {
        const response = await axios.get(
          "http://localhost/projetos/cowork-project-back/public/rental-plans"
        );
        this.planos = response.data.data || response.data || [];
      } catch (error) {
        this.planos = await this.getPlanosMock();
      } finally {
        this.carregandoPlanos = false;
      }
    },

    async getPlanosMock() {
      return [
        { idPlan: 1, planName: "Diária - Manhã", price: 50.00 },
        { idPlan: 2, planName: "Diária - Tarde", price: 50.00 },
        { idPlan: 3, planName: "Diária - Integral", price: 80.00 }
      ];
    },

    async salvarAluguel() {
      this.loading = true;
      this.loadingMessage = "Salvando aluguel...";
      this.showErrorAlert = false;

      try {
        const dadosAluguel = {
          idDesk: parseInt(this.aluguel.idDesk),
          idCustomer: parseInt(this.aluguel.idCustomer),
          idPlan: parseInt(this.aluguel.idPlan),
          startPeriod: `${this.aluguel.dataInicio} 08:00:00`
        };

        const response = await axios.post(
          "http://localhost/projetos/cowork-project-back/public/deskrentals",
          dadosAluguel,
          {
            headers: {
              'Content-Type': 'application/json',
            },
            timeout: 10000
          }
        );

        this.$emit('aluguel-salvo', response.data);
        this.fecharModal();

      } catch (error) {      
        if (error.response) {
          const status = error.response.status;
          const message = error.response.data?.error || error.response.data?.message || 'Erro desconhecido';

          if (status === 400) {
            this.errorMessage = message;
          } else if (status === 409) {
            this.errorMessage = 'Mesa já está alugada neste período';
          } else {
            this.errorMessage = `Erro ao salvar aluguel: ${message}`;
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
      this.limparFormulario();
      this.$emit('close');
    },

    limparFormulario() {
      this.aluguel = {
        idDesk: "",
        idCustomer: "",
        idPlan: "",
        dataInicio: ""
      };
      this.showErrorAlert = false;
    },

    mostrarErro(mensagem) {
      this.errorMessage = mensagem;
      this.showErrorAlert = true;
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
  max-width: 600px;
}
</style>