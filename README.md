
[新文字文件index.html](https://github.com/user-attachments/files/26184610/index.html)
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>橫向多元新聞瀏覽器 | Diverse Horizontal News</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;700;900&display=swap');
        
        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background-color: #020617; 
            color: #f8fafc;
            overflow: hidden;
        }

        /* 隱藏原生捲軸 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

        .horizontal-scroller {
            display: flex;
            overflow-x: auto;
            scroll-snap-type: x mandatory;
            height: calc(100vh - 120px);
            align-items: center;
            padding: 0 5vw;
        }

        .news-card {
            flex: 0 0 320px;
            height: 520px;
            margin: 0 15px;
            scroll-snap-align: center;
            border-radius: 2.5rem;
            overflow: hidden;
            position: relative;
            transition: all 0.5s cubic-bezier(0.23, 1, 0.32, 1);
            cursor: pointer;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
        }

        @media (max-width: 640px) {
            .news-card { flex: 0 0 85vw; height: 65vh; margin: 0 10px; }
        }

        .news-card:hover {
            transform: scale(1.05);
            box-shadow: 0 0 30px rgba(59, 130, 246, 0.3);
        }

        .card-gradient {
            background: linear-gradient(to top, rgba(2, 6, 23, 1) 0%, rgba(2, 6, 23, 0.6) 30%, transparent 100%);
        }

        .category-tag {
            padding: 4px 12px;
            border-radius: 999px;
            font-size: 10px;
            font-weight: 900;
            text-transform: uppercase;
            letter-spacing: 0.1em;
        }

        /* 主題顏色定義 */
        .tag-生活 { background: #3b82f6; color: white; }
        .tag-娛樂 { background: #ec4899; color: white; }
        .tag-科技 { background: #8b5cf6; color: white; }
        .tag-寵物 { background: #f59e0b; color: white; }
        .tag-時尚 { background: #10b981; color: white; }
    </style>
</head>
<body class="flex flex-col h-screen">

    <!-- 頂部導航 -->
    <header class="p-6 flex justify-between items-center z-50">
        <div class="flex items-center gap-3">
            <div class="bg-blue-600 w-8 h-8 rounded-lg flex items-center justify-center">
                <i class="fas fa-layer-group text-white text-sm"></i>
            </div>
            <h1 class="text-lg font-black tracking-tighter">DIVERSE <span class="text-blue-500">FEED</span></h1>
        </div>
        <div class="text-[10px] font-bold text-slate-500 tracking-widest uppercase">
            Inconsistent Mode • 5 Topics
        </div>
    </header>

    <!-- 橫向滑動區 -->
    <main class="flex-1 overflow-hidden">
        <div id="scroller" class="horizontal-scroller no-scrollbar">
            <!-- 新聞內容由 JS 產生 -->
        </div>
    </main>

    <!-- 底部資訊 -->
    <footer class="p-6 text-center text-slate-600 text-[10px] font-medium tracking-widest uppercase">
        Swipe left or right to explore
    </footer>

    <!-- 分享 QR Code 按鈕 -->
    <button onclick="toggleQRModal()" class="fixed bottom-8 right-8 w-14 h-14 bg-white text-slate-900 rounded-full flex items-center justify-center shadow-2xl hover:bg-blue-500 hover:text-white transition-all active:scale-90 z-[60]">
        <i class="fas fa-qrcode text-xl"></i>
    </button>

    <!-- QR Code 彈窗 -->
    <div id="qrModal" class="fixed inset-0 bg-slate-950/90 backdrop-blur-xl z-[100] hidden items-center justify-center p-4">
        <div class="bg-white p-8 rounded-[3rem] shadow-2xl flex flex-col items-center max-w-sm w-full text-slate-900 animate-in zoom-in duration-300">
            <h3 class="text-xl font-bold mb-2">同步手機預覽</h3>
            <p class="text-slate-400 text-[10px] mb-6 text-center uppercase tracking-widest">Scan to sync experience</p>
            
            <div id="qrcode" class="bg-white p-2 border-4 border-slate-50 rounded-2xl mb-6"></div>
            
            <input type="text" id="manualUrl" oninput="generateQR(this.value)" placeholder="輸入您部署後的網址" 
                   class="w-full bg-slate-100 border-none rounded-2xl px-5 py-3 text-sm mb-4 outline-none focus:ring-2 focus:ring-blue-500">
            
            <button onclick="toggleQRModal()" class="w-full bg-slate-900 text-white py-4 rounded-2xl font-bold text-sm tracking-widest uppercase">Close</button>
        </div>
    </div>

    <!-- 閱讀詳情彈窗 -->
    <div id="articleModal" class="fixed inset-0 bg-slate-950/95 z-[110] hidden items-end sm:items-center justify-center p-0 sm:p-4" onclick="handleBackdropClick(event)">
        <div class="bg-slate-900 w-full max-w-2xl sm:rounded-[3rem] overflow-hidden shadow-2xl animate-in slide-in-from-bottom-full duration-500">
            <div class="relative h-72">
                <img id="modalImg" src="" class="w-full h-full object-cover">
                <div class="absolute inset-0 bg-gradient-to-t from-slate-900 to-transparent"></div>
                <button onclick="closeModal()" class="absolute top-6 right-6 bg-white/10 hover:bg-white/20 text-white w-10 h-10 rounded-full flex items-center justify-center backdrop-blur-md transition">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <div class="p-10">
                <span id="modalCat" class="category-tag mb-4 inline-block">生活</span>
                <h3 id="modalTitle" class="text-3xl font-bold mb-6 text-white leading-tight">標題</h3>
                <div class="space-y-4 text-slate-400 leading-relaxed text-lg">
                    <p>這是一則專門為本實驗設計的新聞內容。在「橫向滾動」且「主題多樣」的配置下，使用者通常會感受到較強的新鮮感與探索慾望，因為下一張卡片總是代表著截然不同的領域。</p>
                    <p>您可以將此處替換為任何真實的報導內容，以測試使用者在不同主題間跳躍時的反應。</p>
                </div>
                <button onclick="closeModal()" class="mt-10 w-full bg-white text-slate-900 py-5 rounded-2xl font-black uppercase tracking-widest text-sm hover:bg-blue-500 hover:text-white transition-colors">Confirm & Back</button>
            </div>
        </div>
    </div>

    <script>
        // 資訊不一致數據集
        const diverseNews = [
            { cat: '生活', title: '週末野餐好去處：北部 5 大隱藏版草地推薦', img: 'https://images.unsplash.com/photo-1533221300444-245f73976378?w=800' },
            { cat: '娛樂', title: '年度音樂盛典揭曉：昨晚頒獎典禮的精彩亮點', img: 'https://images.unsplash.com/photo-1514525253361-bee8a187499b?w=800' },
            { cat: '科技', title: '全新 AI 發展：科技巨頭將徹底改變個人電腦體驗', img: 'https://images.unsplash.com/photo-1485827404703-89b55fcc595e?w=800' },
            { cat: '寵物', title: '高齡犬照護指南：老毛孩需要的核心營養建議', img: 'https://images.unsplash.com/photo-1516734212186-a967f81ad0d7?w=800' },
            { cat: '時尚', title: '巴黎時裝週觀察：定義本季風格的膽大色彩設計', img: 'https://images.unsplash.com/photo-1490481651871-ab68de25d43d?w=800' }
        ];

        function renderContent() {
            const scroller = document.getElementById('scroller');
            scroller.innerHTML = '';

            diverseNews.forEach((news, idx) => {
                scroller.innerHTML += `
                    <div class="news-card border border-white/5" onclick="openModal('${news.title}', '${news.cat}', '${news.img}')">
                        <img src="${news.img}" class="w-full h-full object-cover">
                        <div class="absolute inset-0 card-gradient"></div>
                        <div class="absolute bottom-10 left-8 right-8">
                            <span class="category-tag tag-${news.cat}">${news.cat}</span>
                            <h3 class="text-2xl font-bold mt-4 leading-tight">${news.title}</h3>
                            <div class="mt-6 flex items-center justify-between">
                                <span class="text-[10px] text-slate-500 font-bold uppercase tracking-widest">Story 0${idx+1}</span>
                                <i class="fas fa-arrow-right text-blue-500"></i>
                            </div>
                        </div>
                    </div>
                `;
            });
        }

        // 彈窗控制
        function openModal(t, c, i) {
            document.getElementById('modalTitle').innerText = t;
            const catTag = document.getElementById('modalCat');
            catTag.innerText = c;
            catTag.className = `category-tag tag-${c} mb-4 inline-block`;
            document.getElementById('modalImg').src = i;
            document.getElementById('articleModal').classList.remove('hidden');
            document.getElementById('articleModal').classList.add('flex');
            document.body.style.overflow = 'hidden';
        }

        function closeModal() {
            document.getElementById('articleModal').classList.add('hidden');
            document.body.style.overflow = 'auto';
        }

        function handleBackdropClick(e) {
            if (e.target.id === 'articleModal') closeModal();
        }

        // QR Code 生成
        let qrcode;
        function generateQR(val) {
            const container = document.getElementById('qrcode');
            container.innerHTML = '';
            qrcode = new QRCode(container, {
                text: val || window.location.href,
                width: 180,
                height: 180,
                colorDark : "#020617",
                colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });
        }

        function toggleQRModal() {
            const modal = document.getElementById('qrModal');
            if (modal.classList.contains('hidden')) {
                modal.classList.remove('hidden');
                modal.classList.add('flex');
                generateQR(document.getElementById('manualUrl').value);
            } else {
                modal.classList.add('hidden');
            }
        }

        window.onload = renderContent;
    </script>
</body>
</html>
