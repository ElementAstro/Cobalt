<!-- ServerPortScanner.vue -->
<template>
  <div class="server-port-scanner">
    <h1 class="text-h6">服务器端口扫描器</h1>
    <div class="input-group">
      <input
        v-model="baseUrl"
        placeholder="输入基础URL（如 http://example.com）"
        @keyup.enter="searchPorts"
      />
      <button @click="searchPorts" :disabled="loading">
        {{ loading ? "搜索中..." : "搜索端口" }}
      </button>
    </div>
    <div class="options">
      <label>
        <input type="checkbox" v-model="useCache" /> 使用缓存（如果可用）
      </label>
      <label>
        端口范围：
        <input type="number" v-model.number="startPort" min="1" max="65535" /> -
        <input type="number" v-model.number="endPort" min="1" max="65535" />
      </label>
    </div>
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>正在搜索端口 {{ currentPort }} / {{ endPort }}</p>
    </div>
    <transition-group
      name="list"
      tag="ul"
      class="results"
      v-else-if="results.length"
    >
      <li v-for="result in results" :key="result.url" class="result-item">
        <a :href="result.url" target="_blank" class="result-link">
          {{ result.url }} - {{ result.status }}
        </a>
        <span class="result-info">{{ result.title || "无标题" }}</span>
      </li>
    </transition-group>
    <div v-else-if="searched" class="no-results">没有找到开放的端口</div>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ServerPortScanner",
  data() {
    return {
      baseUrl: "",
      results: [],
      loading: false,
      searched: false,
      error: null,
      useCache: true,
      startPort: 1,
      endPort: 1000,
      currentPort: 0,
    };
  },
  methods: {
    async searchPorts() {
      if (!this.baseUrl) {
        this.error = "请输入有效的URL";
        return;
      }

      this.loading = true;
      this.results = [];
      this.searched = false;
      this.error = null;

      if (this.useCache) {
        const cachedResults = this.getCachedResults();
        if (cachedResults) {
          this.results = cachedResults;
          this.searched = true;
          this.loading = false;
          return;
        }
      }

      const ports = Array.from(
        { length: this.endPort - this.startPort + 1 },
        (_, i) => i + this.startPort
      );

      for (const port of ports) {
        this.currentPort = port;
        const result = await this.checkPort(port);
        if (result) {
          this.results.push(result);
        }
      }

      this.searched = true;
      this.loading = false;
      this.cacheResults();
    },
    async checkPort(port) {
      try {
        const url = `${this.baseUrl}:${port}`;
        const response = await axios.get(url, { timeout: 5000 });
        const title = this.extractTitle(response.data);
        return { url, status: response.status, title };
      } catch (error) {
        return null;
      }
    },
    extractTitle(html) {
      const match = html.match(/<title>(.*?)<\/title>/i);
      return match ? match[1] : null;
    },
    cacheResults() {
      localStorage.setItem(
        "searchResults",
        JSON.stringify({
          baseUrl: this.baseUrl,
          results: this.results,
          timestamp: Date.now(),
        })
      );
    },
    getCachedResults() {
      const cachedResults = localStorage.getItem("searchResults");
      if (cachedResults) {
        const { baseUrl, results, timestamp } = JSON.parse(cachedResults);
        const cacheAge = Date.now() - timestamp;
        if (cacheAge < 3600000 && baseUrl === this.baseUrl) {
          return results;
        }
      }
      return null;
    },
  },
  created() {
    if (this.useCache) {
      const cachedResults = this.getCachedResults();
      if (cachedResults) {
        this.results = cachedResults;
        this.searched = true;
      }
    }
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap");

body {
  background-color: #121212;
  color: #e0e0e0;
  font-family: "Roboto", sans-serif;
}

.server-port-scanner {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.title {
  color: #ffffff;
  text-align: center;
  margin-bottom: 20px;
}

.input-group {
  display: flex;
  margin-bottom: 20px;
}

.input-group input {
  flex-grow: 1;
  padding: 10px;
  font-size: 16px;
  border: 1px solid rgba(221, 221, 221, 0.6);
  border-radius: 4px 0 0 4px;
  background-color: rgba(255, 255, 255, 0.8);
}

.input-group button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: rgba(0, 123, 255, 0.8);
  color: white;
  border: none;
  border-radius: 0 4px 4px 0;
  cursor: pointer;
  transition: background-color 0.3s;
}

.input-group button:hover {
  background-color: rgba(0, 86, 179, 0.9);
}

.input-group button:disabled {
  background-color: rgba(204, 204, 204, 0.8);
  cursor: not-allowed;
}

.options {
  margin-bottom: 20px;
}

.options label {
  margin-right: 20px;
  color: #e0e0e0;
}

.options input[type="number"] {
  width: 60px;
  margin-left: 5px;
  background-color: #333;
  color: #e0e0e0;
  border: 1px solid #333;
  border-radius: 4px;
  padding: 5px;
}

.loading {
  text-align: center;
}

.spinner {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 20px auto;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.results {
  list-style-type: none;
  padding: 0;
}

.result-item {
  background-color: #2c2c2c;
  margin: 10px 0;
  padding: 15px;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.5);
  transition: all 0.3s;
}

.result-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.5);
}

.result-link {
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
}

.result-info {
  display: block;
  margin-top: 5px;
  color: #aaaaaa;
  font-size: 14px;
}

.no-results {
  text-align: center;
  color: #aaaaaa;
  margin-top: 20px;
}

.error {
  color: #dc3545;
  text-align: center;
  margin-top: 20px;
}

.list-enter-active,
.list-leave-active {
  transition: all 0.5s;
}
.list-enter,
.list-leave-to {
  opacity: 0;
  transform: translateY(30px);
}
</style>
