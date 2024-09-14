<template>
    <v-row no-gutters>
        <v-col cols="12" md="2">
            <v-list class="categoryWrap">
                <v-list-item title="전체" :value='"all"' v-model="selectedCategoryItem" @update:model-value="setCat" @click="setCat()">
                    <template v-slot:prepend>
                        <div :style="(catFilter == null || selectedCategoryItem == 'all') ? StyleMenuListActive : styleMenuList"></div>
                    </template>
                </v-list-item>

                <template v-for="cat in cat1" :key='cat'>
                    <v-list-item prepend-icon="mdi-flower" :title="cat" :value='cat' v-model="selectedCategoryItem"
                        @update:model-value="setCat"
                        @click="setCat(cat)"
                    >
                        <template v-slot:prepend>
                            <div :style="selectedCategoryItem==cat ? StyleMenuListActive : styleMenuList"></div>
                        </template>
                    </v-list-item>
                </template>
            </v-list>
        </v-col>
        <v-col cols="12" md="10" class="pl-3">
            <v-row no-gutters justify="center">

                <!-- [주의] 안내문 -->
                <v-col cols="12" sm="12" class="text-center">
                    <h3 style="color: red;">프사용도 외 사용불가, 상업적 이용, 2차가공 및 인쇄 금지</h3>
                </v-col>

                <!-- 검색창 -->
                <v-col style="max-width: 240px;">
                    <v-sheet class="ma-2 pa-2 elevation-0" style="background-color: transparent;">
                        <div :style="styleSearchWrap">
                            <input type="text" v-model='search' align-self="center" hide-details="auto" label="검색"
                                :style="styleSearchInput">
                            <v-icon color="gray">
                                mdi-magnify
                            </v-icon>
                        </div>
                    </v-sheet>
                </v-col>
            </v-row>
            <v-row no-gutters>
                <!-- 제2 카테고리 -->
                <div class="chip-container mb-4">
                    <v-chip-group selected-class="text-default" variant="text" mandatory>
                        <v-chip class='cat2Chip' :class="(selectedCategory2Item==''|| selectedCategory2Item==null)?'active':''" text='전체' @click="setCat2()"></v-chip>
                        <v-chip class='cat2Chip' :class="selectedCategory2Item==cat2?'active':''" v-for='cat2 in cat2List' :key='cat2' :text='cat2' @click="setCat2(cat2)" ></v-chip>
                    </v-chip-group>
                </div>

                <!-- 이미지 리스트 -->
                <v-col cols="12" sm="12" class="d-flex align-content-center flex-wrap ga-2">
                    <v-img v-for='img in filteredImages' :key="img?.id" :width="120" max-width="120" min-width="120"
                        aspect-ratio="1/1" cover class="elevation-3" :src="img?.file"></v-img>
                </v-col>
            </v-row>
        </v-col>
    </v-row>
</template>

<script setup>
import { onMounted, ref, computed } from "vue";
import menu_list_icon from '@/assets/img/menu_list_icon.jpg';

// css
const styleMenuList = {
    backgroundImage: `url(${menu_list_icon})`,
    width: '30px',
    height: '30px',
    backgroundSize: '30px 60px',
    backgroundPosition: '0 -30px',
    marginRight: '0.5rem',
}
const StyleMenuListActive = {
    ...styleMenuList,
    backgroundPosition: '0 0',
}
const styleSearchWrap = {
    display: 'flex',
    alignItems: 'center',
    maxWidth: '240px',
    padding: '0.25rem 0.5rem',
    borderRadius: '30px',
    boxShadow: 'rgb(215 215 215) 0px 2px 4px 4px',
    paddingLeft: '1rem',
    backgroundColor: 'white',
}
const styleSearchInput = {
    width: '100%',
    height: '30px',
    outline: 'none',
}



const category = ref([]);
const cat2List = ref([]);
const cat2ListOrg = ref([]);
const search = ref(null);
const catFilter = ref(null);
const images = ref([]);
const selectedCategoryItem = ref(null);
const selectedCategory2Item = ref(null);

const filteredImages = computed(() => {
    let filtered = images.value;
    if (catFilter.value) {
        filtered = filtered.filter(img => {
            return img.category_1.indexOf(catFilter.value) != -1 || img.category_2.indexOf(catFilter.value) != -1
        })
    }
    if (search.value) {
        filtered = filtered.filter(img => {
            const fileLower = img.file.toLowerCase();
            const searchLower = search.value.toLowerCase();
            return img.name.indexOf(search.value) != -1 || img.tag.indexOf(search.value) != -1 || img.category_1.indexOf(search.value) != -1 || img.category_2.indexOf(search.value) != -1 || fileLower.indexOf(searchLower) != -1;
        })
    }
    return filtered;
})
const cat1 = computed(() => {
    return Object.keys(category.value);
})

const setCatFilter = (filter) => {
    catFilter.value = filter;
}
const setCat = (value) => {
    selectedCategoryItem.value = value;
    _generateCat2(value);
    setCatFilter(value)
}
const setCat2 = (value) =>{
    selectedCategory2Item.value = value;
    selectedCategoryItem.value = 'all';
    setCatFilter(value);
}
const _generateCat2 = (value)=>{
    if(value){
        cat2List.value = category.value[value];
    }else{
        cat2List.value = cat2ListOrg.value;
    }
}

onMounted(async () => {
    const response = await fetch("/assets/data/data.json");
    const file = await response.json();
    // console.info('file', file)
    // 데이터
    images.value = file.filter(file => {
        return file.file
    });
    // 카테고리
    let catSet = [];
    let lCat2List = [];
    file.forEach(element => {
        if (!element.file) return;
        if (!catSet[element.category_1]) catSet[element.category_1] = []
        if (catSet[element.category_1].indexOf(element.category_2) == -1) {
            catSet[element.category_1].push(element.category_2)
        }
        element.category_2 && lCat2List.indexOf(element.category_2)==-1 && lCat2List.push(element.category_2);
    });

    // console.info('catSet', catSet)
    // console.info('lCat2List', lCat2List)
    category.value = catSet;
    cat2List.value = lCat2List;
    cat2ListOrg.value = lCat2List;
});



</script>


<style scoped>
.chip-container {
    display: flex;
    flex-wrap: nowrap;
    /* 가로로 쌓이도록 설정 */
    overflow-x: auto;
    /* 가로 스크롤 활성화 */
    width: 100%;
    /* 컨테이너 너비 */
}
.categoryWrap{
    border-radius: 20px;
    margin-bottom: 1rem;
}
.cat2Chip{
    background-color: white;
    color: black;
}

.cat2Chip.active{
    background-color: black;
    color: white;
}
</style>