<template>
  <div class="app-container">
    <header class="header">
      <div class="header-content">
        <div class="logo">
          <span class="logo-icon">✨</span>
          <h1>Simpel Code</h1>
        </div>
        <p class="subtitle">Playground - พิมพ์ปุ๊บ แปลงปั๊บ รันทันที</p>
      </div>
      <div class="header-status">
        <span v-if="isSaved" class="save-status">Auto-saved</span>
      </div>
    </header>

    <div class="content-wrapper">
      
      <aside class="sidebar">
        <div class="sidebar-header">
          <h3>ตัวอย่างโค้ด</h3>
          <p>คลิกเพื่อลองรันคำสั่ง</p>
        </div>
        <div class="example-list">
          <button 
            v-for="(example, index) in codeExamples" 
            :key="index"
            class="example-btn"
            @click="loadExample(example.code)"
          >
            {{ example.title }}
          </button>
        </div>
      </aside>

      <main class="playground-container">
        
        <div class="pane editor-pane">
          <div class="pane-header">
            <div class="pane-title">
              <span class="dot simpel-dot"></span>
              <h2>1. Simpel Code</h2>
            </div>
            <div class="pane-actions">
              <button class="action-btn save-btn" @click="downloadCode">Save .simpel</button>
              <button class="action-btn clear-btn" @click="simpelCode = ''">Clear</button>
            </div>
          </div>
          <textarea 
            v-model="simpelCode" 
            placeholder="พิมพ์คำสั่งที่นี่ เช่น&#10;add 5 and 3&#10;show result"
            spellcheck="false"
            class="code-input"
          ></textarea>
        </div>

        <div class="pane python-pane">
          <div class="pane-header">
            <div class="pane-title">
              <span class="dot python-dot"></span>
              <h2>2. Python</h2>
            </div>
            <div class="pane-actions">
              <span class="badge">Reference</span>
              <button class="action-btn copy-btn" @click="copyCode(pythonTranslation)">Copy</button>
            </div>
          </div>
          <div class="code-output python-bg">
            <pre>{{ pythonTranslation || '# รอรับคำสั่ง...' }}</pre>
          </div>
        </div>

        <div class="pane terminal-pane">
          <div class="pane-header">
            <div class="pane-title">
              <span class="dot terminal-dot"></span>
              <h2>3. Output</h2>
            </div>
            <div class="pane-actions">
              <span class="badge run-badge">Live</span>
              <button class="action-btn copy-btn" @click="copyCode(outputResult.map(o => o.text).join('\n'))">Copy</button>
            </div>
          </div>
          <div class="code-output terminal-bg">
            <template v-if="outputResult.length > 0">
              <pre v-for="(line, index) in outputResult" :key="index" :class="['terminal-line', { 'error-text': line.isError, 'comment-text': line.isComment }]">
                <span class="prompt" v-if="!line.isError && !line.isComment">❯</span>
                <span class="prompt error-prompt" v-else-if="line.isError">✖</span>
                <span class="prompt comment-prompt" v-else>#</span> {{ line.text }}
              </pre>
            </template>
            <template v-else>
              <pre class="terminal-placeholder">รอแสดงผลลัพธ์...</pre>
            </template>
          </div>
        </div>

      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const isSaved = ref(false)

// ข้อมูลตัวอย่างโค้ดทั้งหมด (เอาอิโมจิออก)
const codeExamples = [
  {
    title: "แสดงข้อความ",
    code: `show "Hello World"\nshow "Welcome to Simpel Code"`
  },
  {
    title: "คำนวณ (ตัวเลขและตัวแปร)",
    code: `note ลองคำนวณคะแนนรวมดูนะ\nset myScore to 85\nset friendScore to 90\n\nadd myScore and friendScore\nshow "คะแนนรวม: " + result`
  },
  {
    title: "สร้างตัวแปร",
    code: `set myName to "Nathida"\nshow "Hello " + myName`
  },
  {
    title: "เงื่อนไข (If-Else)",
    code: `// เช็คผลสอบ\nset score to 45\nif score >= 50 show "You Pass!" else show "You Fail!"`
  },
  {
    title: "เทียบข้อความ",
    code: `set role to "admin"\nif role == "admin" show "Welcome Boss!" else show "Access Denied"`
  },
  {
    title: "การวนลูป (Repeat)",
    code: `repeat 3 times show "Hooray!"`
  }
]

// โค้ดเริ่มต้น
const simpelCode = ref(codeExamples[1].code)

onMounted(() => {
  const savedCode = localStorage.getItem('simpelCode_autosave')
  if (savedCode) {
    simpelCode.value = savedCode
  }
})

watch(simpelCode, (newCode) => {
  localStorage.setItem('simpelCode_autosave', newCode)
  isSaved.value = true
  setTimeout(() => isSaved.value = false, 2000)
})

const loadExample = (code) => { simpelCode.value = code }

const copyCode = async (text) => {
  if (!text) return
  await navigator.clipboard.writeText(text)
  alert('คัดลอกลง Clipboard เรียบร้อยแล้ว!')
}

// ฟังก์ชัน Download Code เป็นไฟล์ .simpel
const downloadCode = () => {
  const blob = new Blob([simpelCode.value], { type: 'text/plain' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'main.simpel'
  a.click()
  URL.revokeObjectURL(url)
}

// ฟังก์ชันช่วยประมวลผล String/ตัวแปร
const evaluateValue = (valStr, variables) => {
  if (valStr.startsWith('"') && valStr.endsWith('"')) {
    return valStr.replace(/"/g, '')
  } else if (!isNaN(valStr)) {
    return parseFloat(valStr)
  } else {
    return variables[valStr]
  }
}

const evaluateShow = (target, variables, outputLines) => {
  if (target.includes('+')) {
    const parts = target.split('+').map(p => p.trim())
    let combined = ''
    for (const part of parts) {
      const val = evaluateValue(part, variables)
      combined += val !== undefined ? val : `[Unknown: ${part}]`
    }
    outputLines.push({ text: combined, isError: false })
    return
  }
  
  const val = evaluateValue(target, variables)
  if (val !== undefined) {
    outputLines.push({ text: val, isError: false })
  } else {
    outputLines.push({ text: `Syntax Error: ไม่รู้จัก '${target}'`, isError: true })
  }
}

// ระบบแปลภาษาหลัก
const processCode = computed(() => {
  let pythonLines = []
  let outputLines = []
  let variables = {}
  
  const lines = simpelCode.value.split('\n')

  for (let i = 0; i < lines.length; i++) {
    let line = lines[i].trim()
    if (!line) {
      pythonLines.push('')
      continue
    }

    // 0: การคอมเมนต์ (note หรือ //)
    if (line.startsWith('//') || line.startsWith('note ')) {
      const commentText = line.replace('//', '').replace('note ', '').trim()
      pythonLines.push(`# ${commentText}`)
      outputLines.push({ text: `[Comment] ${commentText}`, isComment: true })
      continue
    }

    // 1: การบวก
    const addMatch = line.match(/^add\s+([a-zA-Z0-9_]+)\s+and\s+([a-zA-Z0-9_]+)$/)
    if (addMatch) {
      const v1 = evaluateValue(addMatch[1], variables)
      const v2 = evaluateValue(addMatch[2], variables)
      if(v1 !== undefined && v2 !== undefined) {
        variables['result'] = Number(v1) + Number(v2)
        pythonLines.push(`result = ${addMatch[1]} + ${addMatch[2]}`)
      } else {
        outputLines.push({ text: `Error: ตัวแปรบวกไม่ได้ค่าที่สมบูรณ์`, isError: true })
      }
      continue
    }

    // 2: การลบ
    const subMatch = line.match(/^subtract\s+([a-zA-Z0-9_]+)\s+from\s+([a-zA-Z0-9_]+)$/)
    if (subMatch) {
      const v1 = evaluateValue(subMatch[1], variables) // ตัวลบ
      const v2 = evaluateValue(subMatch[2], variables) // ตัวตั้ง
      if(v1 !== undefined && v2 !== undefined) {
        variables['result'] = Number(v2) - Number(v1)
        pythonLines.push(`result = ${subMatch[2]} - ${subMatch[1]}`)
      }
      continue
    }

    // 3: ตั้งค่าตัวแปร
    const setMatch = line.match(/^set\s+([a-zA-Z_]\w*)\s+to\s+(.+)$/)
    if (setMatch) {
      const varName = setMatch[1]
      const varValue = setMatch[2]
      pythonLines.push(`${varName} = ${varValue}`)
      variables[varName] = evaluateValue(varValue, variables)
      continue
    }

    // 4: แสดงผล
    const showMatch = line.match(/^show\s+(.+)$/)
    if (showMatch) {
      pythonLines.push(`print(${showMatch[1]})`)
      evaluateShow(showMatch[1], variables, outputLines)
      continue
    }

    // 5: เงื่อนไข If-Else
    const ifMatch = line.match(/^if\s+([a-zA-Z0-9_]+)\s*(>|<|==|>=|<=)\s*("[^"]+"|[^ ]+)\s+show\s+(.+)$/)
    if (ifMatch) {
      const leftName = ifMatch[1]
      const operator = ifMatch[2]
      const rightName = ifMatch[3]
      let targetInfo = ifMatch[4]
      
      let targetIfTrue = targetInfo
      let targetIfFalse = null

      if (targetInfo.includes(' else show ')) {
        const parts = targetInfo.split(' else show ')
        targetIfTrue = parts[0]
        targetIfFalse = parts[1]
      }

      const leftVal = evaluateValue(leftName, variables)
      const rightVal = evaluateValue(rightName, variables)

      pythonLines.push(`if ${leftName} ${operator} ${rightName}:`)
      pythonLines.push(`    print(${targetIfTrue})`)
      if (targetIfFalse) {
        pythonLines.push(`else:`)
        pythonLines.push(`    print(${targetIfFalse})`)
      }

      let isTrue = false
      if (operator === '>' && leftVal > rightVal) isTrue = true
      else if (operator === '<' && leftVal < rightVal) isTrue = true
      else if (operator === '==' && leftVal == rightVal) isTrue = true
      else if (operator === '>=' && leftVal >= rightVal) isTrue = true
      else if (operator === '<=' && leftVal <= rightVal) isTrue = true

      if (isTrue) {
        evaluateShow(targetIfTrue, variables, outputLines)
      } else if (targetIfFalse) {
        evaluateShow(targetIfFalse, variables, outputLines)
      }
      continue
    }

    // 6: การวนลูป
    const repeatMatch = line.match(/^repeat\s+(\d+)\s+times\s+show\s+(.+)$/)
    if (repeatMatch) {
      const times = parseInt(repeatMatch[1])
      const target = repeatMatch[2]

      pythonLines.push(`for i in range(${times}):`)
      pythonLines.push(`    print(${target})`)

      for (let j = 0; j < times; j++) {
        evaluateShow(target, variables, outputLines)
      }
      continue
    }

    pythonLines.push(`# Syntax Error: ${line}`)
    outputLines.push({ text: `Syntax Error บรรทัดที่ ${i + 1}: ไม่เข้าใจคำสั่ง '${line}'`, isError: true })
  }

  return { python: pythonLines.join('\n'), output: outputLines }
})

const pythonTranslation = computed(() => processCode.value.python)
const outputResult = computed(() => processCode.value.output)
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500&family=Inter:wght@400;500;600;700&display=swap');

* { box-sizing: border-box; }

.app-container {
  min-height: 100vh;
  background-color: #f3f4f6;
  font-family: 'Inter', sans-serif;
  display: flex;
  flex-direction: column;
}

.header {
  background-color: #ffffff;
  box-shadow: 0 1px 3px rgba(0,0,0,0.05);
  padding: 1rem 2rem;
  z-index: 10;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header-content { display: flex; align-items: center; gap: 20px; }
.logo { display: flex; align-items: center; gap: 10px; }
.logo-icon { font-size: 1.8rem; }
.header h1 { margin: 0; color: #111827; font-size: 1.5rem; font-weight: 700; }
.subtitle { margin: 0; color: #6b7280; font-size: 0.95rem; border-left: 2px solid #e5e7eb; padding-left: 15px; }

.save-status {
  font-size: 0.85rem;
  color: #10b981;
  background-color: #d1fae5;
  padding: 4px 10px;
  border-radius: 20px;
  font-weight: 500;
  animation: fadeIn 0.3s;
}

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

.content-wrapper {
  display: flex;
  flex: 1;
  max-width: 1600px;
  margin: 0 auto;
  width: 100%;
  overflow: hidden;
}

.sidebar {
  width: 260px;
  background-color: #ffffff;
  border-right: 1px solid #e5e7eb;
  padding: 20px 15px;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.sidebar-header h3 { margin: 0 0 5px 0; font-size: 1rem; color: #374151; }
.sidebar-header p { margin: 0; font-size: 0.8rem; color: #9ca3af; }

.example-list { display: flex; flex-direction: column; gap: 10px; }

.example-btn {
  background-color: #f9fafb;
  border: 1px solid #e5e7eb;
  padding: 12px 15px;
  border-radius: 8px;
  text-align: left;
  font-size: 0.9rem;
  font-weight: 500;
  color: #4b5563;
  cursor: pointer;
  transition: all 0.2s ease;
}

.example-btn:hover {
  background-color: #eff6ff;
  border-color: #bfdbfe;
  color: #1d4ed8;
  transform: translateY(-1px);
}

.playground-container {
  display: flex;
  gap: 20px;
  padding: 20px;
  flex: 1;
  overflow: hidden;
}

.pane {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
  overflow: hidden;
  border: 1px solid #e5e7eb;
}

.pane-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background-color: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
}

.pane-title { display: flex; align-items: center; gap: 8px; }
.dot { width: 10px; height: 10px; border-radius: 50%; }
.simpel-dot { background-color: #3b82f6; }
.python-dot { background-color: #f59e0b; }
.terminal-dot { background-color: #10b981; }

.pane-header h2 { margin: 0; font-size: 0.95rem; color: #374151; font-weight: 600; }

.pane-actions { display: flex; align-items: center; gap: 10px; }
.badge { font-size: 0.75rem; padding: 2px 8px; border-radius: 12px; background-color: #e5e7eb; color: #4b5563; font-weight: 500; }
.run-badge { background-color: #d1fae5; color: #065f46; }

.action-btn {
  background: none;
  border: 1px solid transparent;
  font-size: 0.8rem;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
  font-weight: 500;
  transition: all 0.2s;
}

.clear-btn { color: #ef4444; }
.clear-btn:hover { background-color: #fee2e2; border-color: #fca5a5; }

.copy-btn { color: #6b7280; }
.copy-btn:hover { background-color: #e5e7eb; color: #374151; }

.save-btn { color: #3b82f6; border-color: #bfdbfe; background-color: #eff6ff; }
.save-btn:hover { background-color: #dbeafe; }

.code-input, .code-output {
  flex: 1;
  padding: 20px;
  font-family: 'Fira Code', monospace;
  font-size: 0.95rem;
  line-height: 1.6;
  border: none;
  resize: none;
  outline: none;
  margin: 0;
  overflow-y: auto;
}

.code-input { background-color: #ffffff; color: #1f2937; }
.code-input::placeholder { color: #9ca3af; }

.python-bg { background-color: #282c34; color: #abb2bf; }
.python-bg pre { margin: 0; white-space: pre-wrap; }

.terminal-bg { background-color: #0f172a; color: #e2e8f0; }
.terminal-line { margin: 0 0 4px 0; white-space: pre-wrap; display: flex; }
.prompt { color: #10b981; margin-right: 8px; font-weight: bold; }
.error-prompt { color: #ef4444; }
.comment-prompt { color: #64748b; font-weight: normal; }
.error-text { color: #fca5a5; } 
.comment-text { color: #94a3b8; font-style: italic; } 
.terminal-placeholder { color: #475569; font-style: italic; margin: 0; }

::-webkit-scrollbar { width: 8px; height: 8px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
.python-bg::-webkit-scrollbar-thumb { background: #4b5563; }
.terminal-bg::-webkit-scrollbar-thumb { background: #334155; }

@media (max-width: 1024px) {
  .header-content { flex-direction: column; align-items: flex-start; gap: 5px; }
  .content-wrapper { flex-direction: column; }
  .sidebar { width: 100%; border-right: none; border-bottom: 1px solid #e5e7eb; flex-direction: row; flex-wrap: wrap; }
  .example-list { flex-direction: row; flex-wrap: wrap; }
  .playground-container { flex-direction: column; height: auto; }
  .pane { min-height: 300px; }
}
</style>