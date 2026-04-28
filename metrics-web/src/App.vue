<!-- <template>
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
</script> -->
<template>
  <div class="workspace">
    <section class="hero-band">
      <div class="hero-copy">
        <p class="eyebrow">Enterprise Analysis Engine</p>
        <h1>自动化度量工具设计实验平台</h1>
        <p class="hero-text">
          基于 JDT 解析引擎的深度源码扫描，支持 CK/LK 模型及项目成本精准估算。
        </p>
      </div>
      <div class="hero-summary">
        <div class="summary-chip">Source Scan</div>
        <div class="summary-chip">Metrics Model</div>
        <div class="summary-chip">Cost Estimate</div>
      </div>
    </section>

    <section class="main-grid">
      <aside class="control-panel">
        <div class="panel-shell">
          <div class="panel-head">
            <h2>配置终端 / CONFIG</h2>
            <button class="action-outline-btn" @click="fillExample">
              <span class="icon">⚡</span> 载入标准示例
            </button>
          </div>

          <div class="mode-switch">
            <button class="mode-btn" :class="{ active: inputMode === 'code' }" @click="setInputMode('code')">代码粘贴</button>
            <button class="mode-btn" :class="{ active: inputMode === 'upload' }" @click="setInputMode('upload')">文件上传</button>
          </div>

          <div class="form-scroll">
            <div class="config-section">
              <div class="section-tag">Design Context</div>
              <div class="field-grid">
                <div class="input-item">
                  <label>参与者数</label>
                  <input v-model.number="form.design.actors" type="number" />
                </div>
                <div class="input-item">
                  <label>用例数</label>
                  <input v-model.number="form.design.useCases" type="number" />
                </div>
                <div class="input-item">
                  <label>类图类数</label>
                  <input v-model.number="form.design.classes" type="number" />
                </div>
                <div class="input-item">
                  <label>子类数</label>
                  <input v-model.number="form.design.subclasses" type="number" />
                </div>
                <div class="input-item">
                  <label>判定数</label>
                  <input v-model.number="form.design.decisions" type="number" />
                </div>
                <div class="input-item">
                  <label>事务数</label>
                  <input v-model.number="form.design.transactions" type="number" />
                </div>
              </div>
            </div>

            <div class="config-section">
              <div class="section-tag">Estimation Params</div>
              <div class="field-grid compact">
                <div class="input-item">
                  <label>团队人数</label>
                  <input v-model.number="form.estimate.teamMembers" type="number" />
                </div>
                <div class="input-item">
                  <label>人月成本</label>
                  <input v-model.number="form.estimate.monthlyRate" type="number" />
                </div>
                <div class="input-item">
                  <label>生产率系数</label>
                  <input v-model.number="form.estimate.productivity" type="number" step="0.1" />
                </div>
              </div>
            </div>

            <div class="config-section no-border">
              <div v-if="inputMode === 'code'">
                <div class="inner-head">
                  <span class="section-tag">Source Code</span>
                  <button class="clear-btn" @click="clearCode">CLEAR ALL</button>
                </div>
                <textarea v-model="form.code" class="tech-editor" placeholder="在此粘贴 Java 源码..."></textarea>
              </div>

              <div v-else>
                <div class="inner-head">
                  <span class="section-tag">Source Files</span>
                  <button class="clear-btn" @click="clearFiles">CLEAR ALL</button>
                </div>
                
                <div class="file-upload-wrapper">
                  <input type="file" multiple @change="handleFileChange" accept=".java" id="f-upload" class="hidden-input" />
                  <label for="f-upload" class="upload-dropzone">
                    <div class="upload-icon">
                      <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="#2EC4B6" stroke-width="2">
                        <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M17 8l-5-5-5 5M12 3v12"/>
                      </svg>
                    </div>
                    <span class="upload-text">点击或拖拽 Java 文件至此</span>
                    <span class="upload-hint">支持多选，仅限 .java 格式</span>
                  </label>
                </div>

                <div class="file-tokens-container" v-if="selectedFiles.length">
                  <div v-for="f in selectedFiles" :key="f.name" class="file-token">
                    <span class="f-icon">📄</span>
                    <span class="f-name">{{ f.name }}</span>
                    <span class="f-size">{{ formatFileSize(f.size) }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <button class="run-btn" :disabled="loading" @click="analyzeAll">
            <span v-if="!loading">RUN ANALYSIS</span>
            <span v-else class="loader-text">SCANNING SYSTEM...</span>
          </button>
          <p v-if="errorMessage" class="error-log">{{ errorMessage }}</p>
        </div>
      </aside>

      <main class="dashboard">
        <div v-if="!result" class="idle-state">
          <div class="radar"></div>
          <p>等待系统扫描并输出度量报告</p>
        </div>

        <template v-else>
          <div class="kpi-row">
            <div class="kpi-card cyan">
              <span class="label">工作量 (Effort)</span>
              <div class="value">{{ result.estimation.Effort }}</div>
              <small>Person-Hours</small>
            </div>
            <div class="kpi-card blue">
              <span class="label">开发周期 (Time)</span>
              <div class="value">{{ result.estimation.Time }}</div>
              <small>Months</small>
            </div>
            <div class="kpi-card amber">
              <span class="label">项目成本 (Cost)</span>
              <div class="value">{{ result.estimation.Cost }}</div>
              <small>Total CNY</small>
            </div>
            <div class="kpi-card coral">
              <span class="label">人力配置 (People)</span>
              <div class="value">{{ result.estimation.People }}</div>
              <small>Config: {{ result.estimation.ConfiguredPeople }}</small>
            </div>
          </div>

          <div class="metrics-bar">
            <div class="stat"><span>代码行数</span><strong>{{ result.overview.loc }}</strong></div>
            <div class="stat"><span>类数量</span><strong>{{ result.overview.classes }}</strong></div>
            <div class="stat"><span>方法数量</span><strong>{{ result.overview.methods }}</strong></div>
            <div class="stat"><span>平均复杂度</span><strong>{{ result.traditionalMetrics.AverageComplexity }}</strong></div>
          </div>

          <div class="detail-grid">
            <div class="detail-box">
              <h3>CK 面向对象模型度量</h3>
              <div class="metric-list">
                <div v-for="item in ckRows" :key="item.key" class="metric-item">
                  <div class="m-head">
                    <span class="m-label">{{ item.label }}</span>
                    <span class="m-val">{{ result.ckMetrics[item.key] }}</span>
                  </div>
                  <p class="m-desc">{{ item.note }}</p>
                </div>
              </div>
            </div>

            <div class="detail-box">
              <h3>LK 与逻辑度量</h3>
              <div class="metric-list">
                <div v-for="item in lkRows" :key="item.key" class="metric-item">
                  <div class="m-head">
                    <span class="m-label">{{ item.label }}</span>
                    <span class="m-val">{{ result.lkMetrics[item.key] }}</span>
                  </div>
                  <p class="m-desc">{{ item.note }}</p>
                </div>
                <div class="metric-item highlight">
                  <div class="m-head">
                    <span class="m-label">圈复杂度</span>
                    <span class="m-val">{{ result.traditionalMetrics.CyclomaticComplexity }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

        </template>
      </main>

      <div v-if="result" class="class-section">
        <h3>类级明细 / Class Breakdown</h3>
        <div class="class-cards">
          <div v-for="item in result.classMetrics" :key="item.sourceFile + item.name" class="c-card">
            <div class="c-header">
              <strong>{{ item.name }}</strong>
              <span v-if="showSourceFile(item.sourceFile)" class="path">{{ item.sourceFile }}</span>
            </div>
            <div class="c-body">
              <div class="mini-stat"><span>WMC</span>{{ item.WMC }}</div>
              <div class="mini-stat"><span>CBO</span>{{ item.CBO }}</div>
              <div class="mini-stat"><span>RFC</span>{{ item.RFC }}</div>
              <div class="mini-stat"><span>CC</span>{{ item.complexity }}</div>
            </div>
          </div>
        </div>
      </div>
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

const showSourceFile = (sourceFile) => {
  return sourceFile && sourceFile !== 'InlineSnippet.java'
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

<style scoped>
.workspace { background-color: #0b1120; color: #f8fafc; min-height: 100vh; font-family: 'Segoe UI', system-ui, sans-serif; }

/* 标题区 */
.hero-band { 
  background: linear-gradient(135deg, #0f172a, #111827); 
  padding: 40px 60px; border-bottom: 1px solid #1e293b;
  display: flex; justify-content: space-between; align-items: center;
}
.eyebrow { color: #2ec4b6; font-weight: bold; letter-spacing: 3px; font-size: 11px; text-transform: uppercase; }
.hero-summary { display: flex; gap: 12px; }
.summary-chip { background: rgba(46, 196, 182, 0.1); border: 1px solid rgba(46, 196, 182, 0.2); padding: 6px 16px; border-radius: 20px; font-size: 12px; color: #2ec4b6; }

.main-grid { display: grid; grid-template-columns: 420px 1fr; gap: 30px; padding: 30px; }
.dashboard {
  display: flex;
  flex-direction: column;
  gap: 26px;
  min-width: 0;
}

/* 控制面板 */
.panel-shell { background: #1e293b; border-radius: 20px; padding: 26px; border: 1px solid #334155; box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.2); }
.panel-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 24px; }
.panel-head h2 { font-size: 14px; letter-spacing: 1px; color: #94a3b8; }

/* 按钮统一色调：#2ec4b6 */
.action-outline-btn {
  background: transparent; border: 1px solid #2ec4b6; color: #2ec4b6;
  padding: 6px 14px; border-radius: 8px; font-size: 12px; cursor: pointer;
  transition: all 0.3s; display: flex; align-items: center; gap: 6px;
}
.action-outline-btn:hover { background: rgba(46, 196, 182, 0.1); }

.mode-switch { display: flex; background: #0f172a; padding: 4px; border-radius: 12px; margin-bottom: 24px; }
.mode-btn { 
  flex: 1; padding: 10px; border: none; background: transparent; color: #64748b; 
  cursor: pointer; border-radius: 8px; font-weight: 600; transition: all 0.2s;
}
.mode-btn.active { background: #2ec4b6; color: #0b1120; font-weight: 800; }

.section-tag { color: #2ec4b6; font-size: 11px; font-weight: bold; margin-bottom: 16px; text-transform: uppercase; border-left: 2px solid #2ec4b6; padding-left: 8px; }

/* 输入控件 */
.field-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 16px; margin-bottom: 24px; }
.input-item label { display: block; font-size: 12px; color: #94a3b8; margin-bottom: 6px; }
.input-item input { 
  width: 100%; background: #0f172a; border: 1px solid #334155; 
  padding: 10px 14px; border-radius: 8px; color: white; transition: border-color 0.3s;
}
.input-item input:focus { border-color: #2ec4b6; outline: none; }

/* 文件上传美化 */
.file-upload-wrapper { margin-bottom: 20px; }
.hidden-input { display: none; }
.upload-dropzone {
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  padding: 30px; border: 2px dashed #334155; border-radius: 16px; cursor: pointer;
  background: rgba(15, 23, 42, 0.5); transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.upload-dropzone:hover { border-color: #2ec4b6; background: rgba(46, 196, 182, 0.05); box-shadow: 0 0 15px rgba(46, 196, 182, 0.1); }
.upload-icon { margin-bottom: 12px; }
.upload-text { color: #f8fafc; font-size: 14px; font-weight: 600; margin-bottom: 4px; }
.upload-hint { color: #64748b; font-size: 11px; }

/* 文件列表 Token */
.file-tokens-container { display: flex; flex-direction: column; gap: 8px; margin-top: 16px; max-height: 150px; overflow-y: auto; padding-right: 4px; }
.file-token {
  display: flex; align-items: center; background: #0f172a; 
  padding: 8px 12px; border-radius: 8px; border: 1px solid #334155;
}
.f-icon { margin-right: 10px; font-size: 14px; }
.f-name { flex: 1; font-size: 12px; color: #e2e8f0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.f-size { font-size: 10px; color: #64748b; font-family: monospace; }

/* 代码编辑器及清除按钮 */
.inner-head { display: flex; justify-content: space-between; align-items: center; }
.clear-btn { background: transparent; border: none; color: #ef4444; font-size: 10px; font-weight: bold; cursor: pointer; padding: 4px 8px; }
.tech-editor { 
  width: 100%; height: 220px; background: #0f172a; border: 1px solid #334155;
  border-radius: 12px; color: #2ec4b6; font-family: 'Fira Code', monospace; padding: 16px;
}

/* 主按钮 */
.run-btn { 
  width: 100%; padding: 18px; background: #2ec4b6; color: #0b1120; 
  border: none; border-radius: 14px; font-weight: 800; font-size: 14px; cursor: pointer; margin-top: 24px;
}
.run-btn:disabled { background: #334155; color: #64748b; }

/* KPI 与 结果区域 */
.kpi-row { display: grid; grid-template-columns: repeat(4, 1fr); gap: 18px; margin-bottom: 0; }
.kpi-card { padding: 22px 22px; border-radius: 18px; border: 1px solid rgba(255,255,255,0.05); }
.kpi-card.cyan { border-top: 4px solid #2ec4b6; background: linear-gradient(135deg, #064e3b, #0b1120); }
.kpi-card.blue { border-top: 4px solid #2ec4b6; background: linear-gradient(135deg, #064e3b, #0b1120); }
.kpi-card.amber { border-top: 4px solid #2ec4b6; background: linear-gradient(135deg, #064e3b, #0b1120); }
.kpi-card.coral { border-top: 4px solid #2ec4b6; background: linear-gradient(135deg, #064e3b, #0b1120); }

.kpi-card .label { font-size: 12px; }
.kpi-card .value { font-size: 29px; font-weight: 800; margin: 10px 0 8px; line-height: 1.1; }
.kpi-card small { font-size: 11px; color: #94a3b8; }

.metrics-bar {
  background: #1e293b;
  padding: 22px;
  border-radius: 16px;
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 14px;
  margin-bottom: 0;
  border: 1px solid #334155;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 7px;
  padding: 16px 16px;
  border-radius: 12px;
  background: rgba(15, 23, 42, 0.72);
  border: 1px solid rgba(51, 65, 85, 0.9);
}

.stat span {
  font-size: 12px;
  color: #94a3b8;
  letter-spacing: 0;
}

.stat strong {
  font-size: 23px;
  color: #2ec4b6;
  line-height: 1.1;
}

.detail-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 24px; }
.detail-box { background: #1e293b; padding: 26px; padding-bottom: 38px; border-radius: 18px; border: 1px solid #334155; }
.detail-box h3 { font-size: 15px; color: #f8fafc; margin-bottom: 22px; display: flex; align-items: center; gap: 10px; }
.detail-box h3::before { content: ''; width: 4px; height: 16px; background: #2ec4b6; border-radius: 2px; }

.metric-list {
  display: grid;
  gap: 28.5px;
}

.metric-item {
  padding: 16px 16px;
  border-radius: 12px;
  background: rgba(15, 23, 42, 0.72);
  border: 1px solid rgba(51, 65, 85, 0.9);
}

.metric-item.highlight {
  border-color: rgba(46, 196, 182, 0.45);
  box-shadow: inset 0 0 0 1px rgba(46, 196, 182, 0.12);
}

.m-head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 6px;
}

.m-label {
  font-size: 12px;
  color: #e2e8f0;
  font-weight: 600;
}

.m-val {
  color: #2ec4b6;
  font-size: 19px;
  font-weight: 800;
  min-width: 42px;
  text-align: right;
}

.m-desc {
  margin: 0;
  color: #94a3b8;
  font-size: 11px;
  line-height: 1.55;
}

.class-section {
  grid-column: 1 / -1;
  background: #1e293b;
  padding: 30px;
  border-radius: 20px;
  border: 1px solid #334155;
}

.class-section h3 {
  font-size: 15px;
  color: #f8fafc;
  margin-bottom: 24px;
  display: flex;
  align-items: center;
  gap: 10px;
}

.class-section h3::before {
  content: '';
  width: 4px;
  height: 16px;
  background: #2ec4b6;
  border-radius: 2px;
}

.class-cards {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
  align-items: stretch;
}

.c-card {
  background: #0f172a;
  padding: 20px;
  border-radius: 16px;
  border: 1px solid #334155;
  min-height: 150px;
  width: 100%;
}

.c-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  margin-bottom: 14px;
}

.c-header strong { color: #2ec4b6; }

.path {
  color: #94a3b8;
  font-size: 12px;
}

.c-body {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
}

.mini-stat {
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 10px 12px;
  border-radius: 12px;
  background: rgba(30, 41, 59, 0.78);
  color: #f8fafc;
  text-align: center;
  font-weight: 700;
}

.mini-stat span {
  color: #94a3b8;
  font-size: 11px;
  font-weight: 500;
}
</style>
