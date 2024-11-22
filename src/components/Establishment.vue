<template>
  <div class="establishment">
    <!-- Novo input para establishmentCode -->
    <div class="establishment-input">
      <label for="establishmentCode">Código do Estabelecimento:</label>
      <input
        type="text"
        id="establishmentCode"
        v-model="establishmentCode"
        @change="buscarPedidos"
        placeholder="Digite o código do estabelecimento"
      />
      <button @click="buscarPedidos" class="buscar-btn">Buscar Pedidos</button>
    </div>
    <h2>Pedidos do Estabelecimento</h2>
    <div v-if="loading">Carregando...</div>
    <div v-else-if="pedidos.length === 0">
      <p>Nenhum pedido encontrado</p>
    </div>
    <div v-else>
      <!-- Pedidos Pendentes -->
      <h3>Pedidos Pendentes</h3>
      <div v-if="pedidosPendentes.length === 0">
        <p>Nenhum pedido pendente</p>
      </div>
      <div
        v-for="pedido in pedidosPendentes"
        :key="pedido.id"
        class="pedido pendente"
        @click="selecionarPedido(pedido)"
      >
        <p>Pedido #{{ pedido.id }}</p>

        <div
          v-if="pedidoSelecionado && pedidoSelecionado.id === pedido.id"
          class="pedido-detalhes"
        >
          <p><strong>Status:</strong> {{ pedido.status }}</p>
          <p><strong>Cliente:</strong> {{ pedido.customer_name }}</p>
          <p><strong>Total:</strong> R$ {{ pedido.total_price }}</p>
          <button
            @click.stop="aceitarPedido(pedido)"
            :disabled="pedido.status !== 'pending'"
            class="aceitar-btn"
          >
            Aceitar Pedido
          </button>
          <button
            v-if="pedido.status === 'pending'"
            @click.stop="mostrarCancelamento(pedido)"
            class="cancelar-btn"
          >
            Cancelar Pedido
          </button>
        </div>
      </div>

      <!-- Pedidos em Preparo -->
      <h3>Pedidos em Preparo</h3>
      <div v-if="pedidosEmPreparo.length === 0">
        <p>Nenhum pedido em preparo</p>
      </div>
      <div
        v-for="pedido in pedidosEmPreparo"
        :key="pedido.id"
        class="pedido preparo"
        @click="selecionarPedido(pedido)"
      >
        <p>Pedido #{{ pedido.id }}</p>

        <div
          v-if="pedidoSelecionado && pedidoSelecionado.id === pedido.id"
          class="pedido-detalhes"
        >
          <p><strong>Status:</strong> {{ pedido.status }}</p>
          <p><strong>Cliente:</strong> {{ pedido.customer_name }}</p>
          <p><strong>Total:</strong> R$ {{ pedido.total_price }}</p>
          <button
            @click.stop="aceitarPedido(pedido)"
            :disabled="pedido.status !== 'preparing'"
            class="aceitar-btn"
          >
            Marcar como Pronto
          </button>
          <button
            v-if="pedido.status === 'preparing'"
            @click.stop="mostrarCancelamento(pedido)"
            class="cancelar-btn"
          >
            Cancelar Pedido
          </button>
        </div>
      </div>
    </div>

    <!-- Adicionar modal de cancelamento -->
    <div v-if="mostrarModalCancelamento" class="modal">
      <div class="modal-content">
        <h3>Cancelar Pedido</h3>
        <textarea
          v-model="motivoCancelamento"
          placeholder="Digite o motivo do cancelamento"
          rows="4"
        ></textarea>
        <div class="modal-buttons">
          <button @click="confirmarCancelamento" class="confirmar-btn">
            Confirmar
          </button>
          <button @click="fecharModalCancelamento" class="cancelar-btn">
            Voltar
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Establishment",
  data() {
    return {
      pedidos: [],
      loading: true,
      error: null,
      pedidoSelecionado: null,
      establishmentCode: "",
      mostrarModalCancelamento: false,
      motivoCancelamento: "",
      pedidoParaCancelar: null,
    };
  },
  methods: {
    selecionarPedido(pedido) {
      // Se clicar no mesmo pedido, fecha os detalhes
      if (this.pedidoSelecionado && this.pedidoSelecionado.id === pedido.id) {
        this.pedidoSelecionado = null;
      } else {
        this.pedidoSelecionado = pedido;
      }
    },
    async aceitarPedido(pedido) {
      try {
        const response = await fetch(
          `http://localhost:3000/api/v1/establishments/${this.establishmentCode}/orders/${pedido.code}/ready_order`,
          {
            method: "PATCH",
            headers: {
              "Content-Type": "application/json",
            },
          }
        );

        if (!response.ok) {
          throw new Error("Erro ao aceitar o pedido");
        }

        const index = this.pedidos.findIndex((p) => p.id === pedido.id);
        if (index !== -1) {
          this.pedidos[index].status = "ready";
          this.pedidoSelecionado = null;
        }
      } catch (error) {
        console.error("Erro ao aceitar o pedido:", error);
        alert("Não foi possível aceitar o pedido. Tente novamente.");
      }
    },
    async buscarPedidos() {
      if (!this.establishmentCode) return;

      this.loading = true;
      try {
        const response = await fetch(
          `http://localhost:3000/api/v1/establishments/${this.establishmentCode}/orders?status[]=pending&status[]=preparing`,
          {
            method: "GET",
            headers: {
              "Content-Type": "application/json",
            },
          }
        );

        if (!response.ok) {
          throw new Error("Erro ao buscar pedidos");
        }

        this.pedidos = await response.json();
      } catch (error) {
        this.error = error.message;
        console.error("Erro:", error);
      } finally {
        this.loading = false;
      }
    },
    async cancelarPedido(pedido) {
      try {
        const response = await fetch(
          `http://localhost:3000/api/v1/establishments/${this.establishmentCode}/orders/${pedido.code}/cancelled`,
          {
            method: "PATCH",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              cancellation_reason: this.motivoCancelamento,
            }),
          }
        );

        if (!response.ok) {
          throw new Error("Erro ao cancelar o pedido");
        }

        this.pedidos = this.pedidos.filter((p) => p.id !== pedido.id);
        this.pedidoSelecionado = null;
      } catch (error) {
        console.error("Erro ao cancelar o pedido:", error);
        alert("Não foi possível cancelar o pedido. Tente novamente.");
      }
    },
    mostrarCancelamento(pedido) {
      this.pedidoParaCancelar = pedido;
      this.mostrarModalCancelamento = true;
      this.motivoCancelamento = "";
    },

    fecharModalCancelamento() {
      this.mostrarModalCancelamento = false;
      this.pedidoParaCancelar = null;
      this.motivoCancelamento = "";
    },

    async confirmarCancelamento() {
      if (!this.motivoCancelamento.trim()) {
        alert("Por favor, informe o motivo do cancelamento");
        return;
      }

      try {
        const response = await fetch(
          `http://localhost:3000/api/v1/establishments/${this.establishmentCode}/orders/${this.pedidoParaCancelar.code}/cancelled`,
          {
            method: "PATCH",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              cancellation_reason: this.motivoCancelamento,
            }),
          }
        );

        if (!response.ok) {
          throw new Error("Erro ao cancelar o pedido");
        }

        this.pedidos = this.pedidos.filter(
          (p) => p.id !== this.pedidoParaCancelar.id
        );
        this.pedidoSelecionado = null;
        this.fecharModalCancelamento();
      } catch (error) {
        console.error("Erro ao cancelar o pedido:", error);
        alert("Não foi possível cancelar o pedido. Tente novamente.");
      }
    },
  },
  async marcarPedidoComoPronto(establishmentCode, pedido) {
    try {
      const response = await fetch(
        `http://localhost:3000/api/v1/establishments/${establishmentCode}/orders/${pedido.code}/order_ready`,
        {
          method: "PATCH",
        }
      );
      if (!response.ok) {
        throw new Error("Erro ao marcar o pedido como pronto");
      }
    } catch (error) {
      console.error("Erro ao marcar o pedido como pronto:", error);
      alert("Não foi possível marcar o pedido como pronto. Tente novamente.");
    }
  },
  computed: {
    pedidosOrdenados() {
      return [...this.pedidos].sort((a, b) => {
        return new Date(a.created_at) - new Date(b.created_at);
      });
    },
    pedidosPendentes() {
      return this.pedidosOrdenados.filter(
        (pedido) => pedido.status === "pending"
      );
    },
    pedidosEmPreparo() {
      return this.pedidosOrdenados.filter(
        (pedido) => pedido.status === "preparing"
      );
    },
  },
};
</script>

<style scoped>
.establishment {
  padding: 20px;
}

.pedido {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 4px;
}

.aceitar-btn {
  background-color: #4caf50;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.aceitar-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.pendente {
  border-left: 4px solid #ffa500;
}

.preparo {
  border-left: 4px solid #4caf50;
}

.cancelar-btn {
  background-color: #f44336;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
  margin-left: 10px;
}

.cancelar-btn:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 90%;
  max-width: 500px;
}

.modal-content textarea {
  width: 100%;
  margin: 10px 0;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.modal-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 15px;
}

.confirmar-btn {
  background-color: #4caf50;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
