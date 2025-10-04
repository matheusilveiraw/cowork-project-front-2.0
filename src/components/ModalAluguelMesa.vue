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
                <label>Selecionar Data de Início *</label>
                
                <div class="calendar-header d-flex justify-content-between align-items-center mb-3 p-2 bg-light rounded">
                  <button 
                    class="btn btn-sm btn-outline-secondary" 
                    @click="mesAnterior"
                    :disabled="loading"
                  >
                    <i class="tim-icons icon-minimal-left"></i>
                  </button>
                  
                  <h6 class="mb-0 text-center flex-grow-1">
                    {{ mesAtualFormatado }}
                  </h6>
                  
                  <button 
                    class="btn btn-sm btn-outline-secondary" 
                    @click="proximoMes"
                    :disabled="loading"
                  >
                    <i class="tim-icons icon-minimal-right"></i>
                  </button>
                </div>

                <div class="calendar-weekdays d-flex mb-2">
                  <div 
                    v-for="dia in diasSemana" 
                    :key="dia"
                    class="calendar-weekday text-center flex-fill text-muted small"
                  >
                    {{ dia }}
                  </div>
                </div>

                <!-- Dias do Mês -->
                <div class="calendar-days">
                  <div 
                    v-for="(dia, index) in diasDoMes" 
                    :key="index"
                    class="calendar-day-row d-flex"
                  >
                    <div
                      v-for="celula in dia"
                      :key="celula.data ? celula.data.getTime() : `empty-${index}`"
                      class="calendar-day-cell flex-fill text-center p-2"
                      :class="getClasseDia(celula)"
                      @click="selecionarDia(celula)"
                    >
                      <div class="calendar-day-content">
                        <span class="day-number">{{ celula.numero || '' }}</span>
                        
                        <div v-if="celula.data" class="day-indicators">
                          <div 
                            v-if="getDisponibilidadeMesa(celula.data)"
                            class="availability-dot available"
                            title="Mesa disponível"
                          ></div>
                          <div 
                            v-else
                            class="availability-dot unavailable"
                            title="Mesa indisponível"
                          ></div>
                          
                          <div 
                            v-if="isDiaSelecionado(celula.data)"
                            class="selected-indicator"
                          ></div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div v-if="aluguel.dataInicio" class="mt-3 p-2 bg-primary text-white rounded">
                  <small>
                    <strong>Data selecionada:</strong> 
                    {{ formatarDataParaExibicao(aluguel.dataInicio) }}
                  </small>
                </div>

                <small class="text-muted d-block mt-2">
                  Clique em um dia para selecionar a data de início do aluguel
                </small>
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
    const hoje = new Date();
    
    return {
      loading: false,
      loadingMessage: "",
      showErrorAlert: false,
      errorMessage: "",
      
      aluguel: {
        idDesk: "",
        idCustomer: "",
        idPlan: "",
        startPeriod: ""
      },
      
      clientes: [],
      planos: [],
      carregandoClientes: false,
      carregandoPlanos: false,
      
      dataAtual: hoje,
      diasSemana: ['Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb'],
      
      disponibilidadeCache: new Map()
    };
  },
  computed: {
    formValido() {
      return this.aluguel.idDesk && this.aluguel.idCustomer && this.aluguel.idPlan && this.aluguel.dataInicio;
    },
    
    mesAtualFormatado() {
      return this.dataAtual.toLocaleDateString('pt-BR', { 
        month: 'long', 
        year: 'numeric' 
      }).replace(/^./, c => c.toUpperCase());
    },
    
    diasDoMes() {
      const year = this.dataAtual.getFullYear();
      const month = this.dataAtual.getMonth();
      
      const primeiroDia = new Date(year, month, 1);
      const ultimoDia = new Date(year, month + 1, 0);
      
      const primeiroDiaSemana = primeiroDia.getDay();
      
      const semanas = [];
      let semana = [];
      
      for (let i = 0; i < primeiroDiaSemana; i++) {
        semana.push({ vazio: true });
      }
      
      for (let dia = 1; dia <= ultimoDia.getDate(); dia++) {
        const data = new Date(year, month, dia);
        semana.push({
          data: data,
          numero: dia,
          hoje: this.isHoje(data),
          passado: this.isPassado(data)
        });
        
        if (semana.length === 7) {
          semanas.push(semana);
          semana = [];
        }
      }
      
      if (semana.length > 0) {
        while (semana.length < 7) {
          semana.push({ vazio: true });
        }
        semanas.push(semana);
      }
      
      return semanas;
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
    },
    
    'aluguel.idDesk'() {
      this.disponibilidadeCache.clear();
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
        // console.error('Erro ao buscar clientes:', error);
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
        // console.error('Erro ao buscar planos:', error);
      } finally {
        this.carregandoPlanos = false;
      }
    },
    
    mesAnterior() {
      this.dataAtual = new Date(
        this.dataAtual.getFullYear(),
        this.dataAtual.getMonth() - 1,
        1
      );
    },
    
    proximoMes() {
      this.dataAtual = new Date(
        this.dataAtual.getFullYear(),
        this.dataAtual.getMonth() + 1,
        1
      );
    },
    
    isHoje(data) {
      const hoje = new Date();
      return data.toDateString() === hoje.toDateString();
    },
    
    isPassado(data) {
      const hoje = new Date();
      hoje.setHours(0, 0, 0, 0);
      const dataComparar = new Date(data);
      dataComparar.setHours(0, 0, 0, 0);
      return dataComparar < hoje;
    },
    
    isDiaSelecionado(data) {
      if (!data || !this.aluguel.dataInicio) return false;
      
      const dataSelecionada = new Date(this.aluguel.dataInicio);
      return data.toDateString() === dataSelecionada.toDateString();
    },
    
    getClasseDia(celula) {
      if (celula.vazio) return 'calendar-day-empty';
      
      const classes = ['calendar-day'];
      
      if (celula.hoje) classes.push('calendar-day-today');
      if (celula.passado) classes.push('calendar-day-past');
      if (this.isDiaSelecionado(celula.data)) classes.push('calendar-day-selected');
      
      if (!this.getDisponibilidadeMesa(celula.data)) {
        classes.push('calendar-day-unavailable');
      }
      
      return classes.join(' ');
    },
    
    selecionarDia(celula) {
      if (celula.vazio || celula.passado || !this.getDisponibilidadeMesa(celula.data)) {
        return;
      }
      
      const dataFormatada = celula.data.toISOString().split('T')[0];
      this.aluguel.dataInicio = dataFormatada;
    },
    
    getDisponibilidadeMesa(data) {
      if (!this.aluguel.idDesk) return true; 
      
      const chave = `${data.toDateString()}-${this.aluguel.idDesk}`;
      
      if (this.disponibilidadeCache.has(chave)) {
        return this.disponibilidadeCache.get(chave);
      }
      
      const disponivel = !this.isFimDeSemana(data) && Math.random() > 0.3;
      
      this.disponibilidadeCache.set(chave, disponivel);
      return disponivel;
    },
    
    isFimDeSemana(data) {
      const dia = data.getDay();
      return dia === 0 || dia === 6;
    },
    
    formatarDataParaExibicao(dataString) {
      const data = new Date(dataString + 'T00:00:00');
      return data.toLocaleDateString('pt-BR', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      }).replace(/^./, c => c.toUpperCase());
    },

    async salvarAluguel() {
      this.loading = true;
      this.loadingMessage = "Salvando aluguel...";
      this.showErrorAlert = false;

      try {
        const dadosAluguel = {
          idDesk: this.aluguel.idDesk,
          idCustomer: this.aluguel.idCustomer,
          idPlan: this.aluguel.idPlan,
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
        console.error('Erro ao salvar aluguel:', error);
        
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
      this.dataAtual = new Date();
      this.disponibilidadeCache.clear();
      this.showErrorAlert = false;
    },

    mostrarErro(mensagem) {
      this.errorMessage = mensagem;
      this.showErrorAlert = true;
    }
  }
};
</script>

<style scoped>
.calendar-header {
  background-color: #f8f9fa;
  border: 1px solid #dee2e6;
}

.calendar-weekday {
  font-weight: 600;
  font-size: 0.8rem;
  padding: 0.25rem;
}

.calendar-day-cell {
  min-height: 60px;
  border: 1px solid #e9ecef;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.calendar-day-empty {
  background-color: #f8f9fa;
  cursor: default;
}

.calendar-day {
  background-color: white;
}

.calendar-day:hover:not(.calendar-day-past):not(.calendar-day-unavailable) {
  background-color: #e3f2fd;
  border-color: #2196f3;
}

.calendar-day-today {
  background-color: #fff3cd;
  border-color: #ffc107;
}

.calendar-day-past {
  background-color: #f8f9fa;
  color: #6c757d;
  cursor: not-allowed;
}

.calendar-day-selected {
  background-color: #2196f3 !important;
  color: white !important;
  border-color: #1976d2 !important;
}

.calendar-day-unavailable {
  background-color: #ffebee;
  color: #c62828;
  cursor: not-allowed;
  position: relative;
}

.calendar-day-unavailable::after {
  content: "✕";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.2rem;
  color: #c62828;
  opacity: 0.3;
}

.calendar-day-content {
  position: relative;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.day-number {
  font-weight: 600;
  font-size: 0.9rem;
}

.day-indicators {
  position: absolute;
  bottom: 2px;
  right: 2px;
  display: flex;
  gap: 2px;
}

.availability-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
}

.availability-dot.available {
  background-color: #4caf50;
}

.availability-dot.unavailable {
  background-color: #f44336;
}

.selected-indicator {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: white;
  border: 1px solid #1976d2;
}

.calendar-day-selected .day-number {
  color: white;
  font-weight: bold;
}

.calendar-day-selected .availability-dot {
  background-color: white;
}
</style>

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
  max-width: 800px; 
}
</style>