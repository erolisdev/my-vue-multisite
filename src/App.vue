<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import PocketBase from "pocketbase";

const domain = window.location.hostname;

let apiBase = "";
switch (domain) {
  case "irfanvue.eronor.de":
    apiBase = "https://irfan.eronor.de";
    break;
  case "thomas.eronor.de":
    apiBase = "https://thomasback.eronor.de";
    break;
  case "ozkan.eronor.de":
    apiBase = "https://tatar66.eronor.de";
    break;
  default:
    apiBase = "https://irfan.eronor.de";
}

const pb = new PocketBase(apiBase);

const orders = ref([]);
const selectedOrder = ref(null);
const error = ref(null);
const loading = ref(false);

async function loadDefaultOrders() {
  loading.value = true;
  error.value = null;
  try {
    //  orders.value = await pb.collection("order_infos").getFullList({
    //    sort: "-created",
    //    expand: "customer,address,payments_via_order,order_items_via_order",
    //  });

    const res = await pb.collection("order_infos").getList(1, 100, {
      sort: "-created",
      expand: "customer,address,payments_via_order,order_items_via_order",
    });
    orders.value = Array.isArray(res.items) ? res.items : res;
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
}

async function loadFromCustom(endpoint) {
  loading.value = true;
  error.value = null;
  try {
    //  const res = await fetch(`${apiBase}${endpoint}`);

    const res = await pb.send(endpoint);
    const data = res;
    orders.value = Array.isArray(data.items) ? data.items : data;
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
}

function formatDate(dateStr) {
  const d = new Date(dateStr);
  return d.toLocaleString();
}

function showDetails(order) {
  selectedOrder.value = order;
}

onMounted(() => {
  loadDefaultOrders();
  pb.collection("order_infos").subscribe("*", loadDefaultOrders);
});

onBeforeUnmount(() => {
  pb.collection("order_infos").unsubscribe("*");
});
</script>

<template>
  <div class="container">
    <h1>Sipariş Listesi - {{ domain }}</h1>
    <p>
      Backend: <strong>{{ apiBase }}</strong>
    </p>

    <div class="button-group">
      <button @click="loadDefaultOrders">Ana Siparişleri Yükle</button>
      <button @click="loadFromCustom('/orders1')">/orders1 Siparişleri</button>
      <button @click="loadFromCustom('/orders2')">/orders2 Siparişleri</button>
    </div>

    <p v-if="loading">Yükleniyor...</p>
    <p v-if="error" class="error">Hata: {{ error }}</p>

    <table v-if="orders.length" class="order-table">
      <thead>
        <tr>
          <th>Müşteri</th>
          <th>Email</th>
          <th>Toplam</th>
          <th>Tarih</th>
          <th>Detaylar</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="order in orders" :key="order.id">
          <td>
            {{ order.expand?.customer?.name || order.customer?.name || "-" }}
          </td>
          <td>
            {{ order.expand?.customer?.email || order.customer?.email || "-" }}
          </td>
          <td>{{ order.orderTotal }} ₺</td>
          <td>{{ formatDate(order.created) }}</td>
          <td><button @click="showDetails(order)">Görüntüle</button></td>
        </tr>
      </tbody>
    </table>

    <!-- Modal -->
    <div
      v-if="selectedOrder"
      class="modal-overlay"
      @click.self="selectedOrder = null"
    >
      <div class="modal">
        <h2>🧾 Sipariş Detayı</h2>
        <p>
          <strong>Müşteri:</strong>
          {{
            selectedOrder.expand?.customer?.name || selectedOrder.customer?.name
          }}
        </p>
        <p>
          <strong>Email:</strong>
          {{
            selectedOrder.expand?.customer?.email ||
            selectedOrder.customer?.email
          }}
        </p>
        <p>
          <strong>Adres:</strong>
          {{
            selectedOrder.expand?.address?.city ||
            selectedOrder.shipping_address?.city
          }}
        </p>
        <p><strong>Not:</strong> {{ selectedOrder.orderNote || "-" }}</p>
        <p><strong>Tutar:</strong> {{ selectedOrder.orderTotal }} ₺</p>

        <h3>🛒 Ürünler</h3>
        <ul>
          <li
            v-for="item in selectedOrder.expand?.order_items_via_order ||
            selectedOrder.order_items"
            :key="item.id"
          >
            {{ item.productName }} - {{ item.quantity }} adet x
            {{ item.unitPrice }} = {{ item.totalPrice }} ₺
          </li>
        </ul>

        <h3>💳 Ödeme</h3>
        <p>
          {{
            (
              selectedOrder.expand?.payments_via_order?.[0] ||
              selectedOrder.payments?.[0]
            )?.method
          }}
          -
          {{
            (
              selectedOrder.expand?.payments_via_order?.[0] ||
              selectedOrder.payments?.[0]
            )?.status
          }}
        </p>
        <button class="close" @click="selectedOrder = null">Kapat</button>
      </div>
    </div>
  </div>
</template>

<style>
.container {
  font-family: sans-serif;
  max-width: 900px;
  margin: 2rem auto;
  padding: 1rem;
}

.button-group {
  display: flex;
  gap: 1rem;
  margin-bottom: 1rem;
}

.button-group button {
  padding: 8px 12px;
  font-size: 14px;
  cursor: pointer;
}

.order-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.order-table th,
.order-table td {
  border: 1px solid #ccc;
  padding: 8px 12px;
  text-align: left;
}

.order-table th {
  background-color: #f5f5f5;
}

.error {
  color: red;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  background: white;
  padding: 2rem;
  max-width: 500px;
  border-radius: 8px;
}

.modal h3 {
  margin-top: 1rem;
}

.modal .close {
  margin-top: 1rem;
}
</style>
