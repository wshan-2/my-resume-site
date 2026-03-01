<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const isEntered = ref(false)
const activeModal = ref(null)

// --- 基础逻辑 ---
const enterSite = () => { isEntered.value = true }
const openModal = (section) => { activeModal.value = section }
const closeModal = () => { activeModal.value = null }

// --- 特效 1：打字机 ---
const typedText = ref('')
const fullText = 'Hi, 我是文思涵'

// --- 特效 2：Konami 极客彩蛋 (w e n s i h) ---
const devMode = ref(false)
let keyBuffer = ''
const secretCode = 'wensih'

const handleKeydown = (e) => {
  if (!isEntered.value) return
  keyBuffer += e.key.toLowerCase()
  if (keyBuffer.length > secretCode.length) {
    keyBuffer = keyBuffer.slice(-secretCode.length)
  }
  if (keyBuffer === secretCode) {
    devMode.value = !devMode.value
    console.log(`%c🎮 开发者模式已${devMode.value ? '激活！(UI网格已显示)' : '退出'}`, 'color: #ff8fab; font-size: 16px; font-weight: bold;')
  }
}

// --- 特效 3：名片区解压泡泡纸 ---
const bubbles = ref([])
let bubbleId = 0
const createBubble = (e) => {
  if (bubbles.value.length > 8) bubbles.value.shift()
  const rect = e.currentTarget.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const id = bubbleId++
  bubbles.value.push({ id, x, y })
  setTimeout(() => {
    bubbles.value = bubbles.value.filter(b => b.id !== id)
  }, 800)
}

// --- 特效 4：技能标签的“磁力场”悬浮计算 ---
const skillsCardRef = ref(null)
const mousePos = ref({ x: -1000, y: -1000 })

const handleSkillsMove = (e) => {
  if (!skillsCardRef.value) return
  const rect = skillsCardRef.value.getBoundingClientRect()
  mousePos.value = { x: e.clientX - rect.left, y: e.clientY - rect.top }
}
const handleSkillsLeave = () => {
  mousePos.value = { x: -1000, y: -1000 }
}

const getMagneticStyle = (elX, elY) => {
  const dx = mousePos.value.x - elX
  const dy = mousePos.value.y - elY
  const distance = Math.sqrt(dx * dx + dy * dy)
  if (distance < 120 && distance > 0) {
    const pull = (120 - distance) / 10 
    return { transform: `translate(${dx > 0 ? pull : -pull}px, ${dy > 0 ? pull : -pull}px) scale(1.05)` }
  }
  return { transform: 'translate(0px, 0px) scale(1)' }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
  let i = 0
  const typing = setInterval(() => {
    if (i < fullText.length) {
      typedText.value += fullText.charAt(i)
      i++
    } else {
      clearInterval(typing)
    }
  }, 150)

  console.log(
    '%c🚀 欢迎来到文思涵的代码世界！\n%c虽然我还在成长，但我对代码有 100% 的敬畏心！\n💡 试试在页面键盘输入: wensih',
    'color: #ff8fab; font-size: 22px; font-weight: bold; padding: 10px 0; text-shadow: 1px 1px 2px rgba(0,0,0,0.1);',
    'color: #2d3436; font-size: 14px; padding: 5px 0; line-height: 1.6;'
  )
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <div class="app-container" :class="{ 'dev-mode': devMode }">
    <Transition name="fade">
      <div v-if="!isEntered" class="welcome-screen">
        <div class="welcome-box">
          <div class="floating-avatar">🌷</div>
          <h1 class="typewriter-text">{{ typedText }}<span class="cursor">|</span></h1>
          <p class="subtitle">欢迎来到我的前端灵感空间</p>
          <button @click="enterSite" class="magic-btn">✨ 点击开启探索</button>
        </div>
      </div>
    </Transition>

    <Transition name="slide-up">
      <div v-if="isEntered" class="dashboard-wrapper">
        <div class="bento-layout">
          
          <div class="bento-card profile-bento" @click="createBubble" @mousemove="createBubble">
            <div 
              v-for="bubble in bubbles" 
              :key="bubble.id" 
              class="pop-bubble" 
              :style="{ left: bubble.x + 'px', top: bubble.y + 'px' }"
            ></div>

            <div class="photo-placeholder" style="padding: 0; border: none; background: transparent;">
              <img src="./assets/avatar.jpg" alt="文思涵" class="profile-photo" />
            </div>
            
            <div class="profile-text">
              <h1 class="name-title">文思涵 <span class="status-badge">随时入职</span></h1>
              <p class="school-text">上海立信会计金融学院 <br> 计算机科学与技术 (GPA 3.53 前15%)</p>
              <div class="contact-box">
                <p>📞 17702336322</p>
                <p>✉️ 3088263065@qq.com</p>
                <p>📍 重庆市</p>
              </div>
              <p class="bio-text">"拒绝做单纯的 API 搬运工。能死磕 Canvas 60FPS 底层优化，也能用 AI 赋能业务闭环的硬核前端女生。"</p>
            </div>
            <div v-if="devMode" class="dev-badge">🔧 Dev Mode Active</div>
          </div>

          <div class="bento-card project-bento highlight" @click="openModal('projects')">
            <div class="card-header">
              <h3>🚀 核心实战经验</h3>
              <a href="https://data.w-s-h.top" target="_blank" class="direct-link" @click.stop>🔗 看板 Demo</a>
            </div>
            
            <div class="project-details">
              <div class="project-item">
                <h4>1. Creator Data Dashboard (创作者数据看板)</h4>
                <p>独立开发 <strong>Vue 3 + ECharts</strong> 商业级看板。死磕事件循环，巧妙用 <code>nextTick</code> 绝杀图表异步加载白屏 Bug；并独立跑通 Vercel 全链路自动化部署。</p>
              </div>
              <div class="project-item">
                <h4>2. 智缮口袋 (AI膳食小程序)</h4>
                <p>为解决性能瓶颈，果断弃用臃肿图表库，手搓 <strong>Canvas 原生 API</strong> 绘制食材动态雷达图，在低端安卓机上硬核实现 <strong>60FPS 满帧交互</strong>。</p>
              </div>
            </div>

            <div class="tags-row">
              <span class="tag">Vue 3 组合式 API</span>
              <span class="tag">ECharts</span>
              <span class="tag">CI/CD 部署</span>
              <span class="tag">性能优化</span>
            </div>
            <div class="click-hint">深挖项目代码与业务逻辑 ↗</div>
          </div>

          <div 
            class="bento-card skills-bento" 
            ref="skillsCardRef"
            @mousemove="handleSkillsMove"
            @mouseleave="handleSkillsLeave"
            @click="openModal('skills')"
          >
            <h3>🛠️ 核心技术栈</h3>
            <p class="skill-intro">坚持“先筑基、后赋能”，注重底层原理：</p>
            <div class="skill-bubbles">
              <span class="bubble magnetic-tag" :style="getMagneticStyle(60, 40)">HTML5/CSS3</span>
              <span class="bubble magnetic-tag" :style="getMagneticStyle(180, 40)">Vue 3 全家桶</span>
              <span class="bubble magnetic-tag" :style="getMagneticStyle(80, 80)">JS/ES6 机制</span>
              <span class="bubble magnetic-tag" :style="getMagneticStyle(190, 80)">小程序原生</span>
              <span class="bubble magnetic-tag" :style="getMagneticStyle(100, 120)">Python自动化</span>
            </div>
            <div class="click-hint">查看底层技术清单 ↗</div>
          </div>

          <div class="bento-card exp-bento" @click="openModal('experience')">
            <h3>💼 履历 & 🏆 高光</h3>
            <div class="exp-content">
              <div class="exp-row">
                <strong>沧州埃汉科技 (前端实习)</strong>
                <span>深度参与 ECharts 数据看板开发，实现节点异常分钟级自动告警。</span>
              </div>
              <div class="exp-row">
                <strong>中安电子科技 (AI标注)</strong>
                <span>打破纯手动局限，编写 Python 脚本测试并优化底层软件逻辑。</span>
              </div>
              <div class="exp-row award-row">
                <strong>🔥 互联网+创新创业大赛</strong>
                <span class="red-text">全国一等奖 (核心荣誉)</span>
              </div>
              <div class="exp-row award-row">
                <strong>🏅 国家级大创项目</strong>
                <span>核心成员 (国家级铜奖)</span>
              </div>
            </div>
            <div class="click-hint">查看趣味详情与证书扫描件 ↗</div>
          </div>

        </div>
      </div>
    </Transition>

    <Transition name="pop">
      <div v-if="activeModal" class="modal-overlay" @click.self="closeModal">
        <div class="modal-content">
          <button class="close-btn" @click="closeModal">×</button>
          
          <div v-if="activeModal === 'projects'" class="modal-body scrollable">
             <h2 class="modal-title">🚀 核心项目实战深挖</h2>
             <div class="detail-block">
               <h3>1. Creator Data Dashboard</h3>
               <p><strong>痛点：</strong>父组件 API 异步请求与 ECharts 实例初始化存在时序冲突，导致页面频发白屏。<br><strong>解决：</strong>深入剖析 Vue3 响应式生命周期，将图表渲染逻辑精准挂载至 nextTick 宏任务中，实现丝滑渲染。具备极强的 ROI 意识，独立完成从组件化架构到 Vercel 部署的端到端流程。</p>
             </div>
             <div class="detail-block">
               <h3>2. 智缮口袋 - AI膳食管理小程序</h3>
               <p><strong>贡献：</strong>前端核心开发者。发现第三方图表库导致下沉市场旧手机卡顿后，果断直接调度 Canvas 原生 API 绘制食材新鲜度雷达图。通过精简重绘逻辑，成功拯救低端机性能，维持 60FPS 流畅体验。</p>
             </div>
             <div class="detail-block">
               <h3>3. 互惠互助养宠新模式</h3>
               <p><strong>贡献：</strong>针对平台内极度复杂的领养与遛狗申请表单，采用“高内聚、低耦合”设计模式，独立封装出高度通用的表单组件，让前端代码复用率飙升 30%。</p>
             </div>
          </div>

          <div v-if="activeModal === 'skills'" class="modal-body scrollable">
            <h2 class="modal-title">🛠️ 硬核技能清单</h2>
            <div class="detail-block"><h3>前端基石</h3><p>熟练 HTML5/CSS3，深刻理解闭包、原型链、事件循环机制及 ES6+ 语法（Promise / async-await）。</p></div>
            <div class="detail-block"><h3>现代框架</h3><p>熟悉 Vue 3 (Composition API)，理解 Proxy 响应式底层原理与虚拟 DOM Diff 算法。</p></div>
            <div class="detail-block"><h3>工程与拓展</h3><p>掌握 Git、Vite。具备小程序原生开发能力。能用 Python 编写自动化脚本“偷懒”，了解 MySQL 基础。</p></div>
          </div>

          <div v-if="activeModal === 'experience'" class="modal-body scrollable">
            <h2 class="modal-title">💼 职场印记与高光时刻</h2>
            <div class="detail-block">
              <p><strong>数据采集实习生 (沧州埃汉科技)</strong>：深度参与数据监控看板的前端开发。使用 ECharts 将海量节点健康状态实时可视化，实现了异常状态的分钟级自动告警，帮运维老哥光速排雷。</p>
              <p><strong>AI数据标注 (中安电子)</strong>：拒绝当无情的“点读机”，主动写 Python 脚本测试并优化底层图片标注软件逻辑，修复多处判定 Bug，保障投喂给 AI 的数据绝对纯净。</p>
              <p><strong>店铺运营 (宝可梦上海)</strong>：参与核心活动策划。在这里不仅练就了处理复杂数据报表的“火眼金睛”，还点满了跨部门极限拉扯的沟通技能树。</p>
            </div>
            
            <h2 class="modal-title" style="margin-top:25px;">🏆 核心荣誉墙与证书扫描件</h2>
            <div class="detail-block cert-container">
              <p class="cert-desc">🔥 "互联网+"创新创业大赛 <strong>全国一等奖</strong></p>
              <img src="./assets/cert1.jpg" alt="互联网+创新创业大赛一等奖" class="cert-img full-width" />
              
              <p class="cert-desc" style="margin-top: 15px;">💰 连续两年荣获 <strong>优秀学生奖学金 (三等奖)</strong></p>
              <div class="cert-gallery">
                <img src="./assets/cert2.jpg" alt="24-25学年奖学金" class="cert-img half-width" />
                <img src="./assets/cert3.jpg" alt="23-24学年奖学金" class="cert-img half-width" />
              </div>

              <p class="cert-desc" style="margin-top: 15px;">🏅 国家级大学生创新创业训练计划 <strong>铜奖</strong></p>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
/* ====== 基础锁定框架绝对不变 ====== */
:global(body), :global(html), :global(#app) {
  margin: 0 !important; padding: 0 !important; width: 100vw !important; height: 100vh !important; overflow: hidden !important; max-width: none !important;
}

.app-container { 
  position: fixed; inset: 0; font-family: 'PingFang SC', sans-serif; background-color: #fff0f5; color: #2d3436; 
  cursor: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='32' height='32' style='font-size:24px;'><text y='24'>✨</text></svg>") 16 16, auto;
  transition: background-color 0.3s;
}

.dev-mode { background-color: #f1f2f6; }
.dev-mode .bento-card { border: 1px dashed #ff8fab; box-shadow: none !important; background: rgba(255, 255, 255, 0.8); }
.dev-badge { position: absolute; top: 10px; right: 10px; background: #2d3436; color: #00ff00; font-family: monospace; font-size: 10px; padding: 2px 6px; border-radius: 4px; z-index: 10;}

button, a, .bento-card, .close-btn { cursor: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='32' height='32' style='font-size:24px;'><text y='24'>👆</text></svg>") 16 0, pointer !important; }

/* 🌸 欢迎屏 (100dvh 适配手机端) */
.welcome-screen { position: fixed; inset: 0; height: 100dvh; width: 100vw; background: linear-gradient(135deg, #ffe5ec, #ffc2d1); display: flex; justify-content: center; align-items: center; z-index: 100; }
.welcome-box { text-align: center; }
.floating-avatar { font-size: clamp(3rem, 8vh, 5rem); animation: float 3s ease-in-out infinite; margin-bottom: 2vh;}
.typewriter-text { font-size: clamp(2rem, 5vh, 3.5rem); color: white; margin-bottom: 1.5vh; font-weight: 900; letter-spacing: 2px; text-shadow: 2px 4px 10px rgba(255,143,171,0.4); }
.cursor { font-weight: 300; animation: blink 1s step-end infinite; }
@keyframes blink { 50% { opacity: 0; } }
.subtitle { font-size: clamp(1rem, 2vh, 1.3rem); color: white; margin-bottom: 4vh; font-weight: 500; }
.magic-btn { background: white; color: #ff8fab; border: none; padding: 2vh 3vw; border-radius: 40px; font-size: clamp(1rem, 2vh, 1.2rem); font-weight: bold; transition: all 0.3s; box-shadow: 0 10px 25px rgba(255,143,171,0.4); }
.magic-btn:hover { transform: translateY(-3px) scale(1.02); box-shadow: 0 15px 35px rgba(255,143,171,0.6); }

/* 🌸 主屏布局 */
.dashboard-wrapper { width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; padding: 3vh 3vw; box-sizing: border-box; }
.bento-layout { width: 100%; max-width: 1200px; height: 100%; display: grid; grid-template-columns: minmax(280px, 320px) 1fr 1fr; grid-template-rows: minmax(0, 1fr) minmax(0, 1fr); gap: 2vh; }

.bento-card { background: white; border-radius: 20px; padding: clamp(15px, 3vh, 25px); position: relative; display: flex; flex-direction: column; overflow: hidden; transition: all 0.3s ease; box-shadow: 0 4px 15px rgba(255,143,171,0.05); }
.bento-card:hover { transform: scale(1.01); box-shadow: 0 12px 30px rgba(255,143,171,0.2); border: 1px solid #ff8fab; z-index: 10; }

/* 🎈 解压泡泡纸动效 */
.profile-bento { grid-row: 1 / 3; grid-column: 1 / 2; cursor: crosshair !important; position: relative; display: flex; flex-direction: column; gap: 1.5vh;}
.pop-bubble {
  position: absolute; width: 40px; height: 40px; background: rgba(255, 143, 171, 0.4); border-radius: 50%;
  transform: translate(-50%, -50%) scale(0); pointer-events: none; z-index: 1;
  animation: popAnim 0.8s cubic-bezier(0.165, 0.84, 0.44, 1) forwards;
}
@keyframes popAnim { 0% { transform: translate(-50%, -50%) scale(0); opacity: 1; } 50% { transform: translate(-50%, -50%) scale(1.5); opacity: 0.5; } 100% { transform: translate(-50%, -50%) scale(2); opacity: 0; } }

/* 照片 */
.photo-placeholder { width: 100%; flex: 1 1 0; min-height: 120px; max-height: 35vh; border-radius: 16px; display: flex; justify-content: center; align-items: center; z-index: 2;}
.profile-photo { width: 100%; height: 100%; object-fit: cover; border-radius: 16px; box-shadow: 0 4px 15px rgba(255, 143, 171, 0.2); transition: transform 0.3s ease; }
.profile-photo:hover { transform: scale(1.03); }

.profile-text { display: flex; flex-direction: column; gap: 1.5vh; z-index: 2;}
.name-title { font-size: clamp(1.4rem, 3vh, 1.8rem); display: flex; align-items: center; justify-content: space-between; color: #2d3436; font-weight: 800; margin: 0; }
.status-badge { font-size: clamp(0.7rem, 1.2vh, 0.8rem); background: #ff8fab; color: white; padding: 4px 10px; border-radius: 20px; font-weight: bold;}
.school-text { font-size: clamp(0.85rem, 1.5vh, 0.95rem); color: #636e72; font-weight: bold; margin: 0;}
.contact-box { background: #f8f9fa; padding: clamp(10px, 2vh, 15px); border-radius: 12px; font-size: clamp(0.8rem, 1.4vh, 0.9rem); color: #454545; display: flex; flex-direction: column; gap: 0.5vh; border-left: 4px solid #ff8fab; margin: 0; }
.contact-box p { margin: 0; }
.bio-text { font-size: clamp(0.85rem, 1.5vh, 0.95rem); color: #fb6f92; font-style: italic; margin: 0; line-height: 1.5;}

/* 🚀 项目区 */
.project-bento { grid-row: 1 / 2; grid-column: 2 / 4; border: 2px solid #ff8fab; background: linear-gradient(135deg, #ffffff 0%, #fff0f5 100%); display: flex; flex-direction: column; gap: 1.5vh;}
.card-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.5vh;}
.project-bento h3 { font-size: clamp(1.2rem, 2.5vh, 1.5rem); color: #2d3436; font-weight: 800; margin: 0; }
.direct-link { background: #ff8fab; color: white; padding: 6px 15px; border-radius: 25px; text-decoration: none; font-size: clamp(0.8rem, 1.5vh, 0.9rem); font-weight: bold; transition: all 0.3s; z-index: 5;}
.direct-link:hover { background: #fb6f92; transform: scale(1.05); }

.project-details { display: flex; flex-direction: column; gap: 1.5vh; flex: 1; justify-content: center;}
.project-item h4 { font-size: clamp(0.95rem, 1.8vh, 1.1rem); color: #fb6f92; margin: 0 0 0.5vh 0; font-weight: bold;}
.project-item p { font-size: clamp(0.85rem, 1.6vh, 0.95rem); color: #4a4a4a; margin: 0; line-height: 1.6;}
.project-item strong { color: #ff8fab; }

.tags-row { display: flex; flex-wrap: wrap; gap: 8px; margin: 0; }
.tag { background: white; color: #ff8fab; padding: 4px 10px; border-radius: 10px; font-size: clamp(0.75rem, 1.2vh, 0.85rem); font-weight: bold; border: 1px solid #ff8fab; }

/* 🛠️ 技能区 */
.skills-bento { grid-row: 2 / 3; grid-column: 2 / 3; display: flex; flex-direction: column; gap: 1vh;}
.skills-bento h3 { font-size: clamp(1.1rem, 2.2vh, 1.4rem); color: #2d3436; font-weight: 800; margin: 0; }
.skill-intro { font-size: clamp(0.85rem, 1.5vh, 0.95rem); color: #636e72; margin: 0; }
.skill-bubbles { display: flex; flex-wrap: wrap; gap: 10px; align-content: flex-start; position: relative; flex: 1; padding-top: 1vh;}
.magnetic-tag { 
  background: #fff0f5; color: #fb6f92; padding: 8px 14px; border-radius: 20px; 
  font-size: clamp(0.8rem, 1.5vh, 0.9rem); font-weight: bold; 
  transition: transform 0.2s cubic-bezier(0.34, 1.56, 0.64, 1);
  display: inline-block;
}

/* 🏆 履历区 */
.exp-bento { grid-row: 2 / 3; grid-column: 3 / 4; display: flex; flex-direction: column; gap: 1.5vh;}
.exp-bento h3 { font-size: clamp(1.1rem, 2.2vh, 1.4rem); color: #2d3436; font-weight: 800; margin: 0; }
.exp-content { display: flex; flex-direction: column; gap: 1.5vh; flex: 1; justify-content: center;}
.exp-row { display: flex; flex-direction: column; gap: 0.3vh; position: relative; padding-left: 15px;}
.exp-row::before { content: "•"; position: absolute; left: 0; top: -2px; color: #ff8fab; font-size: 1.2rem;}
.exp-row strong { font-size: clamp(0.9rem, 1.6vh, 1rem); color: #2d3436; }
.exp-row span { font-size: clamp(0.8rem, 1.4vh, 0.9rem); color: #636e72; line-height: 1.4; }
.award-row::before { content: "✨"; font-size: 1rem; top: 0; left: -2px;}
.red-text { color: #ff8fab !important; font-weight: bold; }

/* 交互提示语 */
.click-hint { text-align: right; color: #ff8fab; font-size: clamp(0.75rem, 1.2vh, 0.85rem); font-weight: bold; opacity: 0; transition: opacity 0.3s; margin: 0; margin-top: auto;}
.bento-card:hover .click-hint { opacity: 1; }

/* 🌸 弹窗样式 */
.modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.5); backdrop-filter: blur(8px); display: flex; justify-content: center; align-items: center; z-index: 200; padding: 20px; }
.modal-content { background: white; width: 100%; max-width: 650px; max-height: 85vh; border-radius: 24px; padding: 40px; position: relative; display: flex; flex-direction: column; }
.close-btn { position: absolute; top: 20px; right: 25px; background: none; border: none; font-size: 2.5rem; color: #dfe6e9; cursor: pointer; transition: color 0.2s; line-height: 1; z-index: 10;}
.close-btn:hover { color: #ff8fab; }
.scrollable { overflow-y: auto; padding-right: 15px; }
.scrollable::-webkit-scrollbar { width: 6px; }
.scrollable::-webkit-scrollbar-thumb { background: #ff8fab; border-radius: 3px; }
.modal-title { color: #fb6f92; border-bottom: 2px dashed #fff0f5; padding-bottom: 15px; margin-bottom: 20px; font-size: 1.6rem; font-weight: 800; }
.detail-block { background: #fff0f5; padding: 20px; border-radius: 16px; margin-bottom: 15px; }
.detail-block h3 { color: #fb6f92; margin-bottom: 10px; font-size: 1.15rem; font-weight: bold; }
.detail-block p { color: #2d3436; line-height: 1.6; font-size: 1.05rem; margin-bottom: 8px; }

/* ====== 🎓 证书专属样式 ====== */
.cert-container { display: flex; flex-direction: column; align-items: center; }
.cert-desc { width: 100%; text-align: left; margin-bottom: 10px !important; }
.cert-desc strong { color: #ff8fab; }
.cert-img { border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); border: 1px solid #ffe5ec; transition: transform 0.3s; }
.cert-img:hover { transform: scale(1.02); }
.full-width { width: 100%; max-width: 500px; margin-bottom: 15px; }
.cert-gallery { display: flex; justify-content: space-between; gap: 15px; width: 100%; }
.half-width { width: calc(50% - 7.5px); object-fit: contain; }

/* 动画帧 */
@keyframes float { 0% { transform: translateY(0px); } 50% { transform: translateY(-10px); } 100% { transform: translateY(0px); } }
.fade-enter-active, .fade-leave-active { transition: opacity 0.5s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
.slide-up-enter-active { transition: all 0.6s cubic-bezier(0.16, 1, 0.3, 1); }
.slide-up-enter-from { opacity: 0; transform: translateY(30px); }
.pop-enter-active, .pop-leave-active { transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
.pop-enter-from, .pop-leave-to { opacity: 0; transform: scale(0.95); }

/* 📱 手机端适配：彻底解除高度封印，保证能丝滑滚动 */
@media (max-width: 900px) {
  :global(body), :global(html), :global(#app) { overflow-x: hidden !important; overflow-y: auto !important; height: auto !important; }
  .app-container { position: relative; height: auto; overflow: visible; padding-bottom: 30px; }
  .dashboard-wrapper { height: auto; padding: 20px; }
  .bento-layout { display: flex; flex-direction: column; height: auto; gap: 20px;}
  .cert-gallery { flex-direction: column; }
  .half-width { width: 100%; margin-bottom: 10px; }
}
</style>