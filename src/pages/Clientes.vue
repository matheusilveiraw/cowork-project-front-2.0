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

                <div v-else>
                    <!-- Tabela -->
                    <div class="table-responsive">
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
                                <tr v-for="cliente in clientesPagina" :key="cliente.idCustomer">
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

                    <!-- Paginação Simples -->
                    <div class="d-flex justify-content-between align-items-center mt-3">
                        <small class="text-muted">
                            Mostrando {{ inicio + 1 }}-{{ fim }} de {{ clientes.length }} clientes
                        </small>
                        
                        <div class="pagination-simple">
                            <button 
                                class="btn btn-sm btn-outline-secondary me-2" 
                                @click="paginaAnterior"
                                :disabled="paginaAtual === 1"
                            >
                                ‹ Anterior
                            </button>
                            
                            <span class="mx-2">Página {{ paginaAtual }} de {{ totalPaginas }}</span>
                            
                            <button 
                                class="btn btn-sm btn-outline-secondary ms-2" 
                                @click="proximaPagina"
                                :disabled="paginaAtual === totalPaginas"
                            >
                                Próxima ›
                            </button>
                        </div>
                    </div>
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
import Swal from 'sweetalert2';

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
            loading: false,
            paginaAtual: 1,
            clientesPorPagina: 10
        };
    },
    computed: {
        // Clientes da página atual
        clientesPagina() {
            const inicio = (this.paginaAtual - 1) * this.clientesPorPagina;
            const fim = inicio + this.clientesPorPagina;
            return this.clientes.slice(inicio, fim);
        },
        
        // Total de páginas
        totalPaginas() {
            return Math.ceil(this.clientes.length / this.clientesPorPagina);
        },
        
        // Índice do primeiro cliente da página
        inicio() {
            return (this.paginaAtual - 1) * this.clientesPorPagina;
        },
        
        // Índice do último cliente da página
        fim() {
            const fimCalculado = this.inicio + this.clientesPorPagina;
            return fimCalculado > this.clientes.length ? this.clientes.length : fimCalculado;
        }
    },
    methods: {
        async buscarClientes() {
            this.loading = true;
            try {
                const response = await axios.get(
                    "http://localhost/projetos/cowork-project-back/public/customers"
                );

                this.clientes = response.data.data || response.data || [];
                // Reseta para a primeira página após carregar
                this.paginaAtual = 1;

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

        async confirmarExclusao(cliente) {
            const result = await Swal.fire({
                title: 'Confirmar Exclusão',
                html: `Tem certeza que deseja excluir o cliente <strong>"${cliente.nameCustomer}"</strong>?`,
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#d33',
                cancelButtonColor: '#3085d6',
                confirmButtonText: 'Sim, excluir!',
                cancelButtonText: 'Cancelar',
                reverseButtons: true
            });

            if (result.isConfirmed) {
                await this.excluirCliente(cliente);
            }
        },

        async excluirCliente(cliente) {
            try {
                await axios.delete(
                    `http://localhost/projetos/cowork-project-back/public/customers/${cliente.idCustomer}`
                );

                Swal.fire({
                    title: 'Excluído!',
                    text: 'Cliente excluído com sucesso.',
                    icon: 'success',
                    timer: 2000,
                    showConfirmButton: false
                });

                this.buscarClientes();

            } catch (error) {
                console.error('Erro ao excluir cliente:', error);
                
                let errorMessage = 'Erro ao excluir cliente: ' + (error.response?.data?.message || error.message);
                
                if (error.response?.status === 409) {
                    errorMessage = error.response.data.error;
                }

                Swal.fire({
                    title: 'Erro!',
                    html: errorMessage,
                    icon: 'error',
                    confirmButtonText: 'Entendi'
                });
            }
        },

        // Métodos de paginação
        proximaPagina() {
            if (this.paginaAtual < this.totalPaginas) {
                this.paginaAtual++;
            }
        },

        paginaAnterior() {
            if (this.paginaAtual > 1) {
                this.paginaAtual--;
            }
        },

        formatarData(data) {
            if (!data) return '-';
            return new Date(data).toLocaleDateString('pt-BR');
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

/* Estilos da paginação */
.pagination-simple {
    display: flex;
    align-items: center;
}

.pagination-simple .btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}

@media (max-width: 768px) {
    .table-responsive {
        font-size: 0.875rem;
    }

    .table th,
    .table td {
        padding: 8px 6px;
    }
    
    .d-flex.justify-content-between {
        flex-direction: column;
        gap: 10px;
        text-align: center;
    }
    
    .pagination-simple {
        justify-content: center;
    }
}
</style>