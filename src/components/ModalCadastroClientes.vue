<template>
  <div>
    <Loading :show="loading" :message="loadingMessage" />

    <base-alert v-if="showSuccessAlert" type="info" dismissible with-icon class="success-alert"
      @dismiss="showSuccessAlert = false">
      <span data-notify="icon" class="tim-icons icon-bell-55"></span>
      <span data-notify="message">{{ mensagemSucesso }}</span>
    </base-alert>

    <base-alert v-if="showErrorAlert" type="danger" dismissible with-icon class="modal-alert"
      @dismiss="showErrorAlert = false">
      <span data-notify="icon" class="tim-icons icon-alert-circle-exc"></span>
      <span data-notify="message">{{ errorMessage }}</span>
    </base-alert>

    <div v-if="show" class="modal-backdrop">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">{{ tituloModal }}</h5>
            <button type="button" class="close" @click="fecharModal">
              <span>&times;</span>
            </button>
          </div>

          <div class="modal-body">
            <form>
              <div class="form-group">
                <label for="nomeCliente">Nome do Cliente *</label>
                <input type="text" class="form-control" id="nomeCliente" v-model="cliente.nameCustomer"
                  :disabled="loading" required />
              </div>
              <div class="form-group">
                <label for="emailCliente">Email *</label>
                <input type="email" class="form-control" id="emailCliente" v-model="cliente.emailCustomer"
                  :disabled="loading" required>
              </div>
              <div class="form-group">
                <label for="telefoneCliente">Telefone</label>
                <input type="text" class="form-control" id="telefoneCliente" v-model="cliente.phoneCustomer"
                  :disabled="loading" placeholder="(11) 99999-9999">
              </div>
              <div class="form-group">
                <label for="enderecoCliente">Endereço</label>
                <textarea class="form-control" id="enderecoCliente" v-model="cliente.addressCustomer" :disabled="loading"
                  rows="3" placeholder="Rua, número - Bairro - Cidade/Estado"></textarea>
              </div>
            </form>
          </div>

          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="fecharModal" :disabled="loading">
              Cancelar
            </button>
            <button type="button" class="btn btn-primary" @click="salvarCliente" :disabled="loading || !formValido">
              <span v-if="loading">{{ textoBotaoSalvar }}</span>
              <span v-else>{{ textoBotao }}</span>
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
  name: "ModalCadastroClientes",
  components: {
    BaseAlert
  },
  props: {
    show: Boolean,
    clienteEditando: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      loading: false,
      loadingMessage: "",
      showSuccessAlert: false,
      showErrorAlert: false,
      errorMessage: "",
      mensagemSucesso: "",
      cliente: {
        nameCustomer: "",
        emailCustomer: "",
        phoneCustomer: "",
        addressCustomer: ""
      }
    };
  },
  computed: {
    tituloModal() {
      return this.clienteEditando ? 'Editar Cliente' : 'Cadastrar Novo Cliente';
    },
    textoBotao() {
      return this.clienteEditando ? 'Atualizar' : 'Salvar';
    },
    textoBotaoSalvar() {
      return this.clienteEditando ? 'Atualizando...' : 'Salvando...';
    },
    formValido() {
      return this.cliente.nameCustomer.trim() && this.cliente.emailCustomer.trim();
    }
  },
  watch: {
    clienteEditando: {
      handler(novoCliente) {
        // console.log('Cliente para edição:', novoCliente);
        
        if (novoCliente) {
          this.cliente = {
            idCustomer: novoCliente.idCustomer,
            nameCustomer: novoCliente.nameCustomer || "",
            emailCustomer: novoCliente.emailCustomer || "",
            phoneCustomer: novoCliente.phoneCustomer || "",
            addressCustomer: novoCliente.addressCustomer || ""
          };
        } else {
          this.cliente = {
            nameCustomer: "",
            emailCustomer: "",
            phoneCustomer: "",
            addressCustomer: ""
          };
        }
      },
      immediate: true
    },
    show(newVal) {
      if (!newVal) {
        this.showErrorAlert = false;
        this.errorMessage = "";
      }
    }
  },
  methods: {
    async salvarCliente() {
        this.loading = true;
        this.loadingMessage = this.clienteEditando ? "Atualizando cliente..." : "Salvando cliente...";
        this.showSuccessAlert = false;
        this.showErrorAlert = false;

        // console.log('=== DEBUG UPDATE ===');
        // console.log('Cliente editando:', this.clienteEditando);
        // console.log('Dados do formulário:', this.cliente);
        // console.log('ID do cliente:', this.clienteEditando?.idCustomer);

        try {
            let response;
            let clienteId;

            const dadosParaEnvio = {
            nameCustomer: this.cliente.nameCustomer.trim(),
            emailCustomer: this.cliente.emailCustomer.trim(),
            phoneCustomer: this.cliente.phoneCustomer ? this.cliente.phoneCustomer.trim() : null,
            addressCustomer: this.cliente.addressCustomer ? this.cliente.addressCustomer.trim() : null
            };

            // console.log('Dados sendo enviados:', dadosParaEnvio);
            // console.log('Editando cliente ID:', this.clienteEditando?.idCustomer);

            if (this.clienteEditando) {
            clienteId = this.clienteEditando.idCustomer;

            if (!clienteId) {
                throw new Error('ID do cliente não encontrado para edição');
            }

            // PUT para atualização
            response = await axios.put(
                `http://localhost/projetos/cowork-project-back/public/customers/${clienteId}`,
                dadosParaEnvio,
                {
                headers: {
                    'Content-Type': 'application/json',
                },
                timeout: 10000
                }
            );
            this.mensagemSucesso = "Cliente atualizado com sucesso!";
            } else {
            // POST para criação
            response = await axios.post(
                "http://localhost/projetos/cowork-project-back/public/customers",
                dadosParaEnvio,
                {
                headers: {
                    'Content-Type': 'application/json',
                },
                timeout: 10000
                }
            );
            this.mensagemSucesso = "Cliente salvo com sucesso!";
            }

            // console.log('Resposta da API:', response.data);

            this.showSuccessAlert = true;
            
            setTimeout(() => {
            this.fecharModal();
            }, 1500);

            this.$emit('cliente-salvo');

        } catch (error) {
            // console.error('Erro completo:', error);
            // console.error('Resposta do erro:', error.response);

            if (error.response) {
                const status = error.response.status;
                const errorData = error.response.data;

                // console.log('Dados do erro:', errorData);

                if (status === 400) {
                    if (errorData.errors) {
                        const erros = Object.values(errorData.errors).join(', ');
                        this.errorMessage = `Erro de validação: ${erros}`;
                    } else if (errorData.error) {
                        this.errorMessage = errorData.error;
                    } else {
                        this.errorMessage = 'Dados inválidos. Verifique os campos preenchidos.';
                    }
                } else if (status === 409) {
                    this.errorMessage = errorData.error || 'Já existe um cliente com este email';
                } else if (status === 404) {
                    this.errorMessage = 'Cliente não encontrado';
                } else {
                    this.errorMessage = `Erro ao ${this.clienteEditando ? 'atualizar' : 'salvar'} cliente: ` + 
                                    (errorData.error || errorData.message || 'Erro desconhecido');
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
      this.showErrorAlert = false;
      this.errorMessage = "";
      this.cliente = {
        nameCustomer: "",
        emailCustomer: "",
        phoneCustomer: "",
        addressCustomer: ""
      };
      this.$emit('close');
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

.modal-content .form-control {
  color: #000 !important;
  background-color: #fff;
}

.success-alert {
  position: fixed !important;
  top: 20px !important;
  left: 50% !important;
  transform: translateX(-50%) !important;
  z-index: 1060 !important;
  min-width: 400px !important;
  max-width: 500px !important;
  text-align: center !important;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3) !important;
  border: none !important;
}

.modal-alert {
  position: fixed !important;
  top: 50% !important;
  left: 50% !important;
  transform: translate(-50%, -50%) !important;
  z-index: 1060 !important;
  min-width: 400px !important;
  max-width: 500px !important;
  text-align: center !important;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3) !important;
  border: none !important;
}

.success-alert .alert,
.modal-alert .alert {
  margin-bottom: 0 !important;
  border-radius: 10px !important;
}

.loading-overlay {
  z-index: 1070 !important;
}
</style>