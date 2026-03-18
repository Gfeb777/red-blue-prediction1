<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>红蓝游戏预测分析工具</title>
    <!-- Tailwind CSS v3 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome -->
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.8/dist/chart.umd.min.js"></script>
    <!-- 统一的 Tailwind 配置 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#ef4444', // 红色主色调
                        secondary: '#3b82f6', // 蓝色主色调
                        neutral: '#6b7280', // 中性灰
                        dark: '#1f2937', // 深色背景
                        light: '#f9fafb', // 浅色背景
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif'],
                    },
                    animation: {
                        'pulse-light': 'pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite',
                    }
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .ball-red {
                @apply bg-primary text-white rounded-full w-10 h-10 flex items-center justify-center font-bold shadow-md hover:shadow-lg transition-all;
            }
            .ball-blue {
                @apply bg-secondary text-white rounded-full w-10 h-10 flex items-center justify-center font-bold shadow-md hover:shadow-lg transition-all;
            }
            .ball-predicted-red {
                @apply bg-primary/70 text-white rounded-full w-10 h-10 flex items-center justify-center font-bold border-2 border-dashed border-white shadow-md animate-pulse-light;
            }
            .ball-predicted-blue {
                @apply bg-secondary/70 text-white rounded-full w-10 h-10 flex items-center justify-center font-bold border-2 border-dashed border-white shadow-md animate-pulse-light;
            }
            .btn-primary {
                @apply bg-primary hover:bg-primary/90 text-white font-bold py-2 px-4 rounded transition-colors;
            }
            .btn-secondary {
                @apply bg-secondary hover:bg-secondary/90 text-white font-bold py-2 px-4 rounded transition-colors;
            }
            .btn-neutral {
                @apply bg-neutral hover:bg-neutral/90 text-white font-bold py-2 px-4 rounded transition-colors;
            }
            .input-number {
                @apply w-12 h-12 text-center text-lg font-bold border-2 border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-primary/50 focus:border-primary;
            }
            .history-item {
                @apply p-3 border-b border-gray-200 hover:bg-gray-50 transition-colors;
            }
            .tab-active {
                @apply border-b-2 border-primary text-primary;
            }
            .chart-container {
                @apply bg-white rounded-lg shadow-md p-4 h-64;
            }
        }
    </style>
</head>
<body class="bg-light min-h-screen">
    <!-- 顶部导航栏 -->
    <header class="bg-white shadow-md">
        <div class="container mx-auto px-4 py-4 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <div class="ball-red">33</div>
                <h1 class="text-2xl font-bold text-dark">红蓝游戏预测分析工具</h1>
                <div class="ball-blue">16</div>
            </div>
            <div class="flex space-x-4">
                <button id="helpBtn" class="btn-neutral flex items-center">
                    <i class="fa fa-question-circle mr-2"></i>帮助
                </button>
                <button id="themeToggle" class="btn-neutral flex items-center">
                    <i class="fa fa-moon-o mr-2"></i>切换主题
                </button>
            </div>
        </div>
    </header>

    <!-- 主要内容区 -->
    <main class="container mx-auto px-4 py-8">
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- 左侧：历史记录输入区 -->
            <div class="lg:col-span-1">
                <div class="bg-white rounded-lg shadow-md p-6">
                    <h2 class="text-xl font-bold mb-4 flex items-center">
                        <i class="fa fa-history mr-2 text-primary"></i>历史记录
                    </h2>
                    
                    <!-- 输入新记录 -->
                    <div class="mb-6 p-4 bg-gray-50 rounded-lg">
                        <h3 class="text-lg font-semibold mb-3">添加新记录</h3>
                        <div class="mb-4">
                            <label class="block text-sm font-medium text-gray-700 mb-2">开奖结果</label>
                            <div class="flex items-center justify-center space-x-8 mb-4">
                                <button id="redResultBtn" class="w-20 h-20 rounded-full bg-primary/20 border-2 border-primary/50 flex items-center justify-center text-2xl font-bold text-primary hover:bg-primary/30 transition-colors">
                                    红
                                </button>
                                <button id="blueResultBtn" class="w-20 h-20 rounded-full bg-secondary/20 border-2 border-secondary/50 flex items-center justify-center text-2xl font-bold text-secondary hover:bg-secondary/30 transition-colors">
                                    蓝
                                </button>
                            </div>
                            <div class="text-center text-sm text-gray-500" id="selectedResult">
                                请选择开奖结果
                            </div>
                        </div>
                        <div class="flex space-x-2">
                            <button id="addRecordBtn" class="btn-primary flex-1">
                                <i class="fa fa-plus mr-1"></i>添加记录
                            </button>
                            <button id="clearInputsBtn" class="btn-neutral">
                                <i class="fa fa-eraser mr-1"></i>清空
                            </button>
                        </div>
                    </div>
                    
                    <!-- 历史记录列表 -->
                    <div>
                        <div class="flex justify-between items-center mb-3">
                            <h3 class="text-lg font-semibold">历史记录列表</h3>
                            <div class="flex space-x-2">
                                <button id="undoBtn" class="btn-neutral text-sm py-1 px-2" disabled>
                                    <i class="fa fa-undo mr-1"></i>回退
                                </button>
                                <button id="clearHistoryBtn" class="btn-neutral text-sm py-1 px-2" disabled>
                                    <i class="fa fa-trash mr-1"></i>清除全部
                                </button>
                            </div>
                        </div>
                        <div id="historyList" class="max-h-96 overflow-y-auto border rounded-lg">
                            <div class="text-center text-gray-500 py-8">暂无历史记录</div>
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- 右侧：分析与预测区 -->
            <div class="lg:col-span-2">
                <div class="bg-white rounded-lg shadow-md p-6">
                    <h2 class="text-xl font-bold mb-4 flex items-center">
                        <i class="fa fa-line-chart mr-2 text-secondary"></i>分析与预测
                    </h2>
                    
                    <!-- 预测结果 -->
                    <div class="mb-8 p-6 bg-gradient-to-r from-primary/10 to-secondary/10 rounded-lg">
                        <h3 class="text-lg font-semibold mb-4 text-center">下一期预测结果</h3>
                        <div class="flex flex-col items-center">
                            <div class="flex flex-wrap justify-center gap-8 mb-6">
                                <div id="predictedRedBalls" class="flex flex-wrap justify-center gap-3">
                                    <div class="ball-predicted-red text-xl">?</div>
                                </div>
                                <div id="predictedBlueBall" class="ball-predicted-blue text-xl">?</div>
                            </div>
                            <div class="text-center">
                                <button id="generatePredictionBtn" class="btn-secondary" disabled>
                                    <i class="fa fa-magic mr-2"></i>生成预测
                                </button>
                                <p id="predictionInfo" class="text-sm text-gray-500 mt-2">
                                    请先添加至少3条历史记录以生成预测
                                </p>
                            </div>
                        </div>
                    </div>
                    
                    <!-- 分析图表 -->
                    <div>
                        <div class="border-b border-gray-200">
                            <ul class="flex flex-wrap -mb-px text-sm font-medium text-center" id="chartTabs" role="tablist">
                                <li class="mr-2" role="presentation">
                                    <button class="inline-block p-4 tab-active" id="frequency-tab" data-tab="frequency" type="button" role="tab">
                                        <i class="fa fa-bar-chart mr-1"></i>出现频率
                                    </button>
                                </li>
                                <li class="mr-2" role="presentation">
                                    <button class="inline-block p-4 text-gray-500 hover:text-gray-700" id="distribution-tab" data-tab="distribution" type="button" role="tab">
                                        <i class="fa fa-pie-chart mr-1"></i>分布情况
                                    </button>
                                </li>
                                <li role="presentation">
                                    <button class="inline-block p-4 text-gray-500 hover:text-gray-700" id="trend-tab" data-tab="trend" type="button" role="tab">
                                        <i class="fa fa-area-chart mr-1"></i>趋势分析
                                    </button>
                                </li>
                            </ul>
                        </div>
                        <div id="chartContent" class="py-4">
                            <div id="frequency-chart" class="chart-container">
                                <canvas id="frequencyChart"></canvas>
                            </div>
                            <div id="distribution-chart" class="chart-container hidden">
                                <canvas id="distributionChart"></canvas>
                            </div>
                            <div id="trend-chart" class="chart-container hidden">
                                <canvas id="trendChart"></canvas>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- 帮助模态框 -->
    <div id="helpModal" class="fixed inset-0 bg-black bg-opacity-50 z-50 flex items-center justify-center hidden">
        <div class="bg-white rounded-lg shadow-xl max-w-2xl w-full mx-4 max-h-[80vh] overflow-y-auto">
            <div class="p-6">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-bold">使用帮助</h2>
                    <button id="closeHelpBtn" class="text-gray-500 hover:text-gray-700">
                        <i class="fa fa-times text-xl"></i>
                    </button>
                </div>
                <div class="space-y-4">
                    <div>
                        <h3 class="text-lg font-semibold text-primary">添加历史记录</h3>
                        <p>在左侧点击"红"或"蓝"按钮选择开奖结果，然后点击"添加记录"按钮保存。</p>
                    </div>
                    <div>
                        <h3 class="text-lg font-semibold text-secondary">生成预测</h3>
                        <p>添加至少3条历史记录后，可以点击"生成预测"按钮获取下一期的预测结果。系统会基于历史数据的模式和趋势进行智能分析。</p>
                    </div>
                    <div>
                        <h3 class="text-lg font-semibold text-neutral">数据分析</h3>
                        <p>右侧图表区域提供了三种不同的数据分析视图：</p>
                        <ul class="list-disc pl-5 mt-2">
                            <li><strong>出现频率</strong>：显示每个号码在历史记录中的出现次数</li>
                            <li><strong>分布情况</strong>：展示号码在不同区间的分布比例</li>
                            <li><strong>趋势分析</strong>：分析号码的变化趋势和规律</li>
                        </ul>
                    </div>
                    <div>
                        <h3 class="text-lg font-semibold text-neutral">记录管理</h3>
                        <p>可以使用"回退"按钮删除最近添加的记录，或使用"清除全部"按钮删除所有历史记录。</p>
                    </div>
                    <div class="bg-yellow-50 p-4 rounded-lg">
                        <h3 class="text-lg font-semibold text-yellow-700">免责声明</h3>
                        <p class="text-yellow-700">本工具仅用于数据分析和参考，不保证预测结果的准确性。请理性对待预测结果，购买彩票应量力而行。</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 全局变量
        let historyRecords = [];
        let frequencyChart = null;
        let distributionChart = null;
        let trendChart = null;
        let isDarkMode = false;

        // DOM 元素
        const redBallsInput = document.getElementById('redBallsInput');
        const blueBallInput = document.getElementById('blueBallInput');
        const addRecordBtn = document.getElementById('addRecordBtn');
        const clearInputsBtn = document.getElementById('clearInputsBtn');
        const historyList = document.getElementById('historyList');
        const undoBtn = document.getElementById('undoBtn');
        const clearHistoryBtn = document.getElementById('clearHistoryBtn');
        const generatePredictionBtn = document.getElementById('generatePredictionBtn');
        const predictionInfo = document.getElementById('predictionInfo');
        const predictedRedBalls = document.getElementById('predictedRedBalls');
        const predictedBlueBall = document.getElementById('predictedBlueBall');
        const helpBtn = document.getElementById('helpBtn');
        const helpModal = document.getElementById('helpModal');
        const closeHelpBtn = document.getElementById('closeHelpBtn');
        const themeToggle = document.getElementById('themeToggle');
        const chartTabs = document.getElementById('chartTabs');

        // 初始化
        document.addEventListener('DOMContentLoaded', function() {
            initializeEventListeners();
            updateUIState();
        });

        // 初始化事件监听器
        function initializeEventListeners() {
            // 添加记录按钮
            addRecordBtn.addEventListener('click', addRecord);
            
            // 清空输入按钮
            clearInputsBtn.addEventListener('click', clearInputs);
            
            // 回退按钮
            undoBtn.addEventListener('click', undoRecord);
            
            // 清除历史按钮
            clearHistoryBtn.addEventListener('click', clearHistory);
            
            // 生成预测按钮
            generatePredictionBtn.addEventListener('click', generatePrediction);
            
            // 帮助按钮
            helpBtn.addEventListener('click', () => {
                helpModal.classList.remove('hidden');
            });
            
            // 关闭帮助按钮
            closeHelpBtn.addEventListener('click', () => {
                helpModal.classList.add('hidden');
            });
            
            // 点击模态框外部关闭
            helpModal.addEventListener('click', (e) => {
                if (e.target === helpModal) {
                    helpModal.classList.add('hidden');
                }
            });
            
            // 主题切换按钮
            themeToggle.addEventListener('click', toggleTheme);
            
            // 图表标签切换
            chartTabs.addEventListener('click', (e) => {
                if (e.target.tagName === 'BUTTON') {
                    const tabId = e.target.getAttribute('data-tab');
                    switchChartTab(tabId);
                }
            });
            
            // 结果选择按钮
            document.getElementById('redResultBtn').addEventListener('click', () => {
                selectedResult = 'red';
                document.getElementById('selectedResult').textContent = '已选择: 红';
                document.getElementById('redResultBtn').classList.remove('bg-primary/20', 'text-primary');
                document.getElementById('redResultBtn').classList.add('bg-primary', 'text-white');
                document.getElementById('blueResultBtn').classList.remove('bg-secondary', 'text-white');
                document.getElementById('blueResultBtn').classList.add('bg-secondary/20', 'text-secondary');
            });
            
            document.getElementById('blueResultBtn').addEventListener('click', () => {
                selectedResult = 'blue';
                document.getElementById('selectedResult').textContent = '已选择: 蓝';
                document.getElementById('blueResultBtn').classList.remove('bg-secondary/20', 'text-secondary');
                document.getElementById('blueResultBtn').classList.add('bg-secondary', 'text-white');
                document.getElementById('redResultBtn').classList.remove('bg-primary', 'text-white');
                document.getElementById('redResultBtn').classList.add('bg-primary/20', 'text-primary');
            });
        }

        // 验证输入
        function validateInput(e) {
            const input = e.target;
            const value = parseInt(input.value);
            const min = parseInt(input.min);
            const max = parseInt(input.max);
            
            if (isNaN(value) || value < min || value > max) {
                input.classList.add('border-red-500');
            } else {
                input.classList.remove('border-red-500');
                input.classList.add('border-green-500');
                
                // 自动聚焦到下一个输入框
                const inputs = input.parentElement.querySelectorAll('input');
                const currentIndex = Array.from(inputs).indexOf(input);
                if (currentIndex < inputs.length - 1) {
                    inputs[currentIndex + 1].focus();
                } else if (input.parentElement === redBallsInput) {
                    // 如果是红球最后一个输入框，聚焦到蓝球输入框
                    blueBallInput.querySelector('input').focus();
                }
            }
        }

        // 全局变量
        let selectedResult = null;
        
        // 添加记录
        function addRecord() {
            // 检查是否选择了结果
            if (selectedResult === null) {
                alert('请选择开奖结果（红或蓝）');
                return;
            }
            
            // 创建记录对象
            const record = {
                id: Date.now(),
                result: selectedResult,
                timestamp: new Date().toLocaleString()
            };
            
            // 添加到历史记录
            historyRecords.push(record);
            
            // 更新UI
            updateHistoryList();
            clearInputs();
            updateUIState();
            updateCharts();
            
            // 显示成功提示
            showToast('记录添加成功');
        }

        // 清空输入
        function clearInputs() {
            selectedResult = null;
            document.getElementById('selectedResult').textContent = '请选择开奖结果';
            document.getElementById('redResultBtn').classList.remove('bg-primary', 'text-white');
            document.getElementById('redResultBtn').classList.add('bg-primary/20', 'text-primary');
            document.getElementById('blueResultBtn').classList.remove('bg-secondary', 'text-white');
            document.getElementById('blueResultBtn').classList.add('bg-secondary/20', 'text-secondary');
        }

        // 回退记录
        function undoRecord() {
            if (historyRecords.length > 0) {
                historyRecords.pop();
                updateHistoryList();
                updateUIState();
                updateCharts();
                showToast('已删除最近的记录');
            }
        }

        // 清除历史
        function clearHistory() {
            if (historyRecords.length > 0) {
                if (confirm('确定要清除所有历史记录吗？')) {
                    historyRecords = [];
                    updateHistoryList();
                    updateUIState();
                    updateCharts();
                    showToast('历史记录已清除');
                }
            }
        }

        // 更新历史记录列表
        function updateHistoryList() {
            if (historyRecords.length === 0) {
                historyList.innerHTML = '<div class="text-center text-gray-500 py-8">暂无历史记录</div>';
                return;
            }
            
            let html = '';
            historyRecords.forEach((record, index) => {
                const resultClass = record.result === 'red' ? 'ball-red' : 'ball-blue';
                const resultText = record.result === 'red' ? '红' : '蓝';
                
                html += `
                    <div class="history-item flex items-center justify-between">
                        <div class="flex items-center">
                            <span class="font-medium mr-3">第 ${historyRecords.length - index} 期</span>
                            <div class="${resultClass} text-xl">${resultText}</div>
                        </div>
                        <span class="text-sm text-gray-500">${record.timestamp}</span>
                    </div>
                `;
            });
            
            historyList.innerHTML = html;
        }

        // 更新UI状态
        function updateUIState() {
            // 更新按钮状态
            undoBtn.disabled = historyRecords.length === 0;
            clearHistoryBtn.disabled = historyRecords.length === 0;
            generatePredictionBtn.disabled = historyRecords.length < 3;
            
            // 更新预测信息
            if (historyRecords.length < 3) {
                predictionInfo.textContent = `请先添加至少3条历史记录以生成预测（已添加 ${historyRecords.length} 条）`;
            } else {
                predictionInfo.textContent = `基于 ${historyRecords.length} 条历史记录进行分析`;
            }
            
            // 更新按钮样式
            [undoBtn, clearHistoryBtn, generatePredictionBtn].forEach(btn => {
                if (btn.disabled) {
                    btn.classList.add('opacity-50', 'cursor-not-allowed');
                } else {
                    btn.classList.remove('opacity-50', 'cursor-not-allowed');
                }
            });
        }

        // 生成预测
        function generatePrediction() {
            if (historyRecords.length < 3) {
                alert('请先添加至少3条历史记录');
                return;
            }
            
            // 显示加载状态
            predictedRedBalls.innerHTML = '<div class="ball-predicted-red"><i class="fa fa-spinner fa-spin"></i></div>';
            predictedBlueBall.innerHTML = '<div class="ball-predicted-blue"><i class="fa fa-spinner fa-spin"></i></div>';
            generatePredictionBtn.disabled = true;
            predictionInfo.textContent = '正在分析数据...';
            
            // 模拟分析过程
            setTimeout(() => {
                // 获取预测结果
                const prediction = calculatePrediction();
                
                // 更新UI显示预测结果
                if (prediction.result === 'red') {
                    predictedRedBalls.innerHTML = '<div class="ball-predicted-red text-xl">红</div>';
                    predictedBlueBall.innerHTML = '<div class="ball-predicted-blue/50 text-xl">蓝</div>';
                } else {
                    predictedRedBalls.innerHTML = '<div class="ball-predicted-red/50 text-xl">红</div>';
                    predictedBlueBall.innerHTML = '<div class="ball-predicted-blue text-xl">蓝</div>';
                }
                
                // 恢复按钮状态
                generatePredictionBtn.disabled = false;
                predictionInfo.textContent = `预测结果: ${prediction.result === 'red' ? '红' : '蓝'} (置信度: ${prediction.confidence}%) - ${new Date().toLocaleTimeString()}`;
                
                // 显示成功提示
                showToast('预测结果已生成');
            }, 1500);
        }

        // 计算预测结果（核心算法）
        function calculatePrediction() {
            // 收集所有历史结果数据
            const allResults = historyRecords.map(record => record.result);
            
            // 计算红和蓝的出现频率
            const redCount = allResults.filter(result => result === 'red').length;
            const blueCount = allResults.filter(result => result === 'blue').length;
            
            // 计算最近的趋势
            const recentResults = allResults.slice(-5); // 最近5期
            const recentRedCount = recentResults.filter(result => result === 'red').length;
            const recentBlueCount = recentResults.filter(result => result === 'blue').length;
            
            // 计算概率
            const totalCount = redCount + blueCount;
            const redProbability = totalCount > 0 ? redCount / totalCount : 0.5;
            const blueProbability = totalCount > 0 ? blueCount / totalCount : 0.5;
            
            // 考虑最近趋势调整概率
            const trendFactor = 0.3; // 趋势权重
            const adjustedRedProbability = redProbability * (1 - trendFactor) + (recentRedCount / 5) * trendFactor;
            const adjustedBlueProbability = blueProbability * (1 - trendFactor) + (recentBlueCount / 5) * trendFactor;
            
            // 预测结果
            let prediction;
            
            // 使用加权随机选择
            const random = Math.random();
            if (random < adjustedRedProbability) {
                prediction = 'red';
            } else {
                prediction = 'blue';
            }
            
            // 计算置信度
            const confidence = Math.max(adjustedRedProbability, adjustedBlueProbability) * 100;
            
            return {
                result: prediction,
                confidence: Math.round(confidence)
            };
        }

        // 初始化图表
        function initializeCharts() {
            // 初始化频率图表
            const frequencyCtx = document.getElementById('frequencyChart').getContext('2d');
            frequencyChart = new Chart(frequencyCtx, {
                type: 'bar',
                data: {
                    labels: ['红', '蓝'],
                    datasets: [{
                        label: '出现频率',
                        data: [0, 0],
                        backgroundColor: ['rgba(239, 68, 68, 0.7)', 'rgba(59, 130, 246, 0.7)'],
                        borderColor: ['rgba(239, 68, 68, 1)', 'rgba(59, 130, 246, 1)'],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                stepSize: 1
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: true,
                            position: 'top'
                        }
                    }
                }
            });
            
            // 初始化分布图表
            const distributionCtx = document.getElementById('distributionChart').getContext('2d');
            distributionChart = new Chart(distributionCtx, {
                type: 'pie',
                data: {
                    labels: ['红', '蓝'],
                    datasets: [{
                        data: [0, 0],
                        backgroundColor: ['rgba(239, 68, 68, 0.7)', 'rgba(59, 130, 246, 0.7)'],
                        borderColor: ['rgba(239, 68, 68, 1)', 'rgba(59, 130, 246, 1)'],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'right'
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const label = context.label || '';
                                    const value = context.raw || 0;
                                    const total = context.dataset.data.reduce((a, b) => a + b, 0);
                                    const percentage = total > 0 ? Math.round((value / total) * 100) : 0;
                                    return `${label}: ${value} (${percentage}%)`;
                                }
                            }
                        }
                    }
                }
            });
            
            // 初始化趋势图表
            const trendCtx = document.getElementById('trendChart').getContext('2d');
            trendChart = new Chart(trendCtx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: '开奖结果（红=1，蓝=0）',
                        data: [],
                        borderColor: 'rgba(75, 192, 192, 1)',
                        backgroundColor: 'rgba(75, 192, 192, 0.1)',
                        borderWidth: 2,
                        fill: true,
                        tension: 0.4,
                        pointRadius: 5,
                        pointBackgroundColor: function(context) {
                            const value = context.dataset.data[context.dataIndex];
                            return value === 1 ? 'rgba(239, 68, 68, 1)' : 'rgba(59, 130, 246, 1)';
                        }
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true,
                            max: 1,
                            stepSize: 1,
                            ticks: {
                                callback: function(value) {
                                    return value === 1 ? '红' : (value === 0 ? '蓝' : '');
                                }
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            position: 'top'
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    const value = context.raw;
                                    return value === 1 ? '红' : '蓝';
                                }
                            }
                        }
                    }
                }
            });
        }

        // 更新图表数据
        function updateCharts() {
            // 如果还没有初始化图表，则先初始化
            if (!frequencyChart) {
                initializeCharts();
            }
            
            // 收集数据
            const allResults = historyRecords.map(record => record.result);
            const redCount = allResults.filter(result => result === 'red').length;
            const blueCount = allResults.filter(result => result === 'blue').length;
            
            // 更新频率图表
            frequencyChart.data.labels = ['红', '蓝'];
            frequencyChart.data.datasets[0].data = [redCount, blueCount];
            frequencyChart.data.datasets[0].backgroundColor = ['rgba(239, 68, 68, 0.7)', 'rgba(59, 130, 246, 0.7)'];
            frequencyChart.data.datasets[0].borderColor = ['rgba(239, 68, 68, 1)', 'rgba(59, 130, 246, 1)'];
            frequencyChart.update();
            
            // 更新分布图表
            distributionChart.data.labels = ['红', '蓝'];
            distributionChart.data.datasets[0].data = [redCount, blueCount];
            distributionChart.data.datasets[0].backgroundColor = ['rgba(239, 68, 68, 0.7)', 'rgba(59, 130, 246, 0.7)'];
            distributionChart.data.datasets[0].borderColor = ['rgba(239, 68, 68, 1)', 'rgba(59, 130, 246, 1)'];
            distributionChart.update();
            
            // 更新趋势图表
            const labels = historyRecords.map((_, index) => `第${index + 1}期`);
            const resultValues = allResults.map(result => result === 'red' ? 1 : 0);
            
            trendChart.data.labels = labels;
            trendChart.data.datasets[0].label = '开奖结果（红=1，蓝=0）';
            trendChart.data.datasets[0].data = resultValues;
            trendChart.data.datasets[0].borderColor = 'rgba(75, 192, 192, 1)';
            trendChart.data.datasets[0].backgroundColor = 'rgba(75, 192, 192, 0.1)';
            
            // 移除第二个数据集（蓝球数据）
            if (trendChart.data.datasets.length > 1) {
                trendChart.data.datasets.splice(1);
            }
            
            trendChart.options.scales.y.ticks = {
                stepSize: 1,
                callback: function(value) {
                    return value === 1 ? '红' : (value === 0 ? '蓝' : '');
                }
            };
            
            trendChart.update();
        }

        // 切换图表标签
        function switchChartTab(tabId) {
            // 更新标签样式
            document.querySelectorAll('#chartTabs button').forEach(tab => {
                if (tab.getAttribute('data-tab') === tabId) {
                    tab.classList.add('tab-active');
                    tab.classList.remove('text-gray-500', 'hover:text-gray-700');
                } else {
                    tab.classList.remove('tab-active');
                    tab.classList.add('text-gray-500', 'hover:text-gray-700');
                }
            });
            
            // 显示对应的图表
            document.querySelectorAll('#chartContent > div').forEach(chart => {
                chart.classList.add('hidden');
            });
            
            document.getElementById(`${tabId}-chart`).classList.remove('hidden');
        }

        // 切换主题
        function toggleTheme() {
            isDarkMode = !isDarkMode;
            const body = document.body;
            
            if (isDarkMode) {
                body.classList.add('bg-gray-900', 'text-white');
                body.classList.remove('bg-light');
                themeToggle.innerHTML = '<i class="fa fa-sun-o mr-2"></i>切换主题';
                
                // 更新图表主题
                updateChartsTheme('dark');
            } else {
                body.classList.remove('bg-gray-900', 'text-white');
                body.classList.add('bg-light');
                themeToggle.innerHTML = '<i class="fa fa-moon-o mr-2"></i>切换主题';
                
                // 更新图表主题
                updateChartsTheme('light');
            }
        }

        // 更新图表主题
        function updateChartsTheme(theme) {
            const textColor = theme === 'dark' ? '#ffffff' : '#000000';
            const gridColor = theme === 'dark' ? 'rgba(255, 255, 255, 0.1)' : 'rgba(0, 0, 0, 0.1)';
            
            // 更新频率图表
            if (frequencyChart) {
                frequencyChart.options.scales.x.ticks.color = textColor;
                frequencyChart.options.scales.y.ticks.color = textColor;
                frequencyChart.options.scales.x.grid.color = gridColor;
                frequencyChart.options.scales.y.grid.color = gridColor;
                frequencyChart.update();
            }
            
            // 更新趋势图表
            if (trendChart) {
                trendChart.options.scales.x.ticks.color = textColor;
                trendChart.options.scales.y.ticks.color = textColor;
                trendChart.options.scales.x.grid.color = gridColor;
                trendChart.options.scales.y.grid.color = gridColor;
                trendChart.update();
            }
        }

        // 显示提示消息
        function showToast(message) {
            // 创建提示元素
            const toast = document.createElement('div');
            toast.className = 'fixed bottom-4 right-4 bg-green-500 text-white px-4 py-2 rounded-lg shadow-lg z-50 transition-opacity duration-300';
            toast.textContent = message;
            
            // 添加到页面
            document.body.appendChild(toast);
            
            // 淡入
            setTimeout(() => {
                toast.classList.add('opacity-90');
            }, 10);
            
            // 淡出并移除
            setTimeout(() => {
                toast.classList.remove('opacity-90');
                toast.classList.add('opacity-0');
                
                setTimeout(() => {
                    document.body.removeChild(toast);
                }, 300);
            }, 2000);
        }
    </script>
</body>
</html>
