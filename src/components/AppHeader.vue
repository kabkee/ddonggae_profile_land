
<template>
  
    <v-row no-gutters>
      <v-col
        cols="12"
      >
        <v-sheet class="ma-2 pa-2 text-center position-relative" min-height="" style="align-content: center;">
            <v-img
            max-width="100%"
            aspect-ratio="16/9"
            cover
            :src="top_title"
            ></v-img>
            <a href="https://linktr.ee/ttong._.gae" target="_blank" rel="noopener noreferrer">
                <v-img
                max-width="200"
                cover
                :style="styleDdonggaeLink"
                :src="ddanggae_link"
                ></v-img>
            </a>
        </v-sheet>
      </v-col>
    </v-row>
</template>

<script setup>
import { onMounted, ref } from "vue";
import top_title from '@/assets/img/top_title.jpg';
import ddanggae_link from '@/assets/img/ddanggae_link.jpg';

const styleDdonggaeLink = {
    position: 'absolute',
    width: '20%',
    maxWidth: '200px',
    height: 'auto',
    left: '50%',
    bottom: '5%',
    transform: 'translateX(-50%)'

}

const appTitle = ref(null)

onMounted(async () => {
    const response = await fetch("/assets/data/config.json");
    const file = await response.json();

    // 데이터
    appTitle.value = file.name;
    document.title = file.name;
});
</script>
