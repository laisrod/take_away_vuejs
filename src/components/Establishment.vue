<template>
  <div class="establishment">
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
          <div class="items">
            <h4>Itens do Pedido:</h4>
            <ul>
              <li v-for="item in pedido.items" :key="item.id">
                {{ item.quantity }}x {{ item.name }} - R$ {{ item.price }}
              </li>
            </ul>
          </div>
          <button
            @click.stop="aceitarPedido(pedido), selecionarPedido(null)"
            :disabled="pedido.status !== 'pending'"
            class="aceitar-btn"
          >
            Aceitar Pedido
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
          <div class="items">
            <h4>Itens do Pedido:</h4>
            <ul>
              <li v-for="item in pedido.items" :key="item.id">
                {{ item.quantity }}x {{ item.name }} - R$ {{ item.price }}
              </li>
            </ul>
          </div>
          <button
            @click.stop="aceitarPedido(pedido)"
            :disabled="pedido.status !== 'preparing'"
            class="aceitar-btn"
          >
            Marcar como Pronto
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
    };
  },
  async created() {
    try {
      const response = await fetch(
        "http://localhost:3000/api/v1/establishments/6066d0a2b780/orders?status[]=pending&status[]=preparing",
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
          `http://localhost:3000/api/v1/establishments/6066d0a2b780/orders/${pedido.code}/ready_order`,
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

        // Atualiza o status do pedido para 'ready' após resposta bem-sucedida
        const index = this.pedidos.findIndex((p) => p.id === pedido.id);
        if (index !== -1) {
          this.pedidos[index].status = 'ready';
          // Opcional: fechar os detalhes do pedido após marcar como pronto
          this.pedidoSelecionado = null;
        }
      } catch (error) {
        console.error("Erro ao aceitar o pedido:", error);
        alert("Não foi possível aceitar o pedido. Tente novamente.");
      }
    },
  },
  async marcarPedidoComoPronto(pedido) {
  try {
    const response = await fetch(
      `http://localhost:3000/api/v1/establishments/6066d0a2b780/orders/${pedido.code}/ready_order`,
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
      return this.pedidosOrdenados.filter(pedido => pedido.status === 'pending');
    },
    pedidosEmPreparo() {
      return this.pedidosOrdenados.filter(pedido => pedido.status === 'preparing');
    }
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
</style>
