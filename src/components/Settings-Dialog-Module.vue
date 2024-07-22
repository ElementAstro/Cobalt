<template>
  <div class="mod-manager">
    <v-card>
      <v-container>
        <v-row class="d-flex justify-space-between align-center">
          <v-col cols="auto">
            <h1 class="title">我的模组</h1>
          </v-col>
          <v-col cols="auto">
            <v-btn @click="refreshMods" color="primary">刷新</v-btn>
            <v-btn @click="addMod" color="info">添加模组</v-btn>
            <v-text-field
              v-model="searchQuery"
              label="搜索模组..."
              clearable
              style="width: 200px"
            />
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="filteredMods"
          :loading="isLoading"
          :items-per-page="10"
          item-key="id"
          class="elevation-1"
        >
          <template v-slot:item.actions="{ item }">
            <v-btn small @click="toggleModStatus(item)">
              {{ item.isActive ? "禁用" : "启用" }}
            </v-btn>
            <v-btn small @click="showDetails(item)">详情</v-btn>
            <v-btn small color="error" @click="deleteMod(item)">删除</v-btn>
          </template>
        </v-data-table>

        <v-dialog v-model="showModDetails" max-width="600px">
          <v-card>
            <v-card-title>
              {{ selectedMod?.name }}
              <v-spacer></v-spacer>
              <v-btn icon @click="showModDetails = false">
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </v-card-title>
            <v-card-text>
              <v-list dense>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>版本</v-list-item-title>
                    <v-list-item-subtitle>{{
                      selectedMod?.version
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>作者</v-list-item-title>
                    <v-list-item-subtitle>{{
                      selectedMod?.creator
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>描述</v-list-item-title>
                    <v-list-item-subtitle>{{
                      selectedMod?.description
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title>标签</v-list-item-title>
                    <v-list-item-subtitle>
                      <v-chip
                        v-for="tag in selectedMod?.tags"
                        :key="tag"
                        color="info"
                        text-color="white"
                      >
                        {{ tag }}
                      </v-chip>
                    </v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card-text>
          </v-card>
        </v-dialog>

        <v-dialog v-model="showErrorModal" max-width="400px">
          <v-card>
            <v-card-title>错误</v-card-title>
            <v-card-text>{{ errorMessage }}</v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" @click="showErrorModal = false"
                >确定</v-btn
              >
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-dialog v-model="showAddModModal" max-width="500px">
          <v-card>
            <v-card-title>添加模组</v-card-title>
            <v-card-text>
              <v-form
                ref="addModFormRef"
                v-model="addModFormValid"
                :rules="addModRules"
              >
                <v-text-field
                  v-model="addModForm.name"
                  :rules="addModRules.name"
                  label="名称"
                  required
                />
                <v-text-field
                  v-model="addModForm.version"
                  :rules="addModRules.version"
                  label="版本"
                  required
                />
                <v-text-field
                  v-model="addModForm.creator"
                  :rules="addModRules.creator"
                  label="作者"
                  required
                />
                <v-text-field
                  v-model="addModForm.gameVersion"
                  :rules="addModRules.gameVersion"
                  label="游戏版本"
                  required
                />
                <v-textarea v-model="addModForm.description" label="描述" />
                <v-chip-group v-model="addModForm.tags" column multiple>
                  <v-chip v-for="tag in addModForm.tags" :key="tag">
                    {{ tag }}
                    <v-icon small @click="removeTag(tag)">mdi-close</v-icon>
                  </v-chip>
                  <v-text-field
                    v-model="newTag"
                    label="添加标签"
                    @keyup.enter="addTag"
                  />
                </v-chip-group>
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn @click="showAddModModal = false">取消</v-btn>
              <v-btn color="primary" @click="submitAddMod">确定</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-container>
    </v-card>
  </div>
</template>

<script>
import Vue from "vue";
import Vuetify, {
  VCard,
  VContainer,
  VRow,
  VCol,
  VBtn,
  VTextField,
  VDataTable,
  VDialog,
  VList,
  VListItem,
  VListItemContent,
  VListItemTitle,
  VListItemSubtitle,
  VChip,
  VChipGroup,
  VIcon,
  VForm,
  VTextarea,
  VSpacer,
} from "vuetify/lib";
import "vuetify/dist/vuetify.min.css";
import "@mdi/font/css/materialdesignicons.css";

Vue.use(Vuetify);

export default {
  data() {
    return {
      mods: [],
      isLoading: false,
      searchQuery: "",
      showModDetails: false,
      selectedMod: null,
      showErrorModal: false,
      errorMessage: "",
      showAddModModal: false,
      addModFormValid: false,
      addModForm: {
        name: "",
        version: "",
        creator: "",
        gameVersion: "",
        description: "",
        tags: [],
      },
      newTag: "",
      addModRules: {
        name: [{ required: true, message: "请输入模组名称", trigger: "blur" }],
        version: [
          { required: true, message: "请输入模组版本", trigger: "blur" },
        ],
        creator: [
          { required: true, message: "请输入作者名称", trigger: "blur" },
        ],
        gameVersion: [
          { required: true, message: "请输入游戏版本", trigger: "blur" },
        ],
      },
      headers: [
        { text: "名称", value: "name", sortable: true },
        { text: "版本", value: "version", sortable: false },
        { text: "作者", value: "creator", sortable: false },
        { text: "游戏版本", value: "gameVersion", sortable: false },
        { text: "状态", value: "isActive", sortable: false },
        { text: "操作", value: "actions", sortable: false, align: "center" },
      ],
    };
  },
  computed: {
    filteredMods() {
      if (!this.searchQuery) return this.mods;
      const query = this.searchQuery.toLowerCase();
      return this.mods.filter(
        (mod) =>
          mod.name.toLowerCase().includes(query) ||
          mod.description.toLowerCase().includes(query) ||
          mod.creator.toLowerCase().includes(query)
      );
    },
  },
  methods: {
    fetchMods() {
      this.isLoading = true;
      // 模拟后端请求
      setTimeout(() => {
        this.mods = [
          {
            id: "1",
            name: "模组1",
            version: "1.0.0",
            gameVersion: "1.16.5",
            creator: "作者1",
            description: "模组1描述",
            tags: ["标签1", "标签2"],
            isActive: true,
          },
          {
            id: "2",
            name: "模组2",
            version: "1.1.0",
            gameVersion: "1.16.5",
            creator: "作者2",
            description: "模组2描述",
            tags: ["标签2", "标签3"],
            isActive: false,
          },
        ];
        this.isLoading = false;
      }, 1000);
    },
    refreshMods() {
      this.fetchMods();
    },
    addMod() {
      this.showAddModModal = true;
    },
    submitAddMod() {
      this.$refs.addModFormRef.validate((valid) => {
        if (valid) {
          const newMod = {
            id: Date.now().toString(),
            ...this.addModForm,
            isActive: true,
          };
          this.mods.push(newMod);
          this.showAddModModal = false;
          this.$refs.addModFormRef.reset();
          this.addModForm = {
            name: "",
            version: "",
            creator: "",
            gameVersion: "",
            description: "",
            tags: [],
          };
          this.$toast("模组添加成功", { type: "success" });
        } else {
          this.$toast("请填写所有必填字段", { type: "error" });
        }
      });
    },
    toggleModStatus(mod) {
      mod.isActive = !mod.isActive;
      this.$toast(`模组已${mod.isActive ? "启用" : "禁用"}`, {
        type: "success",
      });
    },
    deleteMod(mod) {
      this.mods = this.mods.filter((m) => m.id !== mod.id);
      this.$toast("模组已删除", { type: "success" });
    },
    showDetails(mod) {
      this.selectedMod = mod;
      this.showModDetails = true;
    },
    addTag() {
      if (this.newTag && !this.addModForm.tags.includes(this.newTag)) {
        this.addModForm.tags.push(this.newTag);
        this.newTag = "";
      }
    },
    removeTag(tag) {
      this.addModForm.tags = this.addModForm.tags.filter((t) => t !== tag);
    },
  },
  created() {
    this.fetchMods();
  },
};
</script>

<style scoped>
.mod-manager {
  padding: 20px;
}

.title {
  margin: 0;
}

.v-btn {
  margin: 0 5px;
}

.v-data-table {
  margin-top: 20px;
}

.v-dialog .v-card {
  overflow: visible;
}
</style>
