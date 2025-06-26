<!-- index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>CSGO开箱站 - 安全合规的开箱平台</title>
  <script src="https://unpkg.com/vue@3.3.4/dist/vue.global.js"></script>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
  <script src="https://unpkg.com/@tailwindcss/browser@4"></script>
  <style type="text/tailwindcss">
    @theme {
      --color-primary: #4d94ff;
      --color-secondary: #ff9900;
      --color-danger: #e63946;
      --color-success: #4CAF50;
      --color-bg: #121212;
      --color-card: #1e1e1e;
      --font-inter: 'Inter', sans-serif;
    }
    
    @utility shake {
      animation: shake 0.5s cubic-bezier(.36,.07,.19,.97) both;
    }
    
    @keyframes shake {
      10%, 90% { transform: translate3d(-1px, 0, 0); }
      20%, 80% { transform: translate3d(2px, 0, 0); }
      30%, 50%, 70% { transform: translate3d(-4px, 0, 0); }
      40%, 60% { transform: translate3d(4px, 0, 0); }
    }
    
    @utility pop-in {
      animation: popIn 0.5s ease-out;
    }
    
    @keyframes popIn {
      0% { transform: scale(0.5); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }
  </style>
</head>
<body class="font-inter bg-bg text-gray-100 min-h-screen">
  <div id="app" class="container mx-auto px-4 py-6">
    <!-- 导航栏 -->
    <nav class="flex justify-between items-center mb-8 py-4 border-b border-gray-700">
      <div class="flex items-center space-x-2">
        <i class="fas fa-crate-full text-primary text-2xl"></i>
        <h1 class="text-2xl font-bold text-white">CSGO开箱站</h1>
      </div>
      <div class="flex items-center space-x-4">
        <a href="#cases" class="hover:text-primary transition">箱子列表</a>
        <a href="#history" class="hover:text-primary transition">开箱历史</a>
        <a href="#rules" class="hover:text-primary transition">开箱规则</a>
        <button v-if="!isLoggedIn" class="bg-primary hover:bg-blue-600 text-white px-4 py-2 rounded-md transition" @click="showLogin = true">
          登录/注册
        </button>
        <div v-else class="flex items-center space-x-3">
          <img :src="user.avatar" alt="用户头像" class="w-8 h-8 rounded-full border-2 border-primary">
          <span class="text-white">{{ user.username }}</span>
          <button class="bg-danger hover:bg-red-700 text-white px-3 py-1 rounded-md transition" @click="logout">
            退出
          </button>
        </div>
      </div>
    </nav>
    
    <!-- 轮播广告 -->
    <div class="relative rounded-lg overflow-hidden mb-10 h-48 md:h-64">
      <img src="https://picsum.photos/1200/400?random=banner" alt="广告" class="w-full h-full object-cover">
      <div class="absolute inset-0 bg-black bg-opacity-40 flex items-center justify-center">
        <div class="text-center">
          <h2 class="text-2xl md:text-3xl font-bold text-white mb-2">新用户首充送10%余额</h2>
          <p class="text-gray-200">立即充值享受开箱乐趣</p>
        </div>
      </div>
    </div>
    
    <!-- 用户面板 -->
    <div v-if="isLoggedIn" class="bg-card rounded-lg p-6 mb-10 shadow-lg">
      <div class="flex flex-col md:flex-row justify-between items-start md:items-center">
        <div class="mb-4 md:mb-0">
          <h2 class="text-xl font-semibold text-white mb-2">账户信息</h2>
          <p class="text-gray-400">注册时间: {{ user.createdAt }}</p>
        </div>
        <div class="flex flex-col items-center md:items-end space-y-3 md:space-y-0 md:space-x-4">
          <div class="flex items-center">
            <i class="fas fa-wallet text-secondary text-xl mr-2"></i>
            <span class="text-2xl font-bold text-white">{{ user.balance }} <span class="text-secondary">$</span></span>
          </div>
          <button class="bg-secondary hover:bg-yellow-600 text-white px-6 py-2 rounded-md transition" @click="showDeposit = true">
            充值余额
          </button>
        </div>
      </div>
    </div>
    
    <!-- 箱子列表 -->
    <section id="cases" class="mb-16">
      <h2 class="text-2xl font-bold text-primary mb-6 flex items-center">
        <i class="fas fa-crate-full mr-2"></i>开箱箱子
      </h2>
      <p class="text-gray-400 mb-8">所有箱子概率已公示，点击查看详细概率</p>
      
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
        <div v-for="caseItem in cases" :key="caseItem.id" class="bg-card rounded-lg overflow-hidden hover:shadow-lg hover:-translate-y-1 transition duration-300 cursor-pointer" @click="openCase(caseItem)">
          <div class="relative h-48">
            <img :src="caseItem.image" alt="箱子" class="w-full h-full object-cover">
            <div class="absolute top-2 right-2 bg-danger text-white text-xs font-bold px-2 py-1 rounded">
              {{ caseItem.price }} $
            </div>
          </div>
          <div class="p-4">
            <h3 class="text-lg font-semibold text-white mb-2">{{ caseItem.name }}</h3>
            <p class="text-sm text-gray-400 mb-3">稀有度: {{ caseItem.rarity }}</p>
            <button class="w-full bg-primary/20 hover:bg-primary/30 text-primary font-medium py-2 rounded transition">
              查看概率
            </button>
          </div>
        </div>
      </div>
    </section>
    
    <!-- 开箱动画模态框 -->
    <div v-if="isOpening" class="fixed inset-0 bg-black bg-opacity-80 flex items-center justify-center z-50">
      <div class="bg-card rounded-lg p-6 max-w-md w-full relative">
        <button class="absolute top-4 right-4 text-gray-400 hover:text-white" @click="closeOpening">
          <i class="fas fa-times text-xl"></i>
        </button>
        <h3 class="text-xl font-bold text-white text-center mb-6">正在开启 {{ currentCase.name }}</h3>
        
        <div class="flex justify-center mb-6">
          <div class="w-64 h-64 relative">
            <img v-if="!itemRevealed" :src="currentCase.image" alt="箱子" class="w-full h-full object-cover rounded-lg shake">
            <div v-else class="absolute inset-0 bg-black bg-opacity-70 rounded-lg flex flex-col items-center justify-center">
              <img :src="currentItem.image" alt="开出的物品" class="w-48 h-48 object-contain pop-in mb-4">
              <h4 :class="{'text-red-500': currentItem.rarity === '珍稀', 
                          'text-cyan-400': currentItem.rarity === '罕见',
                          'text-yellow-400': currentItem.rarity === '受限',
                          'text-gray-400': currentItem.rarity === '军规',
                          'text-purple-400': currentItem.rarity === '工业',
                          'text-blue-400': currentItem.rarity === '消费'}" 
                  class="text-lg font-bold mb-2">{{ currentItem.name }}</h4>
              <p class="text-sm text-gray-400 mb-3">{{ currentItem.rarity }} 级</p>
              <p class="text-yellow-500 font-medium">{{ currentItem.value }} $</p>
            </div>
          </div>
        </div>
        
        <div v-if="!itemRevealed" class="text-center">
          <button class="bg-primary hover:bg-blue-600 text-white px-8 py-3 rounded-md transition text-lg font-bold">
            点击开箱
          </button>
        </div>
        
        <div v-else class="text-center">
          <button class="bg-success hover:bg-green-600 text-white px-8 py-3 rounded-md transition text-lg font-bold" @click="addtoInventory">
            加入库存
          </button>
        </div>
      </div>
    </div>
    
    <!-- 充值模态框 -->
    <div v-if="showDeposit" class="fixed inset-0 bg-black bg-opacity-80 flex items-center justify-center z-50">
      <div class="bg-card rounded-lg p-6 max-w-md w-full">
        <h3 class="text-xl font-bold text-white text-center mb-6">账户充值</h3>
        <form @submit.prevent="processDeposit">
          <div class="mb-4">
            <label for="depositAmount" class="block text-gray-300 mb-2">充值金额 (美元)</label>
            <div class="flex space-x-2 mb-4">
              <button type="button" class="deposit-btn bg-primary/20 hover:bg-primary/30 text-primary px-4 py-2 rounded" @click="depositAmount = 50">50</button>
              <button type="button" class="deposit-btn bg-primary/20 hover:bg-primary/30 text-primary px-4 py-2 rounded" @click="depositAmount = 100">100</button>
              <button type="button" class="deposit-btn bg-primary/20 hover:bg-primary/30 text-primary px-4 py-2 rounded" @click="depositAmount = 200">200</button>
            </div>
            <input type="number" id="depositAmount" v-model="depositAmount" min="10" step="10" 
                   class="w-full bg-gray-800 border border-gray-700 text-white px-4 py-3 rounded-md focus:outline-none focus:border-primary">
          </div>
          
          <div class="mb-6 text-sm text-gray-400">
            <p>• 充值将在1-3分钟内到账</p>
            <p>• 首充用户额外获得10%余额奖励</p>
          </div>
          
          <div class="flex space-x-3">
            <button type="button" class="flex-1 bg-gray-700 hover:bg-gray-600 text-white px-4 py-3 rounded-md transition" @click="showDeposit = false">
              取消
            </button>
            <button type="submit" class="flex-1 bg-secondary hover:bg-yellow-600 text-white px-4 py-3 rounded-md transition">
              确认充值
            </button>
          </div>
        </form>
      </div>
    </div>
    
    <!-- 登录/注册模态框 -->
    <div v-if="showLogin" class="fixed inset-0 bg-black bg-opacity-80 flex items-center justify-center z-50">
      <div class="bg-card rounded-lg p-6 max-w-md w-full">
        <div class="flex justify-between items-center mb-6">
          <h3 class="text-xl font-bold text-white">用户登录</h3>
          <button @click="toggleLoginRegister">
            <span v-if="isLogin">没有账号?</span>
            <span v-else>已有账号?</span>
          </button>
        </div>
        
        <form @submit.prevent="handleAuth">
          <div class="mb-4">
            <label for="username" class="block text-gray-300 mb-2">用户名</label>
            <input type="text" id="username" v-model="authForm.username" 
                   class="w-full bg-gray-800 border border-gray-700 text-white px-4 py-3 rounded-md focus:outline-none focus:border-primary">
          </div>
          
          <div class="mb-6">
            <label for="password" class="block text-gray-300 mb-2">密码</label>
            <input type="password" id="password" v-model="authForm.password" 
                   class="w-full bg-gray-800 border border-gray-700 text-white px-4 py-3 rounded-md focus:outline-none focus:border-primary">
          </div>
          
          <div v-if="!isLogin" class="mb-6">
            <label for="email" class="block text-gray-300 mb-2">邮箱</label>
            <input type="email" id="email" v-model="authForm.email" 
                   class="w-full bg-gray-800 border border-gray-700 text-white px-4 py-3 rounded-md focus:outline-none focus:border-primary">
          </div>
          
          <div class="flex space-x-3 mb-6">
            <button type="button" class="flex-1 bg-gray-700 hover:bg-gray-600 text-white px-4 py-3 rounded-md transition" @click="showLogin = false">
              取消
            </button>
            <button type="submit" class="flex-1 bg-primary hover:bg-blue-600 text-white px-4 py-3 rounded-md transition">
              {{ isLogin ? '登录' : '注册' }}
            </button>
          </div>
        </form>
      </div>
    </div>
    
    <!-- 开箱历史 -->
    <section id="history" class="mb-16">
      <h2 class="text-2xl font-bold text-primary mb-6 flex items-center">
        <i class="fas fa-history mr-2"></i>最近开箱记录
      </h2>
      
      <div class="bg-card rounded-lg overflow-hidden">
        <div class="overflow-x-auto">
          <table class="min-w-full divide-y divide-gray-700">
            <thead class="bg-gray-800">
              <tr>
                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-400 uppercase tracking-wider">用户</th>
                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-400 uppercase tracking-wider">箱子</th>
                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-400 uppercase tracking-wider">开出物品</th>
                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-400 uppercase tracking-wider">价值</th>
                <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-400 uppercase tracking-wider">时间</th>
              </tr>
            </thead>
            <tbody class="bg-gray-900 divide-y divide-gray-800">
              <tr v-for="(history, index) in publicHistory.slice(0, 10)" :key="index" class="hover:bg-gray-800 transition">
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="flex items-center">
                    <img :src="history.avatar" alt="用户头像" class="h-8 w-8 rounded-full mr-3">
                    <div class="text-sm font-medium text-white">{{ history.username }}</div>
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-300">
                  {{ history.caseName }}
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <div class="text-sm" :class="{'text-red-500': history.itemRarity === '珍稀', 
                                              'text-cyan-400': history.itemRarity === '罕见',
                                              'text-yellow-400': history.itemRarity === '受限',
                                              'text-gray-400': history.itemRarity === '军规',
                                              'text-purple-400': history.itemRarity === '工业',
                                              'text-blue-400': history.itemRarity === '消费'}">
                    {{ history.itemName }}
                  </div>
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-yellow-400">
                  {{ history.value }} $
                </td>
                <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-400">
                  {{ history.time }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>
    
    <!-- 开箱规则 -->
    <section id="rules" class="mb-16">
      <h2 class="text-2xl font-bold text-primary mb-6 flex items-center">
        <i class="fas fa-gavel mr-2"></i>开箱规则与概率
      </h2>
      
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div class="bg-card rounded-lg p-6">
          <h3 class="text-xl font-semibold text-white mb-4 flex items-center">
            <i class="fas fa-info-circle text-primary mr-2"></i>开箱规则
          </h3>
          <ul class="space-y-3 text-gray-300">
            <li class="flex items-start">
              <i class="fas fa-check text-success mt-1 mr-2"></i>
              <span>所有开箱概率公开透明，可在各箱子详情查看</span>
            </li>
            <li class="flex items-start">
              <i class="fas fa-check text-success mt-1 mr-2"></i>
              <span>开箱结果由加密随机数生成，无法人为干预</span>
            </li>
            <li class="flex items-start">
              <i class="fas fa-check text-success mt-1 mr-2"></i>
              <span>开出的饰品可在个人库存查看，支持交易到Steam</span>
            </li>
            <li class="flex items-start">
              <i class="fas fa-check text-success mt-1 mr-2"></i>
              <span>平台不支持现金提现，饰品可在平台内交易</span>
            </li>
            <li class="flex items-start">
              <i class="fas fa-check text-success mt-1 mr-2"></i>
              <span>未成年人禁止充值，平台已设置防沉迷系统</span>
            </li>
          </ul>
        </div>
        
        <div class="bg-card rounded-lg p-6">
          <h3 class="text-xl font-semibold text-white mb-4 flex items-center">
            <i class="fas fa-chart-pie text-primary mr-2"></i>开箱概率公示
          </h3>
          <div class="space-y-4 text-gray-300">
            <div>
              <div class="flex justify-between mb-1">
                <span>消费级 (蓝色)</span>
                <span>45%</span>
              </div>
              <div class="w-full bg-gray-700 rounded-full h-2">
                <div class="bg-blue-500 h-2 rounded-full" style="width: 45%"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between mb-1">
                <span>工业级 (紫色)</span>
                <span>35%</span>
              </div>
              <div class="w-full bg-gray-700 rounded-full h-2">
                <div class="bg-purple-500 h-2 rounded-full" style="width: 35%"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between mb-1">
                <span>军规级 (绿色)</span>
                <span>15%</span>
              </div>
              <div class="w-full bg-gray-700 rounded-full h-2">
                <div class="bg-green-500 h-2 rounded-full" style="width: 15%"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between mb-1">
                <span>受限级 (黄色)</span>
                <span>3%</span>
              </div>
              <div class="w-full bg-gray-700 rounded-full h-2">
                <div class="bg-yellow-500 h-2 rounded-full" style="width: 3%"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between mb-1">
                <span>罕见级 (青色)</span>
                <span>1.5%</span>
              </div>
              <div class="w-full bg-gray-700 rounded-full h-2">
                <div class="bg-cyan-500 h-2 rounded-full" style="width: 1.5%"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between mb-1">
                <span>珍稀级 (红色)</span>
                <span>0.5%</span>
              </div>
              <div class="w-full bg-gray-700 rounded-full h-2">
                <div class="bg-red-500 h-2 rounded-full" style="width: 0.5%"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
    
    <!-- 页脚 -->
    <footer class="border-t border-gray-700 py-8 text-center text-gray-500 text-sm">
      <p>© 2023 CSGO开箱站. 保留所有权利.</p>
      <p class="mt-2">本平台仅提供虚拟物品开箱体验，不涉及任何赌博行为</p>
      <div class="mt-4 flex justify-center space-x-6">
        <a href="#" class="hover:text-primary transition"><i class="fab fa-steam text-xl"></i></a>
        <a href="#" class="hover:text-primary transition"><i class="fab fa-discord text-xl"></i></a>
        <a href="#" class="hover:text-primary transition"><i class="fab fa-twitter text-xl"></i></a>
      </div>
    </footer>
  </div>

  <script>
    const { createApp } = Vue;
    
    createApp({
      data() {
        return {
          isLoggedIn: false,
          isLogin: true,
          showLogin: false,
          showDeposit: false,
          isOpening: false,
          itemRevealed: false,
          depositAmount: 50,
          authForm: {
            username: '',
            password: '',
            email: ''
          },
          user: {
            id: 1,
            username: '游客',
            avatar: 'https://picsum.photos/100/100?random=user',
            balance: 0,
            createdAt: '2023-01-01'
          },
          cases: [
            { 
              id: 1, 
              name: '匕首武器箱', 
              price: 20, 
              rarity: '普通', 
              image: 'https://picsum.photos/300/300?random=case1',
              rarityRates: {
                '消费': 45,
                '工业': 35,
                '军规': 10,
                '受限': 5,
                '罕见': 4,
                '珍稀': 1
              }
            },
            { 
              id: 2, 
              name: '幻彩武器箱', 
              price: 15, 
              rarity: '普通', 
              image: 'https://picsum.photos/300/300?random=case2',
              rarityRates: {
                '消费': 45,
                '工业': 35,
                '军规': 10,
                '受限': 5,
                '罕见': 4,
                '珍稀': 1
              }
            },
            { 
              id: 3, 
              name: '光谱武器箱', 
              price: 18, 
              rarity: '普通', 
              image: 'https://picsum.photos/300/300?random=case3',
              rarityRates: {
                '消费': 40,
                '工业': 30,
                '军规': 15,
                '受限': 8,
                '罕见': 6,
                '珍稀': 1
              }
            },
            { 
              id: 4, 
              name: '突围武器箱', 
              price: 25, 
              rarity: '普通', 
              image: 'https://picsum.photos/300/300?random=case4',
              rarityRates: {
                '消费': 35,
                '工业': 25,
                '军规': 20,
                '受限': 10,
                '罕见': 8,
                '珍稀': 2
              }
            }
          ],
          currentCase: {},
          currentItem: {},
          itemsLibrary: [
            { name: '匕首 | 多普勒 (红宝石)', rarity: '珍稀', rarityColor: 'text-red-500', value: 1000, image: 'https://picsum.photos/300/300?random=item1' },
            { name: 'AK-47 | 火神', rarity: '罕见', rarityColor: 'text-cyan-400', value: 300, image: 'https://picsum.photos/300/300?random=item2' },
            { name: 'M4A4 | 咆哮', rarity: '军规', rarityColor: 'text-gray-400', value: 150, image: 'https://picsum.photos/300/300?random=item3' },
            { name: '沙漠之鹰 | 印花集', rarity: '受限', rarityColor: 'text-yellow-400', value: 80, image: 'https://picsum.photos/300/300?random=item4' },
            { name: '格洛克-18 | 水灵', rarity: '工业', rarityColor: 'text-purple-400', value: 20, image: 'https://picsum.photos/300/300?random=item5' },
            { name: 'USP-S | 消音版 | 闪回', rarity: '消费', rarityColor: 'text-blue-400', value: 5, image: 'https://picsum.photos/300/300?random=item6' }
          ],
          publicHistory: [
            { 
              username: '玩家1', 
              avatar: 'https://picsum.photos/100/100?random=u1',
              caseName: '匕首武器箱', 
              itemName: 'AK-47 | 火神', 
              itemRarity: '罕见', 
              value: 300,
              time: '2023-10-25 14:30' 
            },
            { 
              username: '玩家2', 
              avatar: 'https://picsum.photos/100/100?random=u2',
              caseName: '幻彩武器箱', 
              itemName: '格洛克-18 | 水灵', 
              itemRarity: '工业', 
              value: 20,
              time: '2023-10-25 14:25' 
            },
            { 
              username: '玩家3', 
              avatar: 'https://picsum.photos/100/100?random=u3',
              caseName: '突围武器箱', 
              itemName: '匕首 | 多普勒 (红宝石)', 
              itemRarity: '珍稀', 
              value: 1000,
              time: '2023-10-25 14:20' 
            }
          ]
        }
      },
      methods: {
        // 切换登录/注册模式
        toggleLoginRegister() {
          this.isLogin = !this.isLogin;
        },
        
        // 处理认证
        handleAuth() {
          if (this.isLogin) {
            // 模拟登录
            this.isLoggedIn = true;
            this.user.username = this.authForm.username;
            this.showLogin = false;
          } else {
            // 模拟注册
            this.isLoggedIn = true;
            this.user.username = this.authForm.username;
            this.user.email = this.authForm.email;
            this.user.balance = 50; // 注册送50余额
            this.showLogin = false;
          }
        },
        
        // 退出登录
        logout() {
          this.isLoggedIn = false;
          this.user = {
            id: 1,
            username: '游客',
            avatar: 'https://picsum.photos/100/100?random=user',
            balance: 0,
            createdAt: '2023-01-01'
          };
        },
        
        // 打开箱子
        openCase(caseItem) {
          if (!this.isLoggedIn) {
            this.showLogin = true;
            return;
          }
          
          if (this.user.balance < caseItem.price) {
            alert('余额不足，请先充值');
            return;
          }
          
          this.currentCase = caseItem;
          this.isOpening = true;
          this.itemRevealed = false;
          this.currentItem = {};
          
          // 扣除余额
          this.user.balance -= caseItem.price;
          
          // 模拟开箱动画延迟
          setTimeout(() => {
            this.revealItem();
          }, 1000);
        },
        
        // 显示开箱结果
        revealItem() {
          // 获取箱子的概率配置
          const caseRates = this.currentCase.rarityRates;
          const rarityList = Object.keys(caseRates);
          const rates = Object.values(caseRates);
          
          // 生成随机数确定稀有度 (加权随机)
          const randomNum = Math.random() * 100;
          let cumulative = 0;
          let selectedRarity = '';
          
          for (let i = 0; i < rarityList.length; i++) {
            cumulative += rates[i];
            if (randomNum <= cumulative) {
              selectedRarity = rarityList[i];
              break;
            }
          }
          
          // 根据稀有度筛选物品
          const possibleItems = this.itemsLibrary.filter(item => item.rarity === selectedRarity);
          const randomItem = possibleItems[Math.floor(Math.random() * possibleItems.length)];
          
          this.currentItem = randomItem;
          this.itemRevealed = true;
          
          // 记录开箱历史 (简化版，实际应发送到后端存储)
          this.publicHistory.unshift({
            username: this.user.username,
            avatar: this.user.avatar,
            caseName: this.currentCase.name,
            itemName: randomItem.name,
            itemRarity: randomItem.rarity,
            value: randomItem.value,
            time: this.formatDate(new Date())
          });
        },
        
        // 加入库存
        addtoInventory() {
          this.isOpening = false;
          alert(`恭喜获得 ${this.currentItem.name}！已加入您的库存`);
        },
        
        // 关闭开箱
        closeOpening() {
          if (!this.itemRevealed) {
            // 未开箱成功，退还余额
            this.user.balance += this.currentCase.price;
          }
          this.isOpening = false;
        },
        
        // 处理充值
        processDeposit() {
          if (this.depositAmount < 10) {
            alert('充值金额至少为10美元');
            return;
          }
          
          // 模拟充值 (实际应跳转到支付页面)
          const bonus = this.user.balance === 0 ? this.depositAmount * 0.1 : 0; // 首充10%奖励
          this.user.balance += this.depositAmount + bonus;
          this.showDeposit = false;
          
          alert(`充值 ${this.depositAmount} 美元成功，获得 ${bonus} 美元奖励，当前余额: ${this.user.balance} 美元`);
        },
        
        // 格式化日期
        formatDate(date) {
          const year = date.getFullYear();
          const month = String(date.getMonth() + 1).padStart(2, '0');
          const day = String(date.getDate()).padStart(2, '0');
          const hours = String(date.getHours()).padStart(2, '0');
          const minutes = String(date.getMinutes()).padStart(2, '0');
          return `${year}-${month}-${day} ${hours}:${minutes}`;
        }
      }
    }).mount('#app');
  </script>
</body>
</html>
