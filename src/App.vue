<template>
  <div id="app" :class="{ 'mobile': isMobile }">
    <div class="game-container">
      <!-- 游戏标题 -->
      <div class="header">
        <h1 class="game-title">宝宝玩数字</h1>
        <div class="developer">开发商：言心股份有限公司</div>
      </div>
      
      <!-- 关卡信息 -->
      <div class="level-info">
        <span>场景: {{ currentSceneData.name }}</span>
        <span v-if="currentTask && currentTask.type !== 'logicTask'">数字: {{ currentNumber }}</span>
        <span v-else-if="currentTask && currentTask.type === 'logicTask'">挑战: 数一数</span>
      </div>
      
      <!-- 游戏区域 -->
      <div class="game-area">
        <!-- 开始界面 -->
        <div v-if="gameState === 'start'" class="start-screen">
          <div class="game-logo">
            <div class="number-character">🔢</div>
          </div>
          <h2>欢迎来到宝宝玩数字！</h2>
          <p>准备好开始数字之旅了吗？</p>
          
          <!-- 场景选择 -->
          <div class="scene-selector">
            <h3>选择场景</h3>
            <div class="scene-buttons">
              <button 
                v-for="scene in scenes" 
                :key="scene.id"
                class="scene-button"
                @click="selectScene(scene.id)"
              >
                <div class="scene-icon">{{ scene.icon }}</div>
                <div class="scene-name">{{ scene.name }}</div>
              </button>
            </div>
          </div>
          
          <button @click="startGame" class="start-button">开始游戏</button>
        </div>
        
        <!-- 引导界面 -->
        <div v-else-if="gameState === 'guide'" class="guide-screen">
          <div class="guide-content">
            <h2>{{ currentSceneData.name }}</h2>
            <div class="scene-image">
              <img :src="getSceneImage()" :alt="currentSceneData.name" class="scene-picture" />
            </div>
            <div class="guide-steps">
              <div class="guide-step">
                <div class="step-icon">👁️</div>
                <div class="step-content">
                  <h3>听一听</h3>
                  <p v-if="currentTask.type !== 'logicTask'">听数字儿歌，学习数字{{ currentNumber }}</p>
                  <p v-else>听数字儿歌，准备开始挑战</p>
                </div>
              </div>
              <div class="guide-step">
                <div class="step-icon">👀</div>
                <div class="step-content">
                  <h3>看一看</h3>
                  <p>观察{{ currentSceneData.name }}里的物品</p>
                </div>
              </div>
              <div class="guide-step">
                <div class="step-icon">👆</div>
                <div class="step-content">
                  <h3>点一点</h3>
                  <p v-if="currentTask && currentTask.type !== 'logicTask'">{{ currentTask.description }}</p>
                  <p v-else-if="currentTask && currentTask.type === 'logicTask'">{{ currentTask.scenario }}</p>
                  <p v-else>准备好开始游戏了吗？</p>
                </div>
              </div>
            </div>
            <button @click="playNumberSong" class="song-button">🎵 播放数字儿歌</button>
            <button @click="startLevel" class="start-level-button">开始游戏</button>
          </div>
        </div>
        
        <!-- 游戏界面 -->
        <div v-else-if="gameState === 'playing'" class="playing-screen">
          <!-- 场景背景 -->
          <div class="scene-background" :class="currentScene">
            <!-- 任务提示 -->
            <div class="task-prompt">
              <div class="task-description">{{ currentTask ? (currentTask.type === 'logicTask' ? currentTask.scenario : currentTask.description) : '' }}</div>
              <div class="voice-button" @click="playTaskPrompt">
                <span class="voice-icon">🔊</span>
              </div>
            </div>
            
            <!-- 互动区域 -->
            <div class="interaction-area">
              <!-- 数字认识游戏 -->
              <div v-if="currentTask.type === 'numberRecognition'" class="number-recognition">
                <div class="number-display">
                  <span class="number-digit">{{ currentNumber }}</span>
                  <div class="number-shape">数字{{ currentNumber }}像什么？</div>
                </div>
                <div class="shape-options">
                  <div 
                    v-for="(shape, index) in shapeOptions" 
                    :key="'shape-' + index"
                    :class="['shape-option', { 'selected': selectedItems.includes(index) }]"
                    @click="selectItem(index)"
                  >
                    <span class="shape-icon">{{ shape.icon }}</span>
                    <div class="shape-name">{{ shape.name }}</div>
                  </div>
                </div>
              </div>
              
              <!-- 数量对应游戏 -->
              <div v-else-if="currentTask.type === 'quantityMatching'" class="quantity-matching">
                <div class="target-number">
                  <span>需要: {{ currentNumber }}个{{ currentTask.targetItem }}</span>
                </div>
                <div class="items-grid">
                  <div 
                    v-for="(item, index) in taskItems" 
                    :key="'item-' + index"
                    :class="['task-item', { 'selected': selectedItems.includes(index) }]"
                    @click="selectItem(index)"
                  >
                    <span class="item-icon">{{ item.icon }}</span>
                  </div>
                </div>
                <div class="count-display">
                  <span>已选择: {{ selectedItems.length }}</span>
                  <span>目标: {{ currentNumber }}</span>
                </div>
              </div>
              
              <!-- 数理逻辑游戏 -->
              <div v-else-if="currentTask.type === 'logicTask'" class="logic-task">
                <div class="task-scenario">{{ currentTask.scenario }}</div>
                <!-- 场景视觉元素 -->
                <div class="logic-scenario">
                  <div v-if="currentScene === 'pond' && currentTask.description.includes('鱼篓')" class="scenario-image">
                    <div class="fishing-basket">
                      <span class="basket-icon">🎣</span>
                      <span class="basket-question">?</span>
                    </div>
                    <div class="fishing-pond">
                      <span class="fish">🐟</span>
                      <span class="fish">🐟</span>
                      <span class="fish">🐟</span>
                    </div>
                  </div>
                  <div v-else-if="currentScene === 'farm' && currentTask.description.includes('清点动物')" class="scenario-image">
                    <div class="farm-animals">
                      <span class="animal">🐔</span>
                      <span class="animal">🐄</span>
                      <span class="animal">🐑</span>
                    </div>
                  </div>
                  <div v-else-if="currentScene === 'ranch' && currentTask.description.includes('饲料')" class="scenario-image">
                    <div class="feed-piles">
                      <span class="feed">🌾</span>
                      <span class="feed">🌾</span>
                      <span class="feed">🌾</span>
                    </div>
                  </div>
                  <div v-else-if="currentScene === 'orchard' && currentTask.description.includes('收获')" class="scenario-image">
                    <div class="fruit-tree">
                      <span class="fruit">🍎</span>
                      <span class="fruit">🍐</span>
                      <span class="fruit">🍊</span>
                    </div>
                  </div>
                  <div v-else-if="currentScene === 'party' && currentTask.description.includes('蜡烛')" class="scenario-image">
                    <div class="birthday-cake">
                      <span class="cake">🎂</span>
                      <div class="candles">
                        <span class="candle">🕯️</span>
                        <span class="candle">🕯️</span>
                        <span class="candle">🕯️</span>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="logic-options">
                  <div 
                    v-for="(option, index) in logicOptions" 
                    :key="'logic-' + index"
                    :class="['logic-option', { 'selected': selectedItems.includes(index) }]"
                    @click="selectItem(index)"
                  >
                    <span class="option-content">{{ option.content }}</span>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 提交按钮 -->
            <button 
              class="submit-button" 
              :disabled="!canSubmit"
              @click="submitAnswer"
            >
              提交答案
            </button>
          </div>
        </div>
        
        <!-- 成功界面 -->
        <div v-else-if="gameState === 'success'" class="success-screen">
          <div class="success-animation">🎉</div>
          <h2>太棒了！</h2>
          <p>{{ successMessage }}</p>
          <div class="success-stars">
            <span v-for="i in 3" :key="i" class="star">⭐</span>
          </div>
        </div>
        
        <!-- 错误界面 -->
        <div v-else-if="gameState === 'error'" class="error-screen">
          <div class="error-animation">😅</div>
          <h2>再试一次！</h2>
          <p>{{ errorMessage }}</p>
          <button @click="retryLevel" class="retry-button">重新尝试</button>
        </div>
        
        <!-- 完成界面 -->
        <div v-else-if="gameState === 'complete'" class="complete-screen">
          <div class="complete-animation">🏆</div>
          <h2>恭喜完成！</h2>
          <p>你已经学会了数字1-10！</p>
          <button @click="restartGame" class="restart-button">重新开始</button>
          <button @click="backToStart" class="back-button">选择其他场景</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';

// 游戏状态
const gameState = ref('start'); // start, guide, playing, success, error, complete
const currentScene = ref('farm');
const currentNumber = ref(1);
const isMobile = ref(false);
const selectedItems = ref([]);
const currentTask = ref(null);
const shapeOptions = ref([]);
const taskItems = ref([]);
const logicOptions = ref([]);

// 场景配置
const scenes = [
  {
    id: 'farm',
    name: '欢乐农场',
    icon: '🐄',
    numbers: [1, 2, 3, 4, 5],
    tasks: [
      { type: 'quantityMatching', description: '给小鸡喂米，需要拿对应数量的大米', targetItem: '大米' },
      { type: 'numberRecognition', description: '认识数字形状，选择正确的图形联想' },
      { type: 'logicTask', description: '帮农场主清点动物，选择正确的数量' }
    ],
    items: [
      { icon: '🍚', name: '大米' },
      { icon: '🐔', name: '小鸡' },
      { icon: '🐄', name: '奶牛' },
      { icon: '🐑', name: '绵羊' }
    ]
  },
  {
    id: 'pond',
    name: '池塘趣钓',
    icon: '🐟',
    numbers: [6, 7, 8, 9, 10],
    tasks: [
      { type: 'quantityMatching', description: '数数钓了几条鱼，选择正确的数量', targetItem: '鱼' },
      { type: 'numberRecognition', description: '认识数字形状，选择正确的图形联想' },
      { type: 'logicTask', description: '根据鱼篓上的数字，选择正确的钓鱼数量' }
    ],
    items: [
      { icon: '🐟', name: '鱼' },
      { icon: '🎣', name: '钓鱼竿' },
      { icon: '🐸', name: '青蛙' },
      { icon: '🦆', name: '鸭子' }
    ]
  },
  {
    id: 'ranch',
    name: '青青牧场',
    icon: '🐑',
    numbers: [1, 2, 3, 4, 5],
    tasks: [
      { type: 'quantityMatching', description: '帮奶牛挤奶，需要挤对应数量的奶桶', targetItem: '奶桶' },
      { type: 'numberRecognition', description: '认识数字形状，选择正确的图形联想' },
      { type: 'logicTask', description: '根据动物数量，选择正确的饲料数量' }
    ],
    items: [
      { icon: '🥛', name: '奶桶' },
      { icon: '🐄', name: '奶牛' },
      { icon: '🐑', name: '绵羊' },
      { icon: '🌿', name: '青草' }
    ]
  },
  {
    id: 'orchard',
    name: '丰收果园',
    icon: '🍎',
    numbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10],
    tasks: [
      { type: 'quantityMatching', description: '摘果子，根据果篮上的数字选择对应数量', targetItem: '果子' },
      { type: 'numberRecognition', description: '认识数字形状，选择正确的图形联想' },
      { type: 'logicTask', description: '根据果树数量，选择正确的收获数量' }
    ],
    items: [
      { icon: '🍎', name: '苹果' },
      { icon: '🍐', name: '梨子' },
      { icon: '🍊', name: '橘子' },
      { icon: '🍋', name: '柠檬' }
    ]
  },
  {
    id: 'party',
    name: '生日派对',
    icon: '🎂',
    numbers: [1, 2, 3, 4, 5],
    tasks: [
      { type: 'quantityMatching', description: '准备餐具，根据小动物数量摆放对应数量的盘子', targetItem: '盘子' },
      { type: 'numberRecognition', description: '认识数字形状，选择正确的图形联想' },
      { type: 'logicTask', description: '根据生日蛋糕上的蜡烛数量，选择正确的数字' }
    ],
    items: [
      { icon: '🍰', name: '蛋糕' },
      { icon: '🍽️', name: '盘子' },
      { icon: '🥤', name: '杯子' },
      { icon: '🕯️', name: '蜡烛' }
    ]
  }
];

// 数字儿歌
const numberSongs = {
  1: '一，一，小铅笔，一根铅笔细又长。',
  2: '二，二，小鸭子，两只鸭子水中游。',
  3: '三，三，小耳朵，三只耳朵听声音。',
  4: '四，四，小旗子，四面旗子迎风飘。',
  5: '五，五，小钩子，五个钩子挂东西。',
  6: '六，六，小哨子，六个哨子嘟嘟响。',
  7: '七，七，小镰刀，七把镰刀割青草。',
  8: '八，八，小眼镜，八副眼镜看世界。',
  9: '九，九，小勺子，九把勺子吃饭饭。',
  10: '十，十，小棒子，十根棒子搭房子。'
};

// 数字形状联想
const numberShapes = {
  1: { icon: '✏️', name: '铅笔' },
  2: { icon: '🦆', name: '鸭子' },
  3: { icon: '👂', name: '耳朵' },
  4: { icon: '🚩', name: '旗子' },
  5: { icon: '🔗', name: '钩子' },
  6: { icon: '🎺', name: '哨子' },
  7: { icon: '🔪', name: '镰刀' },
  8: { icon: '👓', name: '眼镜' },
  9: { icon: '🥄', name: '勺子' },
  10: { icon: '🔟', name: '棒子' }
};

// 计算属性
const currentSceneData = computed(() => {
  return scenes.find(s => s.id === currentScene.value) || scenes[0];
});

const canSubmit = computed(() => {
  if (!currentTask.value) return false;
  if (currentTask.value.type === 'numberRecognition' || currentTask.value.type === 'logicTask') {
    return selectedItems.value.length === 1;
  } else if (currentTask.value.type === 'quantityMatching') {
    return selectedItems.value.length === currentNumber.value;
  }
  return false;
});

const successMessage = computed(() => {
  if (!currentTask.value) return '太棒了！你答对了！';
  if (currentTask.value.type === 'numberRecognition') {
    return `你认识数字${currentNumber.value}了！`;
  } else if (currentTask.value.type === 'quantityMatching') {
    return `你找到了${currentNumber.value}个${currentTask.value.targetItem}！`;
  } else if (currentTask.value.type === 'logicTask') {
    return `你解决了问题，太棒了！`;
  }
  return '太棒了！你答对了！';
});

const errorMessage = computed(() => {
  if (!currentTask.value) return '再试一次吧！';
  if (currentTask.value.type === 'numberRecognition') {
    return '再看看数字的形状，试试吧！';
  } else if (currentTask.value.type === 'quantityMatching') {
    return `请仔细数清楚，需要${currentNumber.value}个${currentTask.value.targetItem}哦！`;
  } else if (currentTask.value.type === 'logicTask') {
    return '再想想，试试其他选项吧！';
  }
  return '再试一次吧！';
});

// 检测设备类型
const checkDeviceType = () => {
  isMobile.value = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
};

// 检测屏幕大小
const handleResize = () => {
  checkDeviceType();
};

// 获取场景图片
const getSceneImage = () => {
  const prompts = {
    farm: 'cartoon farm scene with animals, colorful, child friendly',
    pond: 'cartoon pond scene with fish and frogs, colorful, child friendly',
    ranch: 'cartoon ranch scene with cows and sheep, colorful, child friendly',
    orchard: 'cartoon orchard scene with fruits, colorful, child friendly',
    party: 'cartoon birthday party scene with cake, colorful, child friendly'
  };
  const prompt = prompts[currentScene.value] || prompts.farm;
  return `https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=${encodeURIComponent(prompt)}&image_size=landscape_16_9`;
};

// 获取数字形状
const getNumberShape = (number) => {
  return numberShapes[number]?.name || '';
};

// 选择场景
const selectScene = (sceneId) => {
  currentScene.value = sceneId;
};

// 开始游戏
const startGame = () => {
  currentNumber.value = currentSceneData.value.numbers[0];
  gameState.value = 'guide';
  generateTask();
};

// 生成任务（固定关卡和题目）
const generateTask = () => {
  const tasks = currentSceneData.value.tasks;
  // 根据数字索引固定选择任务类型，确保每个数字对应固定的题目
  const taskIndex = (currentNumber.value - 1) % tasks.length;
  currentTask.value = tasks[taskIndex];
  
  if (currentTask.value.type === 'numberRecognition') {
    // 生成数字形状选项
    generateShapeOptions();
  } else if (currentTask.value.type === 'quantityMatching') {
    // 生成物品
    generateTaskItems();
  } else if (currentTask.value.type === 'logicTask') {
    // 生成逻辑选项
    generateLogicOptions();
  }
};

// 生成数字形状选项（固定选项）
const generateShapeOptions = () => {
  const options = [numberShapes[currentNumber.value]];
  // 添加其他形状作为干扰选项（固定顺序）
  const shapeKeys = Object.keys(numberShapes);
  const currentIndex = shapeKeys.indexOf(currentNumber.value.toString());
  
  // 添加固定位置的干扰项
  const distractIndices = [
    (currentIndex + 1) % shapeKeys.length,
    (currentIndex + 2) % shapeKeys.length,
    (currentIndex + 3) % shapeKeys.length
  ];
  
  distractIndices.forEach(index => {
    const shape = numberShapes[shapeKeys[index]];
    if (shape && !options.some(opt => opt.icon === shape.icon)) {
      options.push(shape);
    }
  });
  
  // 确保有4个选项
  while (options.length < 4) {
    const index = (currentIndex + options.length) % shapeKeys.length;
    const shape = numberShapes[shapeKeys[index]];
    if (shape && !options.some(opt => opt.icon === shape.icon)) {
      options.push(shape);
    }
  }
  
  // 固定顺序（不随机打乱）
  shapeOptions.value = options;
};

// 生成任务物品（固定物品）
const generateTaskItems = () => {
  const items = [];
  const sceneItems = currentSceneData.value.items;
  const targetItem = sceneItems.find(item => item.name === currentTask.value.targetItem) || sceneItems[0];
  
  // 添加目标物品（固定数量，避免通过数量推断答案）
  const targetCount = Math.min(currentNumber.value + 2, 6);
  for (let i = 0; i < targetCount; i++) {
    items.push({ icon: targetItem.icon, isTarget: true });
  }
  
  // 添加其他物品作为干扰（固定顺序）
  const otherItems = sceneItems.filter(item => item.icon !== targetItem.icon);
  const distractCount = 6 - targetCount;
  for (let i = 0; i < distractCount; i++) {
    const distractItem = otherItems[i % otherItems.length];
    items.push({ icon: distractItem.icon, isTarget: false });
  }
  
  // 固定顺序（不随机打乱）
  taskItems.value = items;
};

// 生成逻辑选项（固定选项）
const generateLogicOptions = () => {
  const options = [];
  // 正确答案
  options.push({ content: currentNumber.value.toString(), isCorrect: true });
  
  // 固定的干扰选项（根据当前数字生成固定的干扰项）
  const wrongAnswers = [
    (currentNumber.value % 10) + 1,
    ((currentNumber.value + 2) % 10) + 1,
    ((currentNumber.value + 4) % 10) + 1
  ];
  
  wrongAnswers.forEach(num => {
    if (!options.some(opt => opt.content === num.toString())) {
      options.push({ content: num.toString(), isCorrect: false });
    }
  });
  
  // 确保有4个选项
  while (options.length < 4) {
    const randomNumber = ((currentNumber.value + options.length) % 10) + 1;
    if (!options.some(opt => opt.content === randomNumber.toString())) {
      options.push({ content: randomNumber.toString(), isCorrect: false });
    }
  }
  
  // 固定顺序（不随机打乱）
  logicOptions.value = options;
  
  // 设置任务场景描述
  if (currentScene.value === 'pond' && currentTask.value.description.includes('鱼篓')) {
    currentTask.value.scenario = `鱼篓上有一个数字，应该钓几条鱼呢？`;
  } else if (currentScene.value === 'farm' && currentTask.value.description.includes('清点动物')) {
    currentTask.value.scenario = `农场里有一些动物，你能数对有几只吗？`;
  } else if (currentScene.value === 'ranch' && currentTask.value.description.includes('饲料')) {
    currentTask.value.scenario = `需要准备一些饲料，你知道需要多少份吗？`;
  } else if (currentScene.value === 'orchard' && currentTask.value.description.includes('收获')) {
    currentTask.value.scenario = `果树上结了一些果子，你能摘对数量吗？`;
  } else if (currentScene.value === 'party' && currentTask.value.description.includes('蜡烛')) {
    currentTask.value.scenario = `生日蛋糕上插了一些蜡烛，你能数对有几根吗？`;
  } else {
    currentTask.value.scenario = currentTask.value.description;
  }
};

// 开始关卡
const startLevel = () => {
  selectedItems.value = [];
  gameState.value = 'playing';
  // 自动播放任务提示
  setTimeout(() => {
    playTaskPrompt();
  }, 500);
};

// 播放数字儿歌
const playNumberSong = () => {
  const song = numberSongs[currentNumber.value];
  if (song) {
    speak(song);
  }
};

// 播放任务提示
const playTaskPrompt = () => {
  if (currentTask.value.type === 'numberRecognition') {
    speak(`数字${currentNumber.value}像什么？`);
  } else if (currentTask.value.type === 'quantityMatching') {
    speak(`请选择${currentNumber.value}个${currentTask.value.targetItem}`);
  } else if (currentTask.value.type === 'logicTask') {
    speak(currentTask.value.scenario);
  }
};

// 语音合成
const speak = (text) => {
  if ('speechSynthesis' in window) {
    const speech = new SpeechSynthesisUtterance(text);
    speech.lang = 'zh-CN';
    speech.rate = 0.8;
    speech.pitch = 1.2;
    speech.volume = 1;
    window.speechSynthesis.speak(speech);
  }
};

// 选择物品
const selectItem = (index) => {
  if (currentTask.value.type === 'numberRecognition' || currentTask.value.type === 'logicTask') {
    // 单选
    selectedItems.value = [index];
  } else if (currentTask.value.type === 'quantityMatching') {
    // 多选
    if (selectedItems.value.includes(index)) {
      // 取消选择
      selectedItems.value = selectedItems.value.filter(i => i !== index);
    } else {
      // 选择物品
      selectedItems.value.push(index);
    }
  }
};

// 提交答案
const submitAnswer = () => {
  let isCorrect = false;
  
  if (currentTask.value.type === 'numberRecognition') {
    // 数字认识游戏
    const selectedIndex = selectedItems.value[0];
    const selectedShape = shapeOptions.value[selectedIndex];
    isCorrect = selectedShape.icon === numberShapes[currentNumber.value].icon;
  } else if (currentTask.value.type === 'quantityMatching') {
    // 数量对应游戏
    if (selectedItems.value.length === currentNumber.value) {
      const allTargetItems = selectedItems.value.every(index => taskItems.value[index].isTarget);
      isCorrect = allTargetItems;
    }
  } else if (currentTask.value.type === 'logicTask') {
    // 数理逻辑游戏
    const selectedIndex = selectedItems.value[0];
    isCorrect = logicOptions.value[selectedIndex].isCorrect;
  }
  
  if (isCorrect) {
    // 正确
    gameState.value = 'success';
    speak('太棒了！你答对了！');
    
    // 进入下一个数字或完成
    setTimeout(() => {
      const currentSceneNumbers = currentSceneData.value.numbers;
      const currentIndex = currentSceneNumbers.indexOf(currentNumber.value);
      
      if (currentIndex < currentSceneNumbers.length - 1) {
        currentNumber.value = currentSceneNumbers[currentIndex + 1];
        generateTask();
        gameState.value = 'guide';
      } else {
        gameState.value = 'complete';
        speak('恭喜你完成了所有数字学习！');
      }
    }, 2000);
  } else {
    // 错误
    gameState.value = 'error';
    speak(errorMessage.value);
  }
};

// 重新尝试
const retryLevel = () => {
  selectedItems.value = [];
  gameState.value = 'playing';
};

// 重新开始
const restartGame = () => {
  currentNumber.value = currentSceneData.value.numbers[0];
  generateTask();
  gameState.value = 'start';
};

// 返回主页
const backToStart = () => {
  gameState.value = 'start';
};

// 生命周期钩子
onMounted(() => {
  checkDeviceType();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});
</script>

<style scoped>
/* 全局样式 */
#app {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 50%, #fecfef 100%);
  font-family: 'Comic Sans MS', 'Arial', sans-serif;
  transition: all 0.3s ease;
  overflow: hidden;
}

/* 游戏容器 */
.game-container {
  width: 90%;
  max-width: 800px;
  height: 90%;
  max-height: 90vh; /* 使用视口高度的90%，确保在手机上也能显示完整 */
  background: #fff9e6;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  display: flex;
  flex-direction: column;
  padding: 20px;
  box-sizing: border-box;
  border: 5px solid #ffcc80;
  position: relative;
  overflow: hidden;
}

/* 装饰元素 */
.game-container::before {
  content: '';
  position: absolute;
  top: -50px;
  right: -50px;
  width: 150px;
  height: 150px;
  background: #ffeb3b;
  border-radius: 50%;
  opacity: 0.3;
  z-index: 0;
}

.game-container::after {
  content: '';
  position: absolute;
  bottom: -50px;
  left: -50px;
  width: 120px;
  height: 120px;
  background: #4caf50;
  border-radius: 50%;
  opacity: 0.2;
  z-index: 0;
}

/* 头部 */
.header {
  text-align: center;
  margin-bottom: 10px;
  z-index: 1;
  position: relative;
}

.game-title {
  color: #ff6b6b;
  margin: 0 0 5px 0;
  font-size: 2.5rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.developer {
  color: #666;
  font-size: 0.9rem;
}

/* 关卡信息 */
.level-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
  font-size: 1.2rem;
  color: #666;
  font-weight: bold;
  z-index: 1;
  position: relative;
}

/* 游戏区域 */
.game-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  position: relative;
  z-index: 1;
  overflow-y: auto;
  padding: 10px;
}

/* 开始界面 */
.start-screen {
  text-align: center;
  width: 100%;
}

.game-logo {
  margin-bottom: 30px;
}

.number-character {
  font-size: 5rem;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-20px); }
  60% { transform: translateY(-10px); }
}

.start-screen h2 {
  color: #4ecdc4;
  margin-bottom: 20px;
  font-size: 1.8rem;
}

.start-screen p {
  color: #666;
  margin-bottom: 30px;
  font-size: 1.1rem;
}

/* 场景选择 */
.scene-selector {
  margin-bottom: 30px;
}

.scene-selector h3 {
  color: #45b7d1;
  margin-bottom: 15px;
}

.scene-buttons {
  display: flex;
  justify-content: center;
  gap: 20px;
  flex-wrap: wrap;
}

.scene-button {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 15px;
  border: 2px solid #45b7d1;
  border-radius: 15px;
  background: white;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 100px;
}

.scene-button:hover {
  background: #45b7d1;
  color: white;
  transform: scale(1.1);
}

.scene-icon {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.scene-name {
  font-weight: bold;
}

/* 引导界面 */
.guide-screen {
  width: 100%;
  text-align: center;
}

.guide-content h2 {
  color: #4ecdc4;
  margin-bottom: 20px;
}

.scene-image {
  margin-bottom: 20px;
}

.scene-picture {
  width: 80%;
  max-width: 400px;
  border-radius: 15px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.guide-steps {
  margin-bottom: 30px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
}

.guide-step {
  display: flex;
  align-items: center;
  gap: 15px;
  background: white;
  padding: 15px;
  border-radius: 15px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  width: 80%;
  max-width: 400px;
  animation: slideIn 0.5s ease-in;
}

@keyframes slideIn {
  from { opacity: 0; transform: translateX(-50px); }
  to { opacity: 1; transform: translateX(0); }
}

.guide-step:nth-child(2) {
  animation-delay: 0.2s;
}

.guide-step:nth-child(3) {
  animation-delay: 0.4s;
}

.step-icon {
  font-size: 2rem;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #f0f8ff;
  border-radius: 50%;
  flex-shrink: 0;
}

.step-content {
  text-align: left;
  flex: 1;
}

.step-content h3 {
  margin: 0 0 5px 0;
  color: #333;
}

.step-content p {
  margin: 0;
  color: #666;
  font-size: 0.9rem;
}

/* 游戏界面 */
.playing-screen {
  width: 100%;
  height: 100%;
}

.scene-background {
  width: 100%;
  height: 100%;
  border-radius: 15px;
  padding: 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  position: relative;
  background-size: cover;
  background-position: center;
}

.scene-background.farm {
  background: linear-gradient(rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.8)), url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cartoon%20farm%20scene%20with%20green%20fields%20and%20barn&image_size=landscape_16_9');
  background-size: cover;
  background-position: center;
}

.scene-background.pond {
  background: linear-gradient(rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.8)), url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cartoon%20pond%20scene%20with%20water%20and%20lily%20pads&image_size=landscape_16_9');
  background-size: cover;
  background-position: center;
}

.scene-background.ranch {
  background: linear-gradient(rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.8)), url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cartoon%20ranch%20scene%20with%20cows%20and%20sheep&image_size=landscape_16_9');
  background-size: cover;
  background-position: center;
}

.scene-background.orchard {
  background: linear-gradient(rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.8)), url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cartoon%20orchard%20scene%20with%20fruits%20and%20trees&image_size=landscape_16_9');
  background-size: cover;
  background-position: center;
}

.scene-background.party {
  background: linear-gradient(rgba(255, 255, 255, 0.8), rgba(255, 255, 255, 0.8)), url('https://trae-api-cn.mchost.guru/api/ide/v1/text_to_image?prompt=cartoon%20birthday%20party%20scene%20with%20cake&image_size=landscape_16_9');
  background-size: cover;
  background-position: center;
}

.task-prompt {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.9);
  padding: 15px;
  border-radius: 15px;
  margin-bottom: 20px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
  width: 100%;
}

.task-description {
  font-size: 1.2rem;
  font-weight: bold;
  color: #333;
  flex: 1;
}

.voice-button {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: #4ecdc4;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: all 0.3s ease;
  margin-left: 15px;
}

.voice-button:hover {
  transform: scale(1.1);
  background: #45b7d1;
}

.voice-icon {
  font-size: 1.5rem;
  color: white;
}

/* 互动区域 */
.interaction-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
  width: 100%;
}

/* 数字认识游戏 */
.number-recognition {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.number-display {
  text-align: center;
  margin-bottom: 30px;
}

.number-digit {
  font-size: 4rem;
  font-weight: bold;
  color: #ff6b6b;
  display: block;
}

.number-shape {
  font-size: 1.5rem;
  color: #666;
  margin-top: 10px;
}

.shape-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  width: 100%;
  max-width: 300px;
}

.shape-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.shape-option:hover {
  transform: scale(1.05);
  background: rgba(255, 255, 255, 1);
}

.shape-option.selected {
  background: #4caf50;
  color: white;
  transform: scale(1.05);
}

.shape-icon {
  font-size: 2.5rem;
  margin-bottom: 10px;
}

.shape-name {
  font-weight: bold;
}

/* 数量对应游戏 */
.quantity-matching {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.target-number {
  background: rgba(255, 255, 255, 0.9);
  padding: 15px;
  border-radius: 15px;
  margin-bottom: 20px;
  font-size: 1.2rem;
  font-weight: bold;
  color: #333;
}

.items-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  width: 100%;
  max-width: 300px;
  margin-bottom: 20px;
}

.task-item {
  aspect-ratio: 1;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.task-item:hover {
  transform: scale(1.1);
  background: rgba(255, 255, 255, 1);
}

.task-item.selected {
  background: #4caf50;
  transform: scale(1.1);
}

.item-icon {
  font-size: 2rem;
}

.count-display {
  display: flex;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.9);
  padding: 10px 15px;
  border-radius: 10px;
  margin-bottom: 20px;
  font-weight: bold;
  color: #333;
  width: 100%;
  max-width: 300px;
}

/* 数理逻辑游戏 */
.logic-task {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.task-scenario {
  background: rgba(255, 255, 255, 0.9);
  padding: 15px;
  border-radius: 15px;
  margin-bottom: 20px;
  font-size: 1.1rem;
  color: #333;
  text-align: center;
  width: 100%;
}

.logic-scenario {
  margin: 20px 0;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 100%;
}

.scenario-image {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

/* 钓鱼场景 */
.fishing-basket {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 24px;
  font-weight: bold;
}

.basket-question {
  background-color: #45b7d1;
  color: white;
  padding: 10px 20px;
  border-radius: 50%;
  animation: bounce 2s infinite;
  font-size: 28px;
}

.fishing-pond {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

.fish {
  font-size: 32px;
  animation: swim 3s infinite ease-in-out;
}

/* 农场场景 */
.farm-animals {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  justify-content: center;
}

.animal {
  font-size: 36px;
  animation: bounce 2s infinite;
}

/* 牧场场景 */
.feed-piles {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

.feed {
  font-size: 28px;
  animation: sway 3s infinite ease-in-out;
}

/* 果园场景 */
.fruit-tree {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  justify-content: center;
}

.fruit {
  font-size: 24px;
  animation: float 2s infinite ease-in-out;
}

/* 派对场景 */
.birthday-cake {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.cake {
  font-size: 48px;
}

.candles {
  display: flex;
  gap: 5px;
}

.candle {
  font-size: 20px;
  animation: flicker 1.5s infinite alternate;
}

/* 动画效果 */
@keyframes swim {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  50% { transform: translateY(-10px) rotate(10deg); }
}

@keyframes sway {
  0%, 100% { transform: rotate(0deg); }
  50% { transform: rotate(10deg); }
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

@keyframes flicker {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.logic-options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
  width: 100%;
  max-width: 300px;
}

.logic-option {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 25px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.logic-option:hover {
  transform: scale(1.05);
  background: rgba(255, 255, 255, 1);
}

.logic-option.selected {
  background: #4caf50;
  color: white;
  transform: scale(1.05);
}

.option-content {
  font-size: 2rem;
  font-weight: bold;
}

/* 提交按钮 */
.submit-button {
  padding: 15px 30px;
  font-size: 1.2rem;
  border: none;
  border-radius: 25px;
  background: linear-gradient(135deg, #4caf50, #45a049);
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
  align-self: center;
}

.submit-button:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.submit-button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 成功界面 */
.success-screen {
  text-align: center;
  animation: fadeIn 0.5s ease-in;
}

@keyframes fadeIn {
  from { opacity: 0; transform: scale(0.8); }
  to { opacity: 1; transform: scale(1); }
}

.success-animation {
  font-size: 4rem;
  margin-bottom: 20px;
  animation: bounce 2s infinite;
}

.success-screen h2 {
  color: #4caf50;
  margin-bottom: 10px;
}

.success-screen p {
  color: #666;
  margin-bottom: 20px;
}

.success-stars {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.star {
  font-size: 1.5rem;
  animation: twinkle 1s ease-in-out infinite;
}

.star:nth-child(2) {
  animation-delay: 0.2s;
}

.star:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes twinkle {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* 错误界面 */
.error-screen {
  text-align: center;
  animation: fadeIn 0.5s ease-in;
}

.error-animation {
  font-size: 4rem;
  margin-bottom: 20px;
}

.error-screen h2 {
  color: #ff6b6b;
  margin-bottom: 10px;
}

.error-screen p {
  color: #666;
  margin-bottom: 20px;
}

/* 完成界面 */
.complete-screen {
  text-align: center;
  animation: fadeIn 0.5s ease-in;
}

.complete-animation {
  font-size: 4rem;
  margin-bottom: 20px;
}

.complete-screen h2 {
  color: #4caf50;
  margin-bottom: 10px;
}

.complete-screen p {
  color: #666;
  margin-bottom: 20px;
}

/* 按钮样式 */
.start-button, .start-level-button, .song-button, .retry-button, .restart-button, .back-button {
  padding: 15px 30px;
  font-size: 1.2rem;
  border: none;
  border-radius: 25px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: bold;
  margin: 5px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.start-button, .start-level-button {
  background: linear-gradient(135deg, #4caf50, #45a049);
}

.song-button {
  background: linear-gradient(135deg, #ff9800, #f57c00);
  margin-bottom: 15px;
}

.retry-button {
  background: linear-gradient(135deg, #ff9800, #f57c00);
}

.restart-button {
  background: linear-gradient(135deg, #4caf50, #45a049);
}

.back-button {
  background: linear-gradient(135deg, #2196f3, #1976d2);
}

.start-button:hover, .start-level-button:hover, .song-button:hover, .retry-button:hover, .restart-button:hover, .back-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .game-title {
    font-size: 2rem;
  }
  
  .level-info {
    font-size: 1rem;
  }
  
  .number-digit {
    font-size: 3rem;
  }
  
  .number-shape {
    font-size: 1.2rem;
  }
  
  .shape-icon, .item-icon {
    font-size: 1.5rem;
  }
  
  .start-button, .start-level-button, .song-button, .retry-button, .restart-button, .back-button {
    padding: 12px 24px;
    font-size: 1rem;
  }
  
  .scene-buttons {
    gap: 15px;
  }
  
  .scene-button {
    min-width: 80px;
    padding: 10px;
  }
  
  .scene-icon {
    font-size: 2rem;
  }
}

@media (max-width: 480px) {
  .game-container {
    width: 95%;
    height: 95%;
    padding: 15px;
  }
  
  .game-title {
    font-size: 1.5rem;
  }
  
  .developer {
    font-size: 0.8rem;
  }
  
  .number-character {
    font-size: 3.5rem;
  }
  
  .number-digit {
    font-size: 2.5rem;
  }
  
  .number-shape {
    font-size: 1rem;
  }
  
  .shape-icon, .item-icon {
    font-size: 1.2rem;
  }
  
  .items-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 10px;
  }
  
  .guide-step {
    width: 90%;
    padding: 12px;
  }
  
  .step-icon {
    font-size: 1.5rem;
    width: 40px;
    height: 40px;
  }
  
  .step-content h3 {
    font-size: 1rem;
  }
  
  .step-content p {
    font-size: 0.8rem;
  }
  
  .scene-buttons {
    gap: 10px;
  }
  
  .scene-button {
    min-width: 70px;
    padding: 8px;
  }
  
  .scene-icon {
    font-size: 1.5rem;
  }
  
  .scene-name {
    font-size: 0.9rem;
  }
}

/* 移动端优化 */
#app.mobile .shape-option, #app.mobile .task-item, #app.mobile .logic-option {
  touch-action: manipulation;
}

#app.mobile .scene-button {
  touch-action: manipulation;
}

#app.mobile .start-button, #app.mobile .start-level-button, #app.mobile .song-button, #app.mobile .retry-button, #app.mobile .restart-button, #app.mobile .back-button {
  touch-action: manipulation;
}
</style>