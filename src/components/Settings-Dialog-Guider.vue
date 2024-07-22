<template>
  <v-dialog
    max-width="400"
    v-model="$store.state.showDeviceSettingsDialog_Guider"
  >
    <v-card
      v-if="$store.state.showDeviceSettingsDialog_Guider"
      class="secondary white--text"
    >
      <v-card-title class="d-flex justify-space-between align-center">
        <div class="text-h5">{{ $t("Guider Settings") }}</div>
        <v-btn icon @click="SwitchPage">
          <v-icon>{{
            activeMenu === "connection" ? "mdi-cog" : "mdi-link"
          }}</v-icon>
        </v-btn>
      </v-card-title>
      <v-card-text>
        <!-- 设备连接的菜单内容 -->
        <div v-if="activeMenu === 'connection'">
          <v-row no-gutters class="mb-3">
            <v-col cols="8">
              <v-select
                label="选择驱动"
                :items="drivers"
                item-text="label"
                item-value="value"
                v-model="selectedDriver"
                dense
              ></v-select>
            </v-col>
            <v-col cols="4" class="d-flex align-center">
              <v-btn @click="confirmDriver" color="primary" class="ml-2" small
                >确定</v-btn
              >
            </v-col>
          </v-row>
          <v-row no-gutters class="mb-3">
            <v-col cols="8">
              <v-select
                label="选择设备"
                :items="devices"
                item-text="label"
                item-value="value"
                v-model="selectedDevice"
                dense
              ></v-select>
            </v-col>
            <v-col cols="4" class="d-flex align-center">
              <v-btn @click="confirmDevice" color="primary" class="ml-2" small
                >确定</v-btn
              >
            </v-col>
          </v-row>
        </div>
        <!-- 设备配置的菜单内容 -->
        <div v-if="activeMenu === 'configuration'">
          <v-btn @touchend="ClearCalibrationData" color="error" class="mb-3">
            清除校准数据
          </v-btn>
          <v-expansion-panels>
            <v-expansion-panel
              v-for="(item, index) in configurationItems"
              :key="index"
            >
              <v-expansion-panel-header>{{
                item.label
              }}</v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-text-field
                  v-model="item.value"
                  :label="item.label"
                  hide-details
                  dense
                ></v-text-field>
                <v-btn
                  @click="confirmConfiguration(item)"
                  color="primary"
                  class="mt-2"
                  >确定</v-btn
                >
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </div>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" text @click.native="closeDialog">{{
          $t("Close")
        }}</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data() {
    return {
      activeMenu: "connection", // 默认显示'设备连接'菜单内容
      // 添加您的数据模型
      drivers: [], // 驱动选项数组
      selectedDriver: null, // 选中的驱动
      devices: [], // 设备选项数组
      selectedDevice: null, // 选中的设备

      configurationItems: [
        { label: "Chip Width (mm)", value: "" },
        { label: "Chip Height (mm)", value: "" },
        { label: "Focal Length (mm)", value: "" },
        { label: "配置项4", value: "" },
        // 在这里添加更多的配置项
      ],
    };
  },
  methods: {
    AddDrivers(option) {
      this.drivers.push(option);
    },
    AddDevices(option) {
      this.devices.push(option);
    },
    confirmDriver() {
      // 确定驱动的逻辑
      console.log("QHYCCD | confirmDriver: ", this.selectedDriver);
      this.$bus.$emit(
        "AppSendMessage",
        "Vue_Command",
        "ConfirmIndiDriver:" + this.selectedDriver
      );
    },
    connectIndiServer() {
      // 连接indiServer的逻辑
      // this.$bus.$emit('AppSendMessage', 'Vue_Command', 'CS');
    },
    confirmDevice() {
      // 确定设备的逻辑
      this.$bus.$emit(
        "AppSendMessage",
        "Vue_Command",
        "ConfirmIndiDevice:" + this.selectedDevice
      );
      this.$bus.$emit("AppUpdateDevices", "Guider", this.selectedDevice);
    },
    closeDialog() {
      this.$store.state.showDeviceSettingsDialog_Guider = false;
    },
    confirmConfiguration(item) {
      // 在这里处理确认按钮的逻辑，可以访问item.value来获取输入框的值
      console.log(`QHYCCD | confirmConfiguration: ${item.value}`);
    },
    SwitchPage() {
      this.activeMenu =
        this.activeMenu === "connection" ? "configuration" : "connection";
    },
    ClearCalibrationData() {
      this.$bus.$emit("AppSendMessage", "Vue_Command", "ClearCalibrationData");
    },
  },
  // 计算属性和其他选项保持不变
};
</script>

<style scoped>
.button-container {
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.v-card-title .button-container {
  margin-left: auto;
}

.v-btn {
  margin-top: 8px;
}

.mb-3 {
  margin-bottom: 16px;
}

.ml-2 {
  margin-left: 8px;
}
</style>
