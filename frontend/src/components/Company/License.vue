<template>
    <div class="container1">
        <h2 class="title1">특허 목록</h2>
        <h5 class="subtitle1" v-if="patents.length === 0">{{ patents.message }}</h5>
        <h5 class="subtitle1" v-else>특허 개수: {{ patents.length }}</h5>
        <ul class="patent-list">
            <li
                class="patent-item"
                v-for="(patent, index) in visiblePatents"
                :key="patent.patentId">
                <div class="patent-info">{{ patent.patentName }} - {{ patent.patentDate }}</div>
                <p class="patent-summary">{{ patent.patentSummary }}</p>
            </li>
        </ul>
        <button class="show-more" v-if="!isExpanded" @click="showMore">더 보기</button>
        <button class="show-more" v-if="isExpanded" @click="foldList">접기</button>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { useCounterStore } from '@/stores/counter';
import { useRoute } from 'vue-router';

const patents = ref([]);
const visiblePatents = ref([]);
const batchSize = 3; // 한 번에 보여줄 특허 수
const isExpanded = ref(false); // 리스트가 확장되었는지 여부를 추적
const store = useCounterStore();
const API_URL = store.API_URL;
const route = useRoute();

const companyId = ref(route.params.companyId);

onMounted(() => {
    // console.log('특허 요청');
    getLicenseData();
});

const getLicenseData = function () {
    axios({
        method: 'get',
        url: `${API_URL}/api/patent/${companyId.value}`,
    })
        .then((res) => {
            patents.value = res.data.data;
            visiblePatents.value = patents.value.slice(0, batchSize);
            console.log(patents.value);
            // console.log('데이터 받음');
        })
        .catch((err) => {
            console.log(err);
            console.log('실패');
        });
};

const showMore = () => {
    if (visiblePatents.value.length < patents.value.length) {
        const startIndex = visiblePatents.value.length;
        const endIndex = patents.value.length;
        visiblePatents.value = [
            ...visiblePatents.value,
            ...patents.value.slice(startIndex, endIndex),
        ];
        isExpanded.value = true; // 리스트가 확장된 상태로 변경
    }
};

const foldList = () => {
    visiblePatents.value = patents.value.slice(0, batchSize); // 처음 3개만 보여주는 상태로 복귀
    isExpanded.value = false; // 리스트가 접힌 상태로 변경
};
</script>

<style scoped>
.container1 {
    font-family: 'Arial', sans-serif;
    width: 97%;
    padding: 20px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 10px;
}

.title {
    color: #333;
}

.subtitle1 {
    color: #666;
    margin-bottom: 20px;
}

.patent-list {
    list-style: none;
    padding: 0;
}

.patent-item {
    background-color: #f9f9f9;
    border-left: 5px solid #8ac2ff;
    margin-bottom: 10px;
    padding: 10px 20px;
    border-radius: 5px;
}

.patent-info {
    font-size: 16px;
    color: #333;
    margin-bottom: 5px;
}

.patent-summary {
    font-size: 14px;
    line-height: 1.5;
    color: #666;
}

.show-more {
    display: block;
    width: 100%;
    padding: 10px;
    background-color: #76b8ff;
    color: rgb(0, 0, 0);
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-size: 16px;
}

.show-more:hover {
    background-color: #0056b3;
}
</style>
