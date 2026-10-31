<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          <h2>PERKIRAAN CUACA KOTA JAKARTA</h2>
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Cuaca Jakarta</ion-title>
        </ion-toolbar>
      </ion-header>

      <!-- CARD -->
      <h2>Perkiraaan Suhu Per Jam</h2>
      <p>Informasi suhu udara Kota Jakarta</p>

      <!-- loading -->
      <div class="ion-text-center" v-if="loading">
        <ion-spinner name="crescent"></ion-spinner>
        <p>Sedang mengambil data cuaca...</p>
      </div>

      <!-- error -->
      <ion-card v-if="error">
        <ion-card-content>
          {{ error }}
        </ion-card-content>
      </ion-card>

      <!-- data suhu dari API -->
      <ion-card v-for="item in suhu.slice(0, 24)" :key="item.waktu">
        <!-- judul -->
        <ion-card-header>
          <ion-card-subtitle>
            {{ item.waktu }}
          </ion-card-subtitle>

          <ion-card-title> {{ item.temperatur }} °C </ion-card-title>
        </ion-card-header>

        <!-- pesan -->
        <ion-card-content>
          Suhu udara yang diprakirakan pada waktu tersebut.
        </ion-card-content>
      </ion-card>

      <ion-button expand="block" @click="ambilCuaca" :disabled="loading">
        Muat Ulang
      </ion-button>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonSpinner,
  IonButton,
} from "@ionic/vue";
import ExploreContainer from "@/components/ExploreContainer.vue";

import { Ref, onMounted, ref } from "vue";
// type data suhu
type dataSuhu = {
  waktu: string;
  temperatur: number | null;
};

// variabel reaktif
const suhu = ref<dataSuhu[]>([]);
const loading = ref(false);
const error = ref("");

// ambil data api
async function ambilCuaca() {
  loading.value = true;
  error.value = "";

  try {
    const response = await fetch(
      "https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m&timezone=Asia%2FJakarta",
    );

    if (!response.ok) {
      throw new Error(`Gagal mengambil data: ${response.status}`);
    }

    const data = await response.json();

    console.log("Respons API:", data);

    if (
      !data.hourly ||
      !Array.isArray(data.hourly.time) ||
      !Array.isArray(data.hourly.temperature_2m)
    ) {
      throw new Error("Data suhu tidak ditemukan dalam respons API");
    }

    suhu.value = data.hourly.time.map((waktu: string, index: number) => ({
      waktu,
      temperatur: data.hourly.temperature_2m[index] ?? null,
    }));
  } catch (err: unknown) {
    error.value = err instanceof Error ? err.message : "Terjadi kesalahan";
    console.error("Kesalahan API:", err);
  } finally {
    loading.value = false;
    console.log("Loading selesai:", loading.value);
  }
}
</script>
