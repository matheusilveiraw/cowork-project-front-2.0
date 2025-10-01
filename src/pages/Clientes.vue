<template>
    <div class="content">
        <div class="card">
            <div class="card-header">
                <h4 class="title">Clientes</h4>
            </div>

            <div class="menu-rentals">
                <div class="d-flex mb-3 ml-3">
                    <button class="btn btn-primary mr-2" @click="abrirModalCadastro">
                        Novo cliente
                    </button>
                </div>
            </div>

            <div class="card-body">
                <div v-if="loading" class="text-center">
                    <p>Carregando clientes...</p>
                </div>

                <div v-else-if="clientes.length === 0" class="text-center">
                    <p>Nenhum cliente cadastrado. Clique em "Novo cliente" para começar.</p>
                </div>

                <div v-else class="table-responsive">
                    <table class="table table-striped table-hover">
                        <thead class="thead-dark">
                            <tr>
                                <th>ID</th>
                                <th>Nome</th>
                                <th>Email</th>
                                <th>Telefone</th>
                                <th>Endereço</th>
                                <th>Data de Cadastro</th>
                                <th>Ações</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="cliente in clientes" :key="cliente.idCustomer">
                                <td>{{ cliente.idCustomer }}</td>
                                <td>
                                    <strong>{{ cliente.nameCustomer }}</strong>
                                </td>
                                <td>{{ cliente.emailCustomer }}</td>
                                <td>{{ cliente.phoneCustomer || '-' }}</td>
                                <td>{{ cliente.addressCustomer || '-' }}</td>
                                <td>{{ formatarData(cliente.created_at) }}</td>
                                <td>
                                    <div class="btn-group-actions">
                                        <button class="btn btn-sm btn-outline-primary"
                                            @click="abrirModalEdicao(cliente)" title="Editar cliente">
                                            <i class="tim-icons icon-pencil"></i>
                                        </button>
                                        <button class="btn btn-sm btn-outline-danger"
                                            @click="confirmarExclusao(cliente)" title="Excluir cliente">
                                            <i class="tim-icons icon-simple-remove"></i>
                                        </button>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- Modal placeholder para futuro uso -->
        <!-- <ModalCadastroClientes 
      :show="abrirModal" 
      :cliente-editando="clienteEditando" 
      @close="fecharModal"
      @cliente-salvo="buscarClientes" 
    /> -->
    </div>
</template>

<script>
import axios from "axios";
// import ModalCadastroClientes from "@/components/ModalCadastroClientes.vue"; // Para uso futuro
// import Swal from 'sweetalert2'; // Para uso futuro

export default {
    name: "Clientes",
    // components: {
    //   ModalCadastroClientes // Para uso futuro
    // },
    data() {
        return {
            abrirModal: false,
            clienteEditando: null,
            clientes: [],
            loading: false
        };
    },
    methods: {
        async buscarClientes() {
            this.loading = true;
            try {
                const response = await axios.get(
                    "http://localhost/projetos/cowork-project-back/public/customers"
                );

                this.clientes = response.data.data || response.data || [];

            } catch (error) {
                console.error('Erro ao buscar clientes:', error);
                this.mostrarErro('Erro ao carregar clientes: ' + (error.response?.data?.message || error.message));
            } finally {
                this.loading = false;
            }
        },

        abrirModalCadastro() {
            console.log('Abrir modal de cadastro - funcionalidade futura');
            // this.clienteEditando = null;
            // this.abrirModal = true;
        },

        abrirModalEdicao(cliente) {
            console.log('Abrir modal de edição - funcionalidade futura', cliente);
            // this.clienteEditando = cliente;
            // this.abrirModal = true;
        },

        confirmarExclusao(cliente) {
            console.log('Confirmar exclusão - funcionalidade futura', cliente);
        },

        formatarData(data) {
            if (!data) return '-';
            return new Date(data).toLocaleDateString('pt-BR');
        },

        mostrarErro(mensagem) {
            console.error('Erro:', mensagem);
        }
    },
    mounted() {
        this.buscarClientes();
    }
};
</script>

<style scoped>
.btn-group-actions {
    display: flex;
    gap: 8px;
}

@media (max-width: 768px) {
    .btn-group-actions {
        gap: 12px;
    }
}

.table-responsive {
    border-radius: 6px;
    overflow: hidden;
}

.table th {
    background-color: #343a40;
    color: white;
    border: none;
    padding: 12px 8px;
    font-weight: 600;
}

.table td {
    padding: 10px 8px;
    vertical-align: middle;
}

.table-striped tbody tr:nth-of-type(odd) {
    background-color: rgba(0, 0, 0, 0.02);
}

.table-hover tbody tr:hover {
    background-color: rgba(0, 123, 255, 0.075);
}

.btn-sm {
    padding: 0.25rem 0.5rem;
    font-size: 0.75rem;
    margin: 0 2px;
}

@media (max-width: 768px) {
    .table-responsive {
        font-size: 0.875rem;
    }

    .table th,
    .table td {
        padding: 8px 6px;
    }
}
</style>