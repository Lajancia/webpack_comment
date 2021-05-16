<template>
  <div>
    <div class="flex">
      <el-input
        placeholder="사용자 입력"
        v-model="cmtWriter"
        class="w-24 ml-4"
      ></el-input>
      <el-input
        placeholder="내용을 입력해주세요"
        v-model="cmtText"
        class="w-64 ml-2"
        @keyup.enter="addComment"
      ></el-input>
      <el-button plain class="ml-4 hover:bg-purple-700" @click="addComment">
        입력</el-button
      >
    </div>
    <br />
    <div class="text-left" v-for="c in comments" :key="`comments-${c.id}`">
      <div v-if="c.editF" class="border-t-2 ">
        <div class="flex my-4">
          <el-avatar
            shape="circle"
            :size="30"
            :src="circleUrl"
            class="mx-4"
          ></el-avatar>
          <div>
            <b>{{ c.createAt }}</b>
            <br />
            {{ c.text }}
          </div>
          <div class="ml-10">
            <i
              class="el-icon-s-comment cursor-pointer hover:text-green-600"
              @click="openSubcmt(c)"
            ></i>
            <i
              class="el-icon-edit cursor-pointer hover:text-green-600"
              @click="openEdit(c)"
            ></i>
            <i
              class="el-icon-delete cursor-pointer hover:text-green-600"
              @click="delComment(c)"
            ></i>
          </div>
        </div>
      </div>
      <div v-else class="mb-4 ml-14">
        <el-input
          placeholder="사용자 입력"
          v-model="c.editWriter"
          class="w-24"
          >{{ c.createAt }}</el-input
        >
        <el-input
          placeholder="내용을 입력해주세요"
          v-model="c.editT"
          class="w-64 ml-2"
          @keyup.enter="editComment(c)"
        ></el-input>
        <el-button plain class="ml-4" @click="editComment(c)">입력</el-button>
        <el-button plain class="ml-4" @click="openEdit(c)">취소</el-button>
      </div>
      <div v-if="c.subCmtF" class="mb-4 ml-14">
        <el-input
          placeholder="사용자 입력"
          v-model="c.subCmtW"
          class="w-24"
        ></el-input>
        <el-input
          placeholder="내용을 입력해주세요"
          v-model="c.subCmtT"
          class="w-64 ml-2"
          @keyup.enter="addSubcomment(c)"
        ></el-input>
        <el-button plain class="ml-4" @click="addSubcomment(c)">입력</el-button>
      </div>
      <div class="mb-4">
        <div
          v-for="s in c.subcomments"
          :key="`subcomment-${s.id}`"
          class="pl-14"
        >
          <div v-if="s.noEdit">
            <div class="flex bg-gray-300 mb-4 w-11/12 p-2">
              <div>
                ┗

                <el-avatar
                  shape="square"
                  :size="20"
                  :src="squareUrl"
                  class="mx-2"
                ></el-avatar>
              </div>
              <div>
                <b>
                  {{ s.createAt }}
                </b>
                <br />
                <div>
                  {{ s.text }}
                </div>
              </div>
              <div class="ml-5">
                <i
                  class="el-icon-edit cursor-pointer hover:text-green-600"
                  @click="openEditSub(s)"
                ></i>
                <i
                  class="el-icon-delete cursor-pointer hover:text-green-600"
                  @click="delSubC(s)"
                ></i>
              </div>
            </div>
          </div>

          <div v-else class="mb-4">
            <el-input
              placeholder="사용자 입력"
              v-model="s.editSW"
              class="w-24"
            ></el-input>
            <el-input
              placeholder="내용을 입력해주세요"
              v-model="s.editSubText"
              class="w-64 ml-2"
              @keyup.enter="editSubComment(s)"
            ></el-input>
            <el-button plain class="ml-4" @click="editSubComment(s)"
              >입력</el-button
            >
            <el-button plain class="ml-4" @click="openEditSub(s)"
              >취소</el-button
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  props: {
    recompId: {
      type: Number,
      default: 1, //recompId 안주면 1
    },
  },
  data() {
    return {
      circleUrl:
        "https://cube.elemecdn.com/3/7c/3ea6beec64369c2642b92c6726f1epng.png",
      squareUrl:
        "https://cube.elemecdn.com/9/c2/f0ee8a3c7c9638a54940382568c9dpng.png",
      comments: [],
      cmtText: "",
      cmtWriter: "",
    };
  },
  mounted() {
    this.getComments();
  },
  methods: {
    async getComments() {
      const res = await axios.get(
        `http://localhost:3000/comments?recompId=${this.recompId}&_embed=subcomments`
      );
      const reRes = res.data.map((c) => {
        (c.subCmtF = false),
          (c.subCmtW = ""),
          (c.subCmtT = ""),
          (c.editF = true),
          (c.editT = "");
        return c;
      });
      this.comments = reRes;
    },
    async addComment() {
      if (!this.cmtText || !this.cmtWriter) {
        this.$message({
          message: "작성자 또는 내용이 입력되지 않았습니다.",
          type: "warning",
        });
        return;
      }
      await axios.post(`http://localhost:3000/comments`, {
        recompId: this.recompId,
        text: this.cmtText,
        createAt: this.cmtWriter,
      });
      this.getComments();
      this.cmtText = "";
    },
    delComment(c) {
      this.$confirm("삭제하시겠습니까??", "Warning", {
        confirmButtonText: "예",
        cancelButtonText: "아니오",
        type: "warning",
      })
        .then(async () => {
          await axios.delete(`http://localhost:3000/comments/` + c.id);
          this.$message({
            type: "success",
            message: "삭제되었습니다.",
          });
          this.getComments();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "삭제를 취소합니다.",
          });
        });
    },
    async openSubcmt(c) {
      c.subCmtF = !c.subCmtF;
    },
    async addSubcomment(c) {
      if (!c.subCmtT || !c.subCmtW) {
        this.$message({
          message: "작성자 또는 내용이 입력되지 않았습니다.",
          type: "warning",
        });
        return;
      }
      await axios.post("http://localhost:3000/subcomments", {
        commentId: c.id,
        text: c.subCmtT,
        createAt: c.subCmtW,
        noEdit: true,
        editSW: "",
        editSubText: "",
      });
      this.getComments();
      c.subCmtT = "";
    },
    delSubC(s) {
      this.$confirm("삭제하시겠습니까??", "Warning", {
        confirmButtonText: "예",
        cancelButtonText: "아니오",
        type: "warning",
      })
        .then(async () => {
          await axios.delete(`http://localhost:3000/subcomments/` + s.id);
          this.$message({
            type: "success",
            message: "삭제되었습니다.",
          });
          this.getComments();
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "삭제를 취소합니다.",
          });
        });
    },
    async openEdit(c) {
      c.editF = !c.editF;
      c.editWriter = c.createAt;
      c.editT = c.text;
    },
    editComment(c) {
      if (!c.editT) {
        this.$message({
          message: "내용이 입력되지 않았습니다.",
          type: "warning",
        });
        return;
      }
      this.$confirm("정말 수정 하시겠습니까??", "Warning", {
        confirmButtonText: "예",
        cancelButtonText: "아니오",
        type: "warning",
      })
        .then(async () => {
          await axios.patch(`http://localhost:3000/comments/${c.id}`, {
            text: c.editT,
            createAt: c.editWriter,
          });
          this.getComments();
          this.$message({
            type: "info",
            message: "수정되었습니다.",
          });
          c.editF = !c.editF;
          c.editT = "";
          c.editWriter = "";
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "수정을 취소합니다.",
          });
        });
    },
    async openEditSub(s) {
      await axios.patch(`http://localhost:3000/subcomments/${s.id}`, {
        noEdit: !s.noEdit,
        editSW: s.createAt,
        editSubText: s.text,
      });
      this.getComments();
    },
    editSubComment(s) {
      if (!s.editSW || !s.editSubText) {
        this.$message({
          message: "내용이 입력되지 않았습니다.",
          type: "warning",
        });
        return;
      }
      this.$confirm("정말 수정 하시겠습니까??", "Warning", {
        confirmButtonText: "예",
        cancelButtonText: "아니오",
        type: "warning",
      })
        .then(async () => {
          await axios.patch(`http://localhost:3000/subcomments/${s.id}`, {
            text: s.editSubText,
            createAt: s.editSW,
            noEdit: true,
            editSW: "",
            editSubText: "",
          });
          this.getComments();
          this.$message({
            type: "info",
            message: "수정되었습니다.",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "수정을 취소합니다.",
          });
        });
    },
  },
};
</script>

<style lang="scss" scoped></style>
