<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="theme-color" content="#ffffff">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>HealthEdu App</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Phosphor Icons -->
    <script src="https://unpkg.com/@phosphor-icons/web"></script>
    <!-- HTML2Canvas & JSPDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <!-- Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Sans+Tamil:wght@400;500;600;700&family=Noto+Sans+Devanagari:wght@400;500;600;700&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        primary: { 50: '#eff6ff', 100: '#dbeafe', 500: '#3b82f6', 600: '#2563eb', 900: '#1e3a8a' },
                        surface: '#f8fafc',
                    },
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                        tamil: ['Noto Sans Tamil', 'sans-serif'],
                        hindi: ['Noto Sans Devanagari', 'sans-serif'],
                    },
                    boxShadow: {
                        'soft': '0 4px 20px -2px rgba(0, 0, 0, 0.05)',
                        'nav': '0 -4px 20px rgba(0,0,0,0.03)',
                    }
                }
            }
        }
    </script>

    <style>
        /* Mobile Optimization & Resets */
        body {
            -webkit-tap-highlight-color: transparent;
            -webkit-touch-callout: none;
            overscroll-behavior-y: none;
        }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        .pb-safe { padding-bottom: env(safe-area-inset-bottom); }
        .pt-safe { padding-top: env(safe-area-inset-top); }

        /* Transitions */
        .tab-content {
            display: none;
            animation: fadeIn 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .tab-content.active { display: block; }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(5px); }
            to { opacity: 1; transform: translateY(0); }
        }

        /* Flashcard Scaling Logic for Mobile */
        .flashcard-wrapper-mobile {
            width: 100%;
            overflow: hidden;
            border-radius: 12px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            background: #e2e8f0;
            position: relative;
        }
        
        .flashcard-scaler {
            transform-origin: top left;
        }

        .flashcard-fixed-layout {
            background: white;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            position: relative;
        }

        .image-overlay-tools {
            opacity: 0;
            transition: all 0.2s;
            background: rgba(0,0,0,0.6);
            backdrop-filter: blur(4px);
        }
        .image-slot:hover .image-overlay-tools, .image-slot:active .image-overlay-tools { opacity: 1; }
        
        /* Fullscreen Overlay */
        .preso-overlay { background: #0f172a; }
        
        /* Form Elements */
        .app-input {
            width: 100%;
            padding: 12px 16px;
            background: white;
            border: 1px solid #e2e8f0;
            border-radius: 12px;
            font-size: 16px; /* Prevents iOS zoom */
            transition: all 0.2s;
        }
        .app-input:focus {
            border-color: #3b82f6;
            outline: none;
            box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
        }
    </style>
</head>
<body class="bg-gray-100 text-slate-800 font-sans h-screen w-screen overflow-hidden flex justify-center items-center">

    <!-- Mobile Frame -->
    <div class="relative w-full h-full md:max-w-[420px] md:h-[90vh] md:max-h-[900px] bg-surface md:rounded-[2.5rem] md:shadow-2xl md:border-[8px] md:border-gray-900 overflow-hidden flex flex-col">
        
        <!-- Status Bar (Desktop Visual Only) -->
        <div class="hidden md:flex w-full h-7 bg-white/90 backdrop-blur items-center justify-between px-6 text-[10px] font-bold z-50 absolute top-0 rounded-t-[2rem]">
            <span>9:41</span>
            <div class="flex gap-1.5">
                <i class="ph-fill ph-cell-signal-full"></i>
                <i class="ph-fill ph-wifi-high"></i>
                <i class="ph-fill ph-battery-full"></i>
            </div>
        </div>

        <!-- === HEADER === -->
        <header class="absolute top-0 left-0 right-0 bg-white/90 backdrop-blur-md z-40 border-b border-gray-100 pt-safe transition-all duration-300" id="mainHeader">
            <div class="h-14 px-5 flex items-center justify-between md:mt-5 mt-0">
                <div class="flex items-center gap-2">
                    <div class="w-8 h-8 rounded-full bg-blue-600 flex items-center justify-center text-white font-bold text-lg">H</div>
                    <span class="font-bold text-xl tracking-tight text-slate-900">HealthEdu</span>
                </div>
                <div class="flex items-center gap-3">
                    <span class="text-[10px] font-bold bg-blue-50 text-blue-600 px-2 py-1 rounded-full">PRO</span>
                </div>
            </div>
        </header>

        <!-- === MAIN CONTENT === -->
        <main class="flex-1 overflow-y-auto no-scrollbar pt-safe pb-safe scroll-smooth bg-surface relative" id="scrollContainer">
            <div class="h-20 md:h-24"></div> <!-- Header Spacer -->

            <!-- TAB 1: SETUP (Home) -->
            <section id="tab-home" class="tab-content active px-5 pb-28">
                <h2 class="text-2xl font-bold text-slate-900 mb-2">Create Project</h2>
                <p class="text-sm text-slate-500 mb-6">Configure your health education deck.</p>

                <div class="space-y-5">
                    <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100">
                        <label class="block text-xs font-bold text-slate-500 uppercase mb-2">Topic</label>
                        <input type="text" id="mainTopic" placeholder="e.g. Hand Hygiene" class="app-input mb-4">
                        
                        <label class="block text-xs font-bold text-slate-500 uppercase mb-2">Target Audience</label>
                        <select id="targetAudience" class="app-input appearance-none bg-no-repeat bg-[right_1rem_center]">
                            <option value="general">General Public</option>
                            <option value="children">Children</option>
                            <option value="nursing-students">Nursing Students</option>
                            <option value="elderly">Elderly</option>
                        </select>
                    </div>

                    <div class="bg-white p-4 rounded-2xl shadow-sm border border-slate-100">
                        <label class="block text-xs font-bold text-slate-500 uppercase mb-3">Configuration</label>
                        
                        <div class="flex gap-4 mb-4">
                            <div class="flex-1">
                                <label class="text-xs text-slate-400 mb-1 block">Paper Size</label>
                                <select id="paperSize" class="app-input text-sm py-2">
                                    <option value="a4-landscape">A4 Landscape</option>
                                    <option value="a4-portrait">A4 Portrait</option>
                                    <option value="sq-post">Square (IG)</option>
                                    <option value="story">Mobile Story</option>
                                </select>
                            </div>
                            <div class="flex-1">
                                <label class="text-xs text-slate-400 mb-1 block">Card Count</label>
                                <input type="number" id="numberOfCards" value="5" min="1" max="20" class="app-input text-sm py-2">
                            </div>
                        </div>

                        <label class="text-xs text-slate-400 mb-2 block">Layout (Images per card)</label>
                        <div class="grid grid-cols-4 gap-2">
                            <button onclick="setImgLayout(1, this)" class="layout-btn active bg-blue-50 border-blue-500 text-blue-600 border rounded-lg p-2 text-center text-xs font-bold transition-all" data-val="1">1</button>
                            <button onclick="setImgLayout(2, this)" class="layout-btn bg-white border-slate-200 text-slate-600 border rounded-lg p-2 text-center text-xs font-bold transition-all" data-val="2">2</button>
                            <button onclick="setImgLayout(3, this)" class="layout-btn bg-white border-slate-200 text-slate-600 border rounded-lg p-2 text-center text-xs font-bold transition-all" data-val="3">3</button>
                            <button onclick="setImgLayout(6, this)" class="layout-btn bg-white border-slate-200 text-slate-600 border rounded-lg p-2 text-center text-xs font-bold transition-all" data-val="6">6</button>
                        </div>
                        <input type="hidden" id="imagesPerCard" value="1">
                    </div>

                    <button onclick="proceedToPrompts()" class="w-full bg-blue-600 text-white py-4 rounded-2xl font-bold shadow-lg shadow-blue-500/30 active:scale-95 transition-transform flex items-center justify-center gap-2">
                        Generate Prompts <i class="ph-bold ph-arrow-right"></i>
                    </button>
                </div>
            </section>

            <!-- TAB 2: PROMPTS (AI) -->
            <section id="tab-prompts" class="tab-content px-5 pb-28">
                <h2 class="text-2xl font-bold text-slate-900 mb-2">AI Generator</h2>
                <div class="flex gap-2 mb-4 overflow-x-auto no-scrollbar">
                    <button onclick="copyPrompt('content')" class="whitespace-nowrap px-4 py-2 bg-white border border-slate-200 rounded-lg text-xs font-bold text-slate-600 hover:bg-slate-50 flex items-center gap-2">
                        <i class="ph-bold ph-copy"></i> Copy Content Prompt
                    </button>
                    <button onclick="copyPrompt('images')" class="whitespace-nowrap px-4 py-2 bg-white border border-slate-200 rounded-lg text-xs font-bold text-slate-600 hover:bg-slate-50 flex items-center gap-2">
                        <i class="ph-bold ph-image"></i> Copy Image Prompts
                    </button>
                </div>

                <!-- Hidden prompt containers for copying -->
                <div id="contentPrompt" class="hidden"></div>
                <div id="imagePrompts" class="hidden"></div>

                <div class="bg-white rounded-2xl p-4 shadow-sm border border-slate-200">
                    <label class="block text-xs font-bold text-slate-400 uppercase mb-2">Paste LLM Response</label>
                    <textarea id="generatedContent" class="w-full h-48 bg-slate-50 border border-slate-100 rounded-xl p-3 text-xs font-mono focus:border-blue-500 focus:outline-none resize-none" placeholder="Paste the generated JSON/Text here..."></textarea>
                </div>

                <button onclick="parseContent()" class="mt-4 w-full bg-slate-900 text-white py-4 rounded-2xl font-bold shadow-lg active:scale-95 transition-transform">
                    Create Cards
                </button>
            </section>

            <!-- TAB 3: EDITOR (Deck) -->
            <section id="tab-editor" class="tab-content px-4 pb-28">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-bold text-slate-900">Editor</h2>
                    <div class="flex bg-white rounded-lg p-1 border border-slate-200 shadow-sm">
                        <button onclick="setGlobalLanguage('tamil')" class="lang-btn px-3 py-1 rounded text-[10px] font-bold bg-blue-100 text-blue-700" data-lang="tamil">TA</button>
                        <button onclick="setGlobalLanguage('hindi')" class="lang-btn px-3 py-1 rounded text-[10px] font-bold text-slate-500" data-lang="hindi">HI</button>
                        <button onclick="setGlobalLanguage('english')" class="lang-btn px-3 py-1 rounded text-[10px] font-bold text-slate-500" data-lang="english">EN</button>
                    </div>
                </div>

                <div id="cardsEditor" class="space-y-6">
                    <!-- Cards will be injected here -->
                    <div class="text-center py-10 text-slate-400 text-sm">
                        No cards yet. Go to Home to start.
                    </div>
                </div>
            </section>

            <!-- TAB 4: LIBRARY (History) -->
            <section id="tab-history" class="tab-content px-5 pb-28">
                <h2 class="text-2xl font-bold text-slate-900 mb-6">Saved Projects</h2>
                <div id="historyList" class="space-y-3">
                    <!-- History items -->
                </div>
            </section>
            
            <!-- TAB 5: MENU (Settings/Export) -->
             <section id="tab-menu" class="tab-content px-5 pb-28">
                <h2 class="text-2xl font-bold text-slate-900 mb-6">Tools & Export</h2>
                
                <div class="grid grid-cols-2 gap-4">
                    <button onclick="startPresentation()" class="bg-indigo-600 text-white p-4 rounded-2xl shadow-lg flex flex-col items-center gap-2 active:scale-95 transition-transform">
                        <i class="ph-fill ph-presentation-chart text-3xl"></i>
                        <span class="font-bold text-sm">Present Mode</span>
                    </button>
                    <button onclick="saveToHistory()" class="bg-emerald-600 text-white p-4 rounded-2xl shadow-lg flex flex-col items-center gap-2 active:scale-95 transition-transform">
                        <i class="ph-fill ph-floppy-disk text-3xl"></i>
                        <span class="font-bold text-sm">Save Project</span>
                    </button>
                    <button onclick="exportAllPNG()" class="bg-white border border-slate-200 text-slate-700 p-4 rounded-2xl shadow-sm flex flex-col items-center gap-2 active:scale-95 transition-transform">
                        <i class="ph-fill ph-image text-3xl text-blue-500"></i>
                        <span class="font-bold text-sm">Export Images</span>
                    </button>
                    <button onclick="exportAllPDF()" class="bg-white border border-slate-200 text-slate-700 p-4 rounded-2xl shadow-sm flex flex-col items-center gap-2 active:scale-95 transition-transform">
                        <i class="ph-fill ph-file-pdf text-3xl text-red-500"></i>
                        <span class="font-bold text-sm">Export PDF</span>
                    </button>
                </div>

                <div class="mt-8 bg-blue-50 p-4 rounded-2xl border border-blue-100">
                    <h3 class="font-bold text-blue-900 mb-2 text-sm">About HealthEdu</h3>
                    <p class="text-xs text-blue-700 leading-relaxed">
                        Mobile-first creation suite for medical educators. Supports offline PWA mode.
                        <br>Version 2.5.0 Mobile
                    </p>
                </div>
            </section>
        </main>

        <!-- === BOTTOM NAVIGATION === -->
        <nav class="absolute bottom-0 w-full bg-white/90 backdrop-blur-xl border-t border-gray-100 shadow-nav pb-safe z-50 rounded-b-[2.5rem] md:rounded-b-[2rem]">
            <div class="flex justify-around items-center h-16 px-2">
                
                <button onclick="switchTab('home')" class="nav-item active w-full flex flex-col items-center gap-1 group" data-target="home">
                    <div class="p-1.5 rounded-xl group-[.active]:bg-blue-50 transition-all duration-300">
                        <i class="ph-fill ph-house text-2xl text-slate-400 group-[.active]:text-blue-600"></i>
                    </div>
                </button>

                <button onclick="switchTab('prompts')" class="nav-item w-full flex flex-col items-center gap-1 group" data-target="prompts">
                    <div class="p-1.5 rounded-xl group-[.active]:bg-blue-50 transition-all duration-300">
                        <i class="ph-fill ph-sparkle text-2xl text-slate-400 group-[.active]:text-blue-600"></i>
                    </div>
                </button>

                <button onclick="switchTab('editor')" class="nav-item w-full flex flex-col items-center gap-1 group" data-target="editor">
                    <div class="relative -top-5 shadow-lg shadow-blue-500/30 bg-blue-600 text-white rounded-full w-14 h-14 flex items-center justify-center transform active:scale-95 transition-all border-4 border-surface">
                        <i class="ph-bold ph-cards text-2xl"></i>
                    </div>
                </button>

                <button onclick="switchTab('history')" class="nav-item w-full flex flex-col items-center gap-1 group" data-target="history">
                    <div class="p-1.5 rounded-xl group-[.active]:bg-blue-50 transition-all duration-300">
                        <i class="ph-fill ph-books text-2xl text-slate-400 group-[.active]:text-blue-600"></i>
                    </div>
                </button>

                <button onclick="switchTab('menu')" class="nav-item w-full flex flex-col items-center gap-1 group" data-target="menu">
                    <div class="p-1.5 rounded-xl group-[.active]:bg-blue-50 transition-all duration-300">
                        <i class="ph-fill ph-dots-three-circle text-2xl text-slate-400 group-[.active]:text-blue-600"></i>
                    </div>
                </button>

            </div>
        </nav>

    </div>

    <!-- Presentation Overlay -->
    <div id="fullscreenContainer"></div>

    <script>
        // --- Core State ---
        let config = {
            paperSize: 'a4-landscape',
            width: 1123, 
            height: 794,
            imagesPerCard: 1,
            numberOfCards: 5,
            mainTopic: '',
            targetAudience: 'general'
        };
        
        const sizeMap = {
            'a4-landscape': { w: 1123, h: 794 },
            'a4-portrait': { w: 794, h: 1123 },
            'sq-post': { w: 1080, h: 1080 },
            'story': { w: 1080, h: 1920 }
        };

        let currentCards = [];
        let globalLanguage = 'tamil';
        let currentPresentationIndex = 0;
        let audioEnabled = true;

        // --- Initialization ---
        document.addEventListener('DOMContentLoaded', () => {
            loadHistory();
        });

        // --- UI Logic ---
        function switchTab(tabId) {
            // Update Nav Icons
            document.querySelectorAll('.nav-item').forEach(item => {
                if(item.dataset.target === tabId) {
                    item.classList.add('active');
                    const icon = item.querySelector('i');
                    if(icon) {
                        icon.style.transform = 'scale(0.8)';
                        setTimeout(() => icon.style.transform = 'scale(1)', 150);
                    }
                } else {
                    item.classList.remove('active');
                }
            });

            // Update Content
            document.querySelectorAll('.tab-content').forEach(content => {
                content.classList.remove('active');
            });
            document.getElementById(`tab-${tabId}`).classList.add('active');
            document.getElementById('scrollContainer').scrollTo({ top: 0, behavior: 'smooth' });
        }

        function setImgLayout(val, btn) {
            document.querySelectorAll('.layout-btn').forEach(b => {
                b.classList.remove('active', 'bg-blue-50', 'border-blue-500', 'text-blue-600');
                b.classList.add('bg-white', 'border-slate-200', 'text-slate-600');
            });
            btn.classList.add('active', 'bg-blue-50', 'border-blue-500', 'text-blue-600');
            btn.classList.remove('bg-white', 'border-slate-200', 'text-slate-600');
            document.getElementById('imagesPerCard').value = val;
        }

        function setGlobalLanguage(lang) {
            globalLanguage = lang;
            document.querySelectorAll('.lang-btn').forEach(btn => {
                if(btn.dataset.lang === lang) {
                    btn.className = "lang-btn px-3 py-1 rounded text-[10px] font-bold bg-blue-100 text-blue-700 transition-colors";
                } else {
                    btn.className = "lang-btn px-3 py-1 rounded text-[10px] font-bold text-slate-500 hover:bg-slate-50 transition-colors";
                }
            });
            renderCards();
        }

        // --- Core App Logic (Adapted for Mobile) ---
        function proceedToPrompts() {
            const topic = document.getElementById('mainTopic').value;
            if (!topic.trim()) return showNotification('Enter a topic first', 'error');
            
            const sizeVal = document.getElementById('paperSize').value;
            config = {
                paperSize: sizeVal,
                width: sizeMap[sizeVal].w,
                height: sizeMap[sizeVal].h,
                imagesPerCard: parseInt(document.getElementById('imagesPerCard').value),
                numberOfCards: parseInt(document.getElementById('numberOfCards').value),
                mainTopic: topic,
                targetAudience: document.getElementById('targetAudience').value
            };
            
            generatePrompts();
            switchTab('prompts');
        }

        function generatePrompts() {
            const isPortrait = config.height > config.width;
            const orientation = isPortrait ? 'Portrait' : 'Landscape';
            
            // Content Prompt
            const contentPrompt = `Topic: ${config.mainTopic}
Audience: ${config.targetAudience}
Cards: ${config.numberOfCards}
Images/Card: ${config.imagesPerCard}
Format: ${orientation}

Output strictly for ${config.numberOfCards} cards:
CARD [N]:
MAIN_TITLE_TAMIL: ...
MAIN_TITLE_HINDI: ...
MAIN_TITLE_ENGLISH: ...
(For each image 1 to ${config.imagesPerCard}):
IMAGE_[N]_SUBTITLE_TAMIL: ...
IMAGE_[N]_SUBTITLE_HINDI: ...
IMAGE_[N]_SUBTITLE_ENGLISH: ...
IMAGE_[N]_DESC_TAMIL: ...
IMAGE_[N]_DESC_HINDI: ...
IMAGE_[N]_DESC_ENGLISH: ...

Medically accurate, Indian context.`;

            document.getElementById('contentPrompt').textContent = contentPrompt;

            // Image Prompts
            let imgPromptsHTML = '';
            for (let c = 1; c <= config.numberOfCards; c++) {
                imgPromptsHTML += `Card ${c}: Medical illustration, ${config.mainTopic}, ${orientation}, white background.\n\n`;
            }
            document.getElementById('imagePrompts').textContent = imgPromptsHTML;
        }

        function parseContent() {
            const content = document.getElementById('generatedContent').value;
            if (!content.trim()) return showNotification('Paste generated content first', 'error');

            const cards = [];
            const cardBlocks = content.split(/CARD \d+:/i).filter(block => block.trim());

            cardBlocks.forEach((block, index) => {
                const card = {
                    id: `card-${index}`,
                    type: 'content',
                    layout: `${config.imagesPerCard}-up`,
                    mainTitle: {
                        tamil: extractValue(block, 'MAIN_TITLE_TAMIL'),
                        hindi: extractValue(block, 'MAIN_TITLE_HINDI'),
                        english: extractValue(block, 'MAIN_TITLE_ENGLISH')
                    },
                    images: []
                };

                for (let i = 1; i <= config.imagesPerCard; i++) {
                    card.images.push({
                        subtitle: {
                            tamil: extractValue(block, `IMAGE_${i}_SUBTITLE_TAMIL`),
                            hindi: extractValue(block, `IMAGE_${i}_SUBTITLE_HINDI`),
                            english: extractValue(block, `IMAGE_${i}_SUBTITLE_ENGLISH`)
                        },
                        description: {
                            tamil: extractValue(block, `IMAGE_${i}_DESC_TAMIL`),
                            hindi: extractValue(block, `IMAGE_${i}_DESC_HINDI`),
                            english: extractValue(block, `IMAGE_${i}_DESC_ENGLISH`)
                        },
                        src: generatePlaceholder(i),
                        transform: { scale: 1, rotation: 0, position: { x: 0, y: 0 }, flip: false }
                    });
                }
                cards.push(card);
            });

            currentCards = cards;
            currentCards.unshift({ id: 'welcome', type: 'welcome', title: { tamil: 'வணக்கம்', hindi: 'नमस्ते', english: 'Welcome' } });
            currentCards.push({ id: 'thankyou', type: 'thankyou', title: { tamil: 'நன்றி', hindi: 'धन्यवाद', english: 'Thank You' } });

            renderCards();
            switchTab('editor');
        }

        function renderCards() {
            const container = document.getElementById('cardsEditor');
            container.innerHTML = '';
            currentCards.forEach((card, index) => {
                container.appendChild(createCardElement(card, index));
            });
        }

        function createCardElement(card, index) {
            // Container for Mobile scaling
            const wrapper = document.createElement('div');
            wrapper.className = 'animate-fade-in';

            // 1. Mobile Controls Header
            const header = `
                <div class="flex justify-between items-center mb-2 px-1">
                    <span class="text-xs font-bold text-slate-400 uppercase">Card ${index + 1}</span>
                    <div class="flex gap-2">
                        ${card.type === 'content' ? `<button onclick="document.getElementById('file-input-${index}').click()" class="text-blue-600 text-xs font-bold bg-blue-50 px-2 py-1 rounded">Upload Img</button>` : ''}
                        <input type="file" id="file-input-${index}" class="hidden" accept="image/*" onchange="handleImageUpload(event, ${index}, 0)">
                        <button onclick="downloadCardImage(${index})" class="text-slate-500 hover:text-blue-600"><i class="ph-bold ph-download-simple text-lg"></i></button>
                    </div>
                </div>
            `;

            // 2. The Scaled Card Logic
            // We need to render the card at its full native resolution (e.g. 1123px wide)
            // but scale it down to fit the mobile screen (e.g. 350px wide)
            const aspectRatio = config.width / config.height;
            
            // Build Inner Content
            let innerHTML = '';
            const title = card.type === 'content' ? (card.mainTitle[globalLanguage] || card.mainTitle.english) : (card.title[globalLanguage] || card.title.english);
            const langFont = globalLanguage === 'tamil' ? 'font-tamil' : globalLanguage === 'hindi' ? 'font-hindi' : 'font-sans';

            if (card.type !== 'content') {
                innerHTML = `
                    <div class="h-full flex items-center justify-center bg-white p-8">
                         <div class="w-full h-full border-[12px] border-slate-800 flex items-center justify-center rounded-[3rem]">
                            <h2 contenteditable class="text-8xl font-bold text-slate-800 ${langFont} text-center outline-none leading-tight">${title}</h2>
                        </div>
                    </div>`;
            } else {
                // Layout Grids
                const gridClass = card.layout === '1-up' ? 'grid-cols-1' : card.layout === '2-up' ? 'grid-cols-2' : card.layout === '3-up' ? 'grid-cols-3' : 'grid-cols-3 grid-rows-2';
                const count = parseInt(card.layout.split('-')[0]);

                innerHTML = `
                    <div class="flex flex-col h-full w-full relative">
                        <div class="flex-shrink-0 z-10 w-full px-8 py-6 bg-white border-b-2 border-slate-100 relative">
                            <h2 contenteditable class="text-5xl font-extrabold text-slate-900 outline-none text-center ${langFont}">${title}</h2>
                        </div>
                        <div class="flex-1 min-h-0 grid gap-1 bg-slate-100 p-1 ${gridClass}">
                            ${card.images.slice(0, count).map((img, i) => `
                                <div class="relative bg-white overflow-hidden group">
                                    <img src="${img.src}" class="w-full h-full object-cover" style="transform: scale(${img.transform.scale}) rotate(${img.transform.rotation}deg)">
                                    <div class="absolute bottom-0 w-full bg-slate-50/90 backdrop-blur border-t border-slate-100 p-3">
                                        <p contenteditable class="text-2xl font-bold text-slate-800 text-center ${langFont}">${img.subtitle[globalLanguage] || img.subtitle.english}</p>
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            }

            // Wrapper logic to handle the scaling via CSS variables or JS
            // Here we use a JS helper to resize on mount
            const cardId = `card-render-${index}`;
            const scalerHTML = `
                <div class="flashcard-wrapper-mobile" style="aspect-ratio: ${aspectRatio}">
                    <div id="${cardId}" class="flashcard-fixed-layout" style="width: ${config.width}px; height: ${config.height}px; transform-origin: top left;">
                        ${innerHTML}
                    </div>
                </div>
            `;

            // Educator Notes (Mobile Friendly)
            const notes = card.type === 'content' ? `
                <div class="mt-2 bg-white rounded-xl p-3 border border-slate-100 shadow-sm">
                    <div class="flex justify-between items-start mb-1">
                        <span class="text-[10px] font-bold text-orange-500 uppercase">Educator Note</span>
                        <button onclick="speak('${(card.images[0]?.description[globalLanguage] || '').replace(/'/g, "\\'")}')" class="text-blue-500"><i class="ph-fill ph-speaker-high"></i></button>
                    </div>
                    <p class="text-xs text-slate-600 leading-relaxed ${langFont}">${card.images[0]?.description[globalLanguage] || card.images[0]?.description.english || ''}</p>
                </div>
            ` : '';

            wrapper.innerHTML = header + scalerHTML + notes;
            
            // Queue the resize calculation
            setTimeout(() => fitCardToMobile(cardId, config.width), 0);
            
            return wrapper;
        }

        function fitCardToMobile(id, nativeW) {
            const el = document.getElementById(id);
            if(!el) return;
            const parent = el.parentElement;
            const scale = parent.clientWidth / nativeW;
            el.style.transform = `scale(${scale})`;
        }

        // Handle window resize to adjust cards
        window.addEventListener('resize', () => {
            currentCards.forEach((_, i) => fitCardToMobile(`card-render-${i}`, config.width));
        });

        // --- Helpers ---
        function extractValue(text, key) {
            const match = text.match(new RegExp(`${key}:\\s*(.+?)(?=\\s*(?:\\n\\s*[A-Z]|IMAGE_|MAIN_|CARD)|$)`, 'is'));
            return match ? match[1].trim() : '';
        }

        function generatePlaceholder(num) {
            const colors = ['#f0f9ff', '#fdf2f8', '#f0fdf4', '#fefce8'];
            const color = colors[num % colors.length];
            return `data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' width='400' height='300'><rect width='100%' height='100%' fill='${encodeURIComponent(color)}'/><text x='50%' y='50%' dominant-baseline='middle' text-anchor='middle' font-family='sans-serif' font-size='30' fill='%2394a3b8'>IMG ${num}</text></svg>`;
        }

        function copyPrompt(type) {
            const text = type === 'content' ? document.getElementById('contentPrompt').textContent : document.getElementById('imagePrompts').textContent;
            navigator.clipboard.writeText(text).then(() => showNotification('Copied!', 'success'));
        }

        function showNotification(msg, type) {
            const div = document.createElement('div');
            div.className = `fixed top-6 left-1/2 transform -translate-x-1/2 px-4 py-2 rounded-full text-white text-xs font-bold shadow-xl z-[60] animate-fade-in flex items-center gap-2 ${type === 'error' ? 'bg-red-500' : 'bg-slate-900'}`;
            div.innerHTML = msg;
            document.body.appendChild(div);
            setTimeout(() => div.remove(), 2000);
        }

        function handleImageUpload(e, cIdx, iIdx) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (evt) => {
                    currentCards[cIdx].images[iIdx].src = evt.target.result;
                    renderCards();
                };
                reader.readAsDataURL(file);
            }
        }

        // --- Export & History ---
        function downloadCardImage(index) {
            const el = document.getElementById(`card-render-${index}`);
            // Temporarily reset transform to capture full res
            const originalTransform = el.style.transform;
            el.style.transform = 'scale(1)';
            
            html2canvas(el, { scale: 1, useCORS: true }).then(canvas => {
                const link = document.createElement('a');
                link.download = `card-${index+1}.png`;
                link.href = canvas.toDataURL();
                link.click();
                el.style.transform = originalTransform;
                showNotification('Saved to Photos', 'success');
            });
        }

        function exportAllPNG() {
            showNotification('Exporting All...', 'info');
            let i = 0;
            const next = () => {
                if(i >= currentCards.length) return;
                downloadCardImage(i);
                i++;
                setTimeout(next, 800);
            };
            next();
        }

        function exportAllPDF() {
            showNotification('Building PDF...', 'info');
            // Simplified PDF export logic for mobile context
            // In a real app, this would be more robust
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF({ orientation: config.width > config.height ? 'l' : 'p', unit: 'px', format: [config.width, config.height] });
            
            let i = 0;
            const processCard = () => {
                if (i >= currentCards.length) {
                    doc.save('health-deck.pdf');
                    return;
                }
                const el = document.getElementById(`card-render-${i}`);
                const originalTransform = el.style.transform;
                el.style.transform = 'scale(1)';

                html2canvas(el, { scale: 1 }).then(canvas => {
                    if(i > 0) doc.addPage();
                    doc.addImage(canvas.toDataURL('image/jpeg', 0.9), 'JPEG', 0, 0, config.width, config.height);
                    el.style.transform = originalTransform;
                    i++;
                    processCard();
                });
            };
            processCard();
        }

        function saveToHistory() {
            const h = JSON.parse(localStorage.getItem('he_history') || '[]');
            h.unshift({ id: Date.now(), date: new Date().toISOString(), config, cards: currentCards });
            localStorage.setItem('he_history', JSON.stringify(h.slice(0, 10)));
            loadHistory();
            showNotification('Project Saved', 'success');
        }

        function loadHistory() {
            const h = JSON.parse(localStorage.getItem('he_history') || '[]');
            const list = document.getElementById('historyList');
            if(h.length === 0) {
                list.innerHTML = '<div class="text-center py-8 text-slate-400 text-xs">No saved projects.</div>';
                return;
            }
            list.innerHTML = h.map(p => `
                <div class="bg-white p-4 rounded-xl shadow-sm border border-slate-100 flex justify-between items-center" onclick="loadProject(${p.id})">
                    <div>
                        <h4 class="font-bold text-slate-800 text-sm">${p.config.mainTopic || 'Untitled'}</h4>
                        <p class="text-xs text-slate-500">${new Date(p.date).toLocaleDateString()} • ${p.cards.length} Cards</p>
                    </div>
                    <i class="ph-bold ph-caret-right text-slate-300"></i>
                </div>
            `).join('');
        }

        function loadProject(id) {
            const h = JSON.parse(localStorage.getItem('he_history'));
            const p = h.find(x => x.id === id);
            if(p) {
                config = p.config;
                currentCards = p.cards;
                document.getElementById('mainTopic').value = config.mainTopic;
                renderCards();
                switchTab('editor');
            }
        }

        // --- Presentation ---
        function startPresentation() {
            if(currentCards.length === 0) return showNotification('No cards', 'error');
            currentPresentationIndex = 0;
            renderPresentation();
        }

        function renderPresentation() {
            const container = document.getElementById('fullscreenContainer');
            const card = currentCards[currentPresentationIndex];
            const title = card.type === 'content' ? (card.mainTitle[globalLanguage] || card.mainTitle.english) : (card.title[globalLanguage] || card.title.english);
            const langFont = globalLanguage === 'tamil' ? 'font-tamil' : globalLanguage === 'hindi' ? 'font-hindi' : 'font-sans';
            const img = card.images?.[0];

            container.innerHTML = `
                <div class="fixed inset-0 z-[100] bg-slate-900 flex flex-col">
                    <div class="flex-1 flex flex-col items-center justify-center p-6 text-center space-y-8">
                        <h1 class="text-4xl font-bold text-white ${langFont}">${title}</h1>
                        ${img ? `<img src="${img.src}" class="max-h-[40vh] rounded-xl shadow-2xl">` : ''}
                        ${img ? `<p class="text-xl text-slate-300 ${langFont}">${img.subtitle[globalLanguage] || img.subtitle.english}</p>` : ''}
                    </div>
                    <div class="h-24 pb-safe bg-slate-800 flex items-center justify-between px-6">
                        <button onclick="document.getElementById('fullscreenContainer').innerHTML = ''" class="text-white font-bold text-xs uppercase bg-red-500/20 px-4 py-2 rounded-full">Exit</button>
                        <div class="flex gap-6 items-center">
                            <button onclick="prevSlide()" class="text-white text-3xl"><i class="ph-bold ph-caret-left"></i></button>
                            <span class="text-slate-400 font-mono text-sm">${currentPresentationIndex + 1}/${currentCards.length}</span>
                            <button onclick="nextSlide()" class="text-white text-3xl"><i class="ph-bold ph-caret-right"></i></button>
                        </div>
                        <button onclick="speak('${img?.description[globalLanguage]?.replace(/'/g, "\\'") || ''}')" class="text-blue-400 text-2xl"><i class="ph-fill ph-speaker-high"></i></button>
                    </div>
                </div>
            `;
        }

        function prevSlide() { if(currentPresentationIndex > 0) { currentPresentationIndex--; renderPresentation(); } }
        function nextSlide() { if(currentPresentationIndex < currentCards.length - 1) { currentPresentationIndex++; renderPresentation(); } }
        function speak(text) {
            window.speechSynthesis.cancel();
            const u = new SpeechSynthesisUtterance(text);
            u.lang = globalLanguage === 'tamil' ? 'ta-IN' : globalLanguage === 'hindi' ? 'hi-IN' : 'en-US';
            window.speechSynthesis.speak(u);
        }

    </script>
</body>
</html>
