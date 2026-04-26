<template>
  <div class="container">
    <header>
      <h1>软件度量自动化工具 <small>Vue3 版</small></h1>
    </header>

    <main>
      <div class="editor-box">
        <textarea
            v-model="sourceCode"
            placeholder="在此粘贴 Java 源代码进行质量度量分析..."
        ></textarea>
        <button @click="handleAnalyze" :disabled="loading">
          {{ loading ? '分析中...' : '开始执行度量' }}
        </button>
      </div>

      <div v-if="results" class="results-box">
        <h2>度量结果报告</h2>
        <div class="grid">
          <div class="item"><span>代码行数 (LoC):</span> <strong>{{ results.LoC }}</strong></div>
          <div class="item"><span>方法加权 (WMC):</span> <strong>{{ results.WMC }}</strong></div>
          <div class="item"><span>继承深度 (DIT):</span> <strong>{{ results.DIT }}</strong></div>
          <div class="item"><span>圈复杂度 (Complexity):</span> <strong>{{ results.Complexity }}</strong></div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const sourceCode = ref('');
const results = ref(null);
const loading = ref(false);

const handleAnalyze = async () => {
  if (!sourceCode.value) return alert("请先输入代码");
  loading.value = true;
  try {
    const res = await axios.post('http://localhost:8080/api/analyze', sourceCode.value, {
      headers: { 'Content-Type': 'text/plain' }
    });
    results.value = res.data;
  } catch (err) {
    alert("连接后端失败，请确认 IDEA 项目已启动 (Port 8080)");
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
.container { max-width: 800px; margin: 0 auto; padding: 20px; font-family: system-ui; }
header { text-align: center; margin-bottom: 30px; }
textarea { width: 100%; height: 300px; border-radius: 8px; border: 1px solid #ddd; padding: 15px; font-family: monospace; }
button { width: 100%; margin-top: 15px; padding: 12px; background: #42b883; color: white; border: none; border-radius: 8px; cursor: pointer; font-size: 16px; }
.results-box { margin-top: 30px; padding: 20px; background: #f8f9fa; border-radius: 8px; border: 1px solid #e9ecef; }
.grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 10px; }
.item { padding: 10px; background: white; border-radius: 4px; border-left: 4px solid #42b883; }
strong { float: right; color: #42b883; }
</style>