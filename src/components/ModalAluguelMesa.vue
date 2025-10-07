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

          <div class="modal-body modal-body-scrollable">
            <form>
              <div class="form-group">
                <label for="mesaSelect">Selecionar Mesa *</label>
                <select class="form-control" id="mesaSelect" v-model="aluguel.idDesk" :disabled="loading"
                  @change="onMesaChange">
                  <option value="">Selecione uma mesa</option>
                  <option v-for="mesa in mesas" :key="mesa.idDesk" :value="mesa.idDesk">
                    Mesa {{ mesa.deskNumber }} - {{ mesa.deskName || 'Mesa sem nome' }}
                  </option>
                </select>
              </div>

              <div class="form-group">
                <label for="clienteSelect">Selecionar Cliente *</label>
                <select class="form-control" id="clienteSelect" v-model="aluguel.idCustomer"
                  :disabled="loading || carregandoClientes">
                  <option value="">Selecione um cliente</option>
                  <option v-for="cliente in clientes" :key="cliente.idCustomer" :value="cliente.idCustomer">
                    {{ cliente.nameCustomer }} - {{ cliente.emailCustomer }}
                  </option>
                </select>
                <small v-if="carregandoClientes" class="text-muted">Carregando clientes...</small>
              </div>

              <div class="form-group">
                <label for="planoSelect">Selecionar Plano *</label>
                <select class="form-control" id="planoSelect" v-model="aluguel.idPlan"
                  :disabled="loading || carregandoPlanos">
                  <option value="">Selecione um plano</option>
                  <option v-for="plano in planos" :key="plano.idPlan" :value="plano.idPlan">
                    {{ plano.planName }} - R$ {{ plano.price }}
                  </option>
                </select>
                <small v-if="carregandoPlanos" class="text-muted">Carregando planos...</small>
              </div>

              <div class="form-group">
                <label for="dataInicio">Data de Início do Aluguel *</label>
                <input type="date" class="form-control" id="dataInicio" v-model="aluguel.dataInicio" :min="dataMinima"
                  :disabled="loading">
                <small class="text-muted">Selecione a data em que o aluguel começará</small>
              </div>

              <div v-if="aluguel.idDesk" class="form-group">
                <label>Visualizar Disponibilidade da Mesa</label>

                <div
                  class="calendar-header d-flex justify-content-between align-items-center mb-3 p-2 bg-light rounded">
                  <button class="btn btn-sm btn-outline-secondary" @click="mesAnterior"
                    :disabled="loading || carregandoDisponibilidade">
                    <i class="tim-icons icon-minimal-left"></i>
                  </button>

                  <h6 class="mb-0 text-center flex-grow-1">
                    {{ mesAtualFormatado }}
                    <small v-if="carregandoDisponibilidade" class="text-muted d-block">
                      Carregando disponibilidade...
                    </small>
                  </h6>

                  <button class="btn btn-sm btn-outline-secondary" @click="proximoMes"
                    :disabled="loading || carregandoDisponibilidade">
                    <i class="tim-icons icon-minimal-right"></i>
                  </button>
                </div>

                <!-- Dias da semana -->
                <div class="calendar-weekdays d-flex mb-2">
                  <div v-for="dia in diasSemana" :key="dia"
                    class="calendar-weekday text-center flex-fill text-muted small font-weight-bold py-2">
                    {{ dia }}
                  </div>
                </div>

                <div class="calendar-days">
                  <div v-for="(semana, semanaIndex) in diasDoMes" :key="semanaIndex" class="calendar-day-row d-flex">
                    <div v-for="(dia, diaIndex) in semana" :key="`${semanaIndex}-${diaIndex}`"
                      class="calendar-day-cell flex-fill text-center position-relative" :class="getClasseDia(dia)">
                      <div
                        class="calendar-day-content d-flex flex-column align-items-center justify-content-center w-100 h-100 p-2">
                        <span class="day-number">{{ dia.numero || '' }}</span>

                        <div v-if="!dia.vazio" class="day-status mt-1">
                          <div class="d-flex flex-column gap-1">
                            <span v-if="dia.ocupacao?.manha" class="badge badge-sm" :class="{
                              'badge-warning': dia.ocupacao.manha === 'parcial',
                              'badge-danger': dia.ocupacao.manha === 'integral'
                            }">
                              {{ dia.ocupacao.manha === 'parcial' ? 'Manhã' : 'M' }}
                            </span>

                            <span v-if="dia.ocupacao?.tarde" class="badge badge-sm" :class="{
                              'badge-warning': dia.ocupacao.tarde === 'parcial',
                              'badge-danger': dia.ocupacao.tarde === 'integral'
                            }">
                              {{ dia.ocupacao.tarde === 'parcial' ? 'Tarde' : 'T' }}
                            </span>

                            <span v-if="!dia.ocupacao?.manha && !dia.ocupacao?.tarde"
                              class="badge badge-success badge-sm">
                              Livre
                            </span>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <div class="mt-3 p-2 border rounded">
                  <small class="text-muted d-block mb-1"><strong>Legenda:</strong></small>
                  <div class="d-flex flex-wrap gap-2">
                    <div class="d-flex align-items-center">
                      <span class="badge badge-success badge-sm mr-1">Livre</span>
                      <small>Dia disponível</small>
                    </div>
                    <div class="d-flex align-items-center">
                      <span class="badge badge-warning badge-sm mr-1">Manhã</span>
                      <small>Manhã ocupada</small>
                    </div>
                    <div class="d-flex align-items-center">
                      <span class="badge badge-warning badge-sm mr-1">Tarde</span>
                      <small>Tarde ocupada</small>
                    </div>
                    <div class="d-flex align-items-center">
                      <span class="badge badge-danger badge-sm mr-1">M/T</span>
                      <small>Dia todo ocupado</small>
                    </div>
                  </div>
                </div>

                <small class="text-muted d-block mt-2">
                  Use as setas para navegar entre os meses e visualizar a disponibilidade
                </small>
              </div>

              <div v-else class="alert alert-info">
                <small>Selecione uma mesa para visualizar a disponibilidade no calendário</small>
              </div>
            </form>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="fecharModal" :disabled="loading">
              Cancelar
            </button>
            <button type="button" class="btn btn-primary" @click="salvarAluguel" :disabled="loading || !formValido">
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
      carregandoDisponibilidade: false,

      aluguel: {
        idDesk: "",
        idCustomer: "",
        idPlan: "",
        dataInicio: ""
      },

      clientes: [],
      planos: [],
      carregandoClientes: false,
      carregandoPlanos: false,

      dataAtual: new Date(hoje.getFullYear(), hoje.getMonth(), 1),
      diasSemana: ['Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sáb'],
      disponibilidadeMes: new Map(),

      disponibilidadeCache: new Map()
    };
  },
  computed: {
    formValido() {
      return this.aluguel.idDesk && this.aluguel.idCustomer && this.aluguel.idPlan && this.aluguel.dataInicio;
    },
    dataMinima() {
      return new Date().toISOString().split('T')[0];
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
        semana.push({
          vazio: true,
          numero: ''
        });
      }

      for (let dia = 1; dia <= ultimoDia.getDate(); dia++) {
        const data = new Date(year, month, dia);
        const dataString = data.toISOString().split('T')[0];
        const ocupacao = this.disponibilidadeMes.get(dataString) || {};

        semana.push({
          data: data,
          numero: dia,
          dataString: dataString,
          hoje: this.isHoje(data),
          passado: this.isPassado(data),
          vazio: false,
          ocupacao: ocupacao,
          livre: !ocupacao.manha && !ocupacao.tarde
        });

        if (semana.length === 7) {
          semanas.push(semana);
          semana = [];
        }
      }

      if (semana.length > 0) {
        while (semana.length < 7) {
          semana.push({
            vazio: true,
            numero: ''
          });
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

    dataAtual() {
      this.carregarDisponibilidadeMes();
    }
  },
  methods: {
    inicializarMesa() {
      if (this.mesaPreSelecionada) {
        this.aluguel.idDesk = this.mesaPreSelecionada.idDesk;
        this.onMesaChange();
      } else {
        this.aluguel.idDesk = "";
        this.disponibilidadeMes.clear();
      }
    },

    onMesaChange() {
      this.disponibilidadeMes.clear();
      this.disponibilidadeCache.clear();
      this.aluguel.dataInicio = "";

      if (this.aluguel.idDesk) {
        this.carregarDisponibilidadeMes();
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
        console.error('Erro ao buscar planos:', error);
      } finally {
        this.carregandoPlanos = false;
      }
    },

    async carregarDisponibilidadeMes() {
      if (!this.aluguel.idDesk) {
        this.disponibilidadeMes.clear();
        return;
      }

      const chaveCache = `${this.aluguel.idDesk}-${this.dataAtual.getFullYear()}-${this.dataAtual.getMonth()}`;

      if (this.disponibilidadeCache.has(chaveCache)) {
        const cachedData = this.disponibilidadeCache.get(chaveCache);
        this.disponibilidadeMes = new Map(cachedData);
        return;
      }

      this.carregandoDisponibilidade = true;

      try {
        const year = this.dataAtual.getFullYear();
        const month = this.dataAtual.getMonth();

        const primeiroDia = new Date(year, month, 1);
        const ultimoDia = new Date(year, month + 1, 0);

        const response = await axios.post(
          "http://localhost/projetos/cowork-project-back/public/deskrentals/check-month-availability",
          {
            idDesk: parseInt(this.aluguel.idDesk),
            startDate: primeiroDia.toISOString().split('T')[0],
            endDate: ultimoDia.toISOString().split('T')[0]
          }
        );

        const disponibilidade = response.data.disponibilidade || {};
        const novoMap = new Map();

        Object.keys(disponibilidade).forEach(dataString => {
          novoMap.set(dataString, disponibilidade[dataString]);
        });

        this.disponibilidadeMes = novoMap;

        this.disponibilidadeCache.set(chaveCache, [...novoMap]);

      } catch (error) {
        console.error('Erro ao carregar disponibilidade:', error);
        this.disponibilidadeMes.clear();
      } finally {
        this.carregandoDisponibilidade = false;
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

    getClasseDia(dia) {
      if (dia.vazio) return 'calendar-day-empty';

      const classes = ['calendar-day'];

      if (dia.hoje) classes.push('calendar-day-today');
      if (dia.passado) classes.push('calendar-day-past');
      if (!dia.livre) classes.push('calendar-day-occupied');

      return classes.join(' ');
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
      this.dataAtual = new Date(new Date().getFullYear(), new Date().getMonth(), 1);
      this.disponibilidadeMes.clear();
      this.disponibilidadeCache.clear();
      this.showErrorAlert = false;
      this.carregandoDisponibilidade = false;
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
  border-bottom: 2px solid #dee2e6;
}

.calendar-day-cell {
  height: 90px;
  border: 1px solid #e9ecef;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  flex: 1;
  min-width: 0;
}

.calendar-day-empty {
  background-color: #f8f9fa;
  border: 1px solid #f8f9fa;
}

.calendar-day {
  background-color: white;
}

.calendar-day-today {
  background-color: #fff3cd;
  border-color: #ffc107;
  font-weight: bold;
}

.calendar-day-past {
  background-color: #f8f9fa;
  color: #6c757d;
}

.calendar-day-occupied {
  background-color: #ffebee;
  color: #c62828;
}

.calendar-day-content {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.day-number {
  font-weight: 600;
  font-size: 0.9rem;
  line-height: 1;
}

.day-status {
  font-size: 0.7rem;
}

.badge-sm {
  font-size: 0.6rem;
  padding: 0.2em 0.4em;
}

.calendar-day-row {
  display: flex;
  min-height: 90px;
}

.calendar-days {
  border: 1px solid #e9ecef;
  border-radius: 4px;
  overflow: hidden;
}

.calendar-day-cell {
  flex: 1 1 calc(100% / 7);
  max-width: calc(100% / 7);
}

.calendar-weekday {
  flex: 1 1 calc(100% / 7);
  max-width: calc(100% / 7);
}

.modal-body-scrollable {
  max-height: 60vh;
  overflow-y: auto;
  padding-right: 5px;
}

.modal-body-scrollable::-webkit-scrollbar {
  width: 6px;
}

.modal-body-scrollable::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.modal-body-scrollable::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

.modal-body-scrollable::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
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
  padding: 20px;
}

.modal-dialog {
  width: 90%;
  max-width: 800px;
  margin: auto;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
}

.modal-content {
  max-height: 90vh;
  display: flex;
  flex-direction: column;
}

.modal-body {
  flex: 1;
  overflow: hidden;
}
</style>