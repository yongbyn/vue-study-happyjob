<template>
  <div class="divNoticeList">
    <NoticeModal
      v-if="modalState.modalState"
      @postSuccess="searchList"
      @modalClose="() => (noticeIdx = 0)"
      :idx="noticeIdx"
    />
    현재 페이지: {{ cPage }} 총 개수: {{ noticeList?.noticeCnt }}
    <table>
      <colgroup>
        <col width="10%" />
        <col width="50%" />
        <col width="30%" />
        <col width="10%" />
      </colgroup>

      <thead>
        <tr>
          <th scope="col">번호</th>
          <th scope="col">제목</th>
          <th scope="col">작성일</th>
          <th scope="col">작성자</th>
        </tr>
      </thead>
      <tbody>
        <template v-if="noticeList">
          <template v-if="noticeList.noticeCnt > 0">
            <tr
              v-for="notice in noticeList.notice"
              :key="notice.noticeIdx"
              @click="handlerModal(notice.noticeIdx)"
            >
              <td>{{ notice.noticeIdx }}</td>
              <td>{{ notice.title }}</td>
              <td>{{ notice.createdDate.substr(0, 10) }}</td>
              <td>{{ notice.author }}</td>
            </tr>
          </template>
          <template v-else>
            <tr>
              <td colspan="7">일치하는 검색 결과가 없습니다</td>
            </tr>
          </template>
        </template>
      </tbody>
    </table>
    <Pagination
      :totalItems="noticeList?.noticeCnt || 0"
      :items-per-page="5"
      :max-pages-shown="5"
      :onClick="searchList"
      v-model="cPage"
    />
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute } from "vue-router";
import Pagination from "../../../common/Pagination.vue";
import { useModalStore } from "../../../../stores/modalState";

const route = useRoute();
const noticeList = ref();
const cPage = ref(1);
const modalState = useModalStore();
const noticeIdx = ref(0);

const searchList = () => {
  const param = new URLSearchParams({
    searchTitle: route.query.searchTitle || "",
    searchStDate: route.query.searchStDate || "",
    searchEdDate: route.query.searchEdDate || "",
    currentPage: cPage.value,
    pageSize: 5,
  });
  axios.post("/api/board/noticeListJson.do", param).then((res) => {
    noticeList.value = res.data;
  });
};

const handlerModal = (idx) => {
  noticeIdx.value = idx;
  modalState.setModalState();
};

watch(route, searchList);
// watch(modalState, searchList); // emit 대신 이거 해도 됐었음 (용빈)

// 화면이 초기에 열렸을 때 실행
onMounted(() => {
  searchList();
});
</script>

<style lang="scss" scoped>
table {
  width: 100%;
  border-collapse: collapse;
  margin: 20px 0px 0px 0px;
  font-size: 18px;
  text-align: left;

  th,
  td {
    padding: 8px;
    text-align: left;
    border-bottom: 1px solid #ddd;
    text-align: center;
  }

  th {
    background-color: #2676bf;
    color: #ddd;
  }

  /* 테이블 올렸을 때 */
  tbody tr:hover {
    background-color: #d3d3d3;
    opacity: 0.9;
    cursor: pointer;
  }
}
</style>
