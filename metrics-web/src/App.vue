<template>
  <div class="workspace">
    <section class="hero-band">
      <div class="hero-copy">
        <p class="eyebrow">Software Metrics Studio</p>
        <h1>自动化度量工具设计实验平台</h1>
        <p class="hero-text">
          支持代码粘贴与真实 Java 文件上传，统一完成 CK、LK、复杂度、规模与项目估算分析。
        </p>
      </div>
      <div class="hero-summary">
        <div class="summary-chip">AST 代码分析</div>
        <div class="summary-chip">多文件上传</div>
        <div class="summary-chip">设计输入与估算</div>
      </div>
    </section>

    <section class="main-grid">
      <aside class="control-panel">
        <div class="panel-shell">
          <div class="panel-head">
            <h2>实验输入</h2>
            <button class="ghost-btn" @click="fillExample">载入示例</button>
          </div>

          <div class="mode-switch">
            <button class="mode-btn" :class="{ active: inputMode === 'code' }" @click="setInputMode('code')">
              代码粘贴
            </button>
            <button class="mode-btn" :class="{ active: inputMode === 'upload' }" @click="setInputMode('upload')">
              文件上传
            </button>
          </div>

          <div class="form-block">
            <div class="block-title">设计阶段特征</div>
            <div class="field-grid">
              <label>
                <span>参与者数</span>
                <input v-model.number="form.design.actors" type="number" min="0" />
              </label>
              <label>
                <span>用例数</span>
                <input v-model.number="form.design.useCases" type="number" min="0" />
              </label>
              <label>
                <span>类图类数</span>
                <input v-model.number="form.design.classes" type="number" min="0" />
              </label>
              <label>
                <span>子类数</span>
                <input v-model.number="form.design.subclasses" type="number" min="0" />
              </label>
              <label>
                <span>流程判定数</span>
                <input v-model.number="form.design.decisions" type="number" min="0" />
              </label>
              <label>
                <span>事务数</span>
                <input v-model.number="form.design.transactions" type="number" min="0" />
              </label>
              <label>
                <span>实体数</span>
                <input v-model.number="form.design.entities" type="number" min="0" />
              </label>
            </div>
          </div>

          <div class="form-block">
            <div class="block-title">估算参数</div>
            <div class="field-grid compact">
              <label>
                <span>团队人数</span>
                <input v-model.number="form.estimate.teamMembers" type="number" min="1" />
              </label>
              <label>
                <span>人月成本</span>
                <input v-model.number="form.estimate.monthlyRate" type="number" min="1000" step="1000" />
              </label>
              <label>
                <span>生产率系数</span>
                <input v-model.number="form.estimate.productivity" type="number" min="0.2" step="0.1" />
              </label>
            </div>
          </div>

          <div v-if="inputMode === 'code'" class="form-block">
            <div class="panel-head inner">
              <div class="block-title">Java 源代码</div>
              <div class="code-actions">
                <button class="inline-btn" @click="clearCode">清空</button>
              </div>
            </div>
            <textarea
              v-model="form.code"
              class="code-editor"
              placeholder="粘贴 Java 类代码，系统将基于 AST 统计 CK / LK / 复杂度 / LoC 等指标。"
            />
          </div>

          <div v-else class="form-block">
            <div class="panel-head inner">
              <div class="block-title">Java 文件上传</div>
              <div class="code-actions">
                <button class="inline-btn" @click="clearFiles">清空</button>
              </div>
            </div>
            <label class="upload-box">
              <input class="file-input" type="file" accept=".java" multiple @change="handleFileChange" />
              <span class="upload-title">选择一个或多个 .java 文件</span>
              <span class="upload-hint">上传后由后端直接按文件内容批量解析并汇总指标</span>
            </label>
            <div v-if="selectedFiles.length" class="file-list">
              <div v-for="file in selectedFiles" :key="file.name + file.size" class="file-item">
                <strong>{{ file.name }}</strong>
                <span>{{ formatFileSize(file.size) }}</span>
              </div>
            </div>
          </div>

          <button class="primary-btn" :disabled="loading" @click="analyzeAll">
            {{ loading ? '分析中...' : '执行度量分析' }}
          </button>
          <p v-if="errorMessage" class="error-text">{{ errorMessage }}</p>
        </div>
      </aside>

      <main class="dashboard">
        <div v-if="!result" class="empty-shell">
          <div class="empty-title">等待分析结果</div>
          <p>输入代码或上传 Java 文件后，即可生成课程实验所需的自动化度量报告。</p>
        </div>

        <template v-else>
          <section class="kpi-grid">
            <article class="kpi-card navy">
              <span>工作量</span>
              <strong>{{ result.estimation.Effort }}</strong>
              <small>结合代码规模与设计输入的综合估算</small>
            </article>
            <article class="kpi-card green">
              <span>开发周期</span>
              <strong>{{ result.estimation.Time }}</strong>
              <small>按简化 COCOMO 模型推导</small>
            </article>
            <article class="kpi-card amber">
              <span>项目成本</span>
              <strong>{{ result.estimation.Cost }}</strong>
              <small>按团队单价估算总人月成本</small>
            </article>
            <article class="kpi-card coral">
              <span>建议人力</span>
              <strong>{{ result.estimation.People }}</strong>
              <small>当前配置 {{ result.estimation.ConfiguredPeople }}</small>
            </article>
          </section>

          <section class="overview-band">
            <div class="metric-pill">
              <span>LoC</span>
              <strong>{{ result.overview.loc }}</strong>
            </div>
            <div class="metric-pill">
              <span>类数</span>
              <strong>{{ result.overview.classes }}</strong>
            </div>
            <div class="metric-pill">
              <span>方法数</span>
              <strong>{{ result.overview.methods }}</strong>
            </div>
            <div class="metric-pill">
              <span>文件数</span>
              <strong>{{ result.overview.sourceFiles }}</strong>
            </div>
            <div class="metric-pill">
              <span>可维护性</span>
              <strong>{{ result.traditionalMetrics.MaintainabilityLevel }}</strong>
            </div>
          </section>

          <section v-if="result.uploadSummary && result.uploadSummary.fileCount" class="data-panel upload-panel">
            <div class="section-title">上传摘要</div>
            <div class="upload-tags">
              <span class="upload-tag" v-for="name in result.uploadSummary.files" :key="name">{{ name }}</span>
            </div>
          </section>

          <section class="panel-row">
            <article class="data-panel">
              <div class="section-title">CK 模型</div>
              <table class="metrics-table">
                <tbody>
                  <tr v-for="item in ckRows" :key="item.key">
                    <td>{{ item.label }}</td>
                    <td>{{ result.ckMetrics[item.key] }}</td>
                    <td>{{ item.note }}</td>
                  </tr>
                </tbody>
              </table>
            </article>
            <article class="data-panel">
              <div class="section-title">LK 与传统指标</div>
              <table class="metrics-table">
                <tbody>
                  <tr v-for="item in lkRows" :key="item.key">
                    <td>{{ item.label }}</td>
                    <td>{{ result.lkMetrics[item.key] }}</td>
                    <td>{{ item.note }}</td>
                  </tr>
                  <tr>
                    <td>圈复杂度</td>
                    <td>{{ result.traditionalMetrics.CyclomaticComplexity }}</td>
                    <td>流程控制复杂程度</td>
                  </tr>
                  <tr>
                    <td>平均复杂度</td>
                    <td>{{ result.traditionalMetrics.AverageComplexity }}</td>
                    <td>方法级复杂度均值</td>
                  </tr>
                </tbody>
              </table>
            </article>
          </section>

          <section class="panel-row">
            <article class="data-panel">
              <div class="section-title">设计阶段度量</div>
              <div class="insight-list">
                <div class="insight-item">
                  <span>用例点</span>
                  <strong>{{ result.designMetrics.UseCasePoints }}</strong>
                </div>
                <div class="insight-item">
                  <span>设计类数</span>
                  <strong>{{ result.designMetrics.Classes }}</strong>
                </div>
                <div class="insight-item">
                  <span>事务数</span>
                  <strong>{{ result.designMetrics.Transactions }}</strong>
                </div>
                <div class="insight-item">
                  <span>一致性判断</span>
                  <strong>{{ result.designMetrics.DecisionAlignment }}</strong>
                </div>
              </div>
            </article>
            <article class="data-panel">
              <div class="section-title">风险信号</div>
              <div class="risk-list">
                <div
                  v-for="signal in result.riskSignals"
                  :key="signal.title"
                  class="risk-item"
                  :class="signal.level"
                >
                  <strong>{{ signal.title }}</strong>
                  <p>{{ signal.detail }}</p>
                </div>
              </div>
            </article>
          </section>

          <section class="data-panel">
            <div class="section-title">类级明细</div>
            <div class="class-grid">
              <article v-for="item in result.classMetrics" :key="item.sourceFile + item.name" class="class-card">
                <div class="class-head">
                  <strong>{{ item.name }}</strong>
                  <span>{{ item.sourceFile }}</span>
                </div>
                <div class="class-meta">
                  <span>WMC {{ item.WMC }}</span>
                  <span>CBO {{ item.CBO }}</span>
                  <span>RFC {{ item.RFC }}</span>
                  <span>LCOM {{ item.LCOM }}</span>
                  <span>复杂度 {{ item.complexity }}</span>
                </div>
                <table class="mini-table">
                  <tbody>
                    <tr v-for="method in item.methods" :key="method.name">
                      <td>{{ method.name }}</td>
                      <td>{{ method.visibility }}</td>
                      <td>CC {{ method.complexity }}</td>
                    </tr>
                  </tbody>
                </table>
              </article>
            </div>
          </section>
        </template>
      </main>
    </section>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import axios from 'axios'

const exampleCode = `public class OrderService extends BaseService {
    private OrderRepository repository;
    private PaymentGateway paymentGateway;
    private int retryCount;

    @Override
    public void execute() {
        if (retryCount > 0 && repository != null) {
            for (int i = 0; i < retryCount; i++) {
                paymentGateway.pay();
            }
        }
    }

    public double calculateTotal(double amount, double discount) {
        if (discount > 0) {
            return amount - discount;
        }
        return amount;
    }
}

class BaseService {
    public void execute() {
    }
}

class OrderRepository {
}

class PaymentGateway {
    public void pay() {
    }
}`

const inputMode = ref('code')
const selectedFiles = ref([])
const form = reactive({
  code: '',
  design: {
    actors: 2,
    useCases: 4,
    classes: 4,
    subclasses: 1,
    decisions: 4,
    transactions: 6,
    entities: 3,
  },
  estimate: {
    teamMembers: 4,
    monthlyRate: 15000,
    productivity: 1.0,
  },
})

const loading = ref(false)
const result = ref(null)
const errorMessage = ref('')

const ckRows = [
  { key: 'WMC', label: 'WMC', note: '类方法规模与综合复杂程度' },
  { key: 'DIT', label: 'DIT', note: '继承层次深度' },
  { key: 'CBO', label: 'CBO', note: '类之间耦合程度' },
  { key: 'RFC', label: 'RFC', note: '潜在响应集合规模' },
  { key: 'LCOM', label: 'LCOM', note: '类内聚性水平' },
  { key: 'NOC', label: 'NOC', note: '子类数量或扩展能力' },
]

const lkRows = [
  { key: 'NOA', label: 'NOA', note: '属性数量' },
  { key: 'NPM', label: 'NPM', note: '公开方法数量' },
  { key: 'NIV', label: 'NIV', note: '实例变量数量' },
  { key: 'NVO', label: 'NVO', note: '重写方法数量' },
  { key: 'ClassCount', label: '类数量', note: '本次分析的类实体数' },
  { key: 'MethodCount', label: '方法数量', note: '本次分析的方法总数' },
]

const setInputMode = (mode) => {
  inputMode.value = mode
  errorMessage.value = ''
}

const fillExample = () => {
  form.code = exampleCode
  inputMode.value = 'code'
}

const clearCode = () => {
  form.code = ''
}

const handleFileChange = (event) => {
  selectedFiles.value = Array.from(event.target.files || []).filter((file) => file.name.endsWith('.java'))
}

const clearFiles = () => {
  selectedFiles.value = []
}

const formatFileSize = (size) => {
  if (size < 1024) return `${size} B`
  if (size < 1024 * 1024) return `${(size / 1024).toFixed(1)} KB`
  return `${(size / (1024 * 1024)).toFixed(1)} MB`
}

const analyzeByCode = async () => {
  const response = await axios.post('http://localhost:8080/api/analyze', {
    code: form.code,
    design: form.design,
    estimate: form.estimate,
  })
  result.value = response.data
}

const analyzeByUpload = async () => {
  const payload = new FormData()
  selectedFiles.value.forEach((file) => payload.append('files', file))
  payload.append('actors', String(form.design.actors))
  payload.append('useCases', String(form.design.useCases))
  payload.append('classes', String(form.design.classes))
  payload.append('subclasses', String(form.design.subclasses))
  payload.append('decisions', String(form.design.decisions))
  payload.append('transactions', String(form.design.transactions))
  payload.append('entities', String(form.design.entities))
  payload.append('teamMembers', String(form.estimate.teamMembers))
  payload.append('monthlyRate', String(form.estimate.monthlyRate))
  payload.append('productivity', String(form.estimate.productivity))

  const response = await axios.post('http://localhost:8080/api/analyze/upload', payload, {
    headers: {
      'Content-Type': 'multipart/form-data',
    },
  })
  result.value = response.data
}

const analyzeAll = async () => {
  if (inputMode.value === 'code' && !form.code.trim()) {
    errorMessage.value = '请先输入或载入 Java 代码样例。'
    return
  }
  if (inputMode.value === 'upload' && selectedFiles.value.length === 0) {
    errorMessage.value = '请先选择至少一个 .java 文件。'
    return
  }

  errorMessage.value = ''
  loading.value = true
  try {
    if (inputMode.value === 'code') {
      await analyzeByCode()
    } else {
      await analyzeByUpload()
    }
  } catch (error) {
    console.error(error)
    errorMessage.value = '后端服务连接失败，请确认 Spring Boot 服务已运行在 8080 端口。'
  } finally {
    loading.value = false
  }
}

fillExample()
</script>
