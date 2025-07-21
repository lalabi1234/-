<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>化學詳解查詢系統</title>
    <script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
    <script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
    <style>
        :root {
            --primary-color: #4a6fa5;
            --secondary-color: #6b8cae;
            --text-color: #333;
            --light-bg: #f8f9fa;
            --border-radius: 8px;
        }
        
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            line-height: 1.6;
            color: var(--text-color);
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: var(--light-bg);
        }
        
        h1 {
            color: var(--primary-color);
            text-align: center;
            margin-bottom: 30px;
            font-weight: 600;
        }
        
        .search-container {
            background: white;
            padding: 25px;
            border-radius: var(--border-radius);
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            margin-bottom: 30px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .search-box {
            display: flex;
            width: 100%;
            max-width: 500px;
        }
        
        input[type="text"] {
            flex: 1;
            padding: 12px 15px;
            border: 1px solid #ddd;
            border-radius: var(--border-radius) 0 0 var(--border-radius);
            font-size: 16px;
            outline: none;
            transition: border 0.3s;
        }
        
        input[type="text"]:focus {
            border-color: var(--primary-color);
        }
        
        button {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 0 20px;
            border-radius: 0 var(--border-radius) var(--border-radius) 0;
            cursor: pointer;
            font-size: 16px;
            transition: background-color 0.3s;
        }
        
        button:hover {
            background-color: var(--secondary-color);
        }
        
        .result-container {
            background: white;
            padding: 25px;
            border-radius: var(--border-radius);
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            min-height: 300px;
        }
        
        .page-header {
            color: var(--primary-color);
            font-size: 1.4em;
            margin-bottom: 20px;
            padding-bottom: 10px;
            border-bottom: 1px solid #eee;
            font-weight: 600;
        }
        
        .answer-item {
            margin-bottom: 25px;
            padding-bottom: 15px;
            border-bottom: 1px dashed #eee;
        }
        
        .answer-item:last-child {
            border-bottom: none;
        }
        
        .answer-title {
            font-weight: 600;
            color: var(--secondary-color);
            margin-bottom: 8px;
        }
        
        .answer-content {
            font-size: 15px;
            line-height: 1.7;
        }
        
        .no-result {
            text-align: center;
            color: #888;
            padding: 50px 0;
        }
        
        .search-tips {
            margin-top: 15px;
            font-size: 14px;
            color: #666;
            text-align: center;
        }
        
        @media (max-width: 600px) {
            .search-container, .result-container {
                padding: 15px;
            }
            
            h1 {
                font-size: 1.8em;
            }
        }
    </style>
</head>
<body>
    <h1>化學詳解查詢系統</h1>
    
    <div class="search-container">
        <div class="search-box">
            <input type="text" id="searchInput" placeholder="輸入頁碼 (例如: P.3 或 3)" autocomplete="off">
            <button id="searchButton">查詢</button>
        </div>
        <p class="search-tips">請輸入正確的頁碼格式 (如: P.3 或 3) 查詢對應頁面的化學詳解</p>
    </div>
    
    <div class="result-container" id="resultContainer">
        <div class="no-result" id="noResult">
            查詢結果將顯示在這裡
        </div>
        <div id="searchResults"></div>
    </div>

    <script>
        // 示例數據庫 - 您需要替換為實際的化學詳解內容
        const answerDatabase = {
            "P.3": [
                {
                    id: "1",
                    answer: "Ans: \\(E = h\\nu\\)，其中 \\(h\\) 是普朗克常數，\\(\\nu\\) 是頻率。"
                },
                {
                    id: "2",
                    answer: "Ans: 波長 \\(\\lambda\\) 與頻率的關係為 \\(\\lambda = \\frac{c}{\\nu}\\)，其中 \\(c\\) 是光速。"
                },
                {
                    id: "3",
                    answer: "Ans: 氫原子能級 \\(E_n = -\\frac{13.6}{n^2} \\text{eV}\\)，其中 \\(n\\) 為主量子數。"
                }
            ],
            "P.4": [
                {
                    id: "1", 
                    answer: "Ans: 理想氣體方程式為 \\(PV = nRT\\)，\\(R\\) 是通用氣體常數。"
                },
                {
                    id: "2",
                    answer: "Ans: 酸鹼中和反應：\\(\\text{H}^+ + \\text{OH}^- \\rightarrow \\text{H}_2\\text{O}\\)。"
                }
            ],
            "P.10": [
                {
                    id: "1",
                    answer: "Ans: 反應速率與濃度關係 \\(\\text{rate} = k[\\text{A}]^m[\\text{B}]^n\\)。"
                },
                {
                    id: "2",
                    answer: "Ans: 勒沙特列原理：平衡受到擾動時，系統會產生變化抵消此擾動。"
                }
            ]
        };

        // DOM元素
        const searchInput = document.getElementById('searchInput');
        const searchButton = document.getElementById('searchButton');
        const resultContainer = document.getElementById('resultContainer');
        const noResult = document.getElementById('noResult');
        const searchResults = document.getElementById('searchResults');
        
        // 執行查詢
        function searchAnswers() {
            let query = searchInput.value.trim();
            
            // 處理不同輸入格式
            if (query.startsWith('P.')) {
                // 已經是P.格式
            } else if (/^\d+$/.test(query)) {
                query = `P.${query}`;
            } else {
                noResult.style.display = 'block';
                searchResults.innerHTML = '';
                noResult.textContent = '請輸入有效的頁碼格式 (如: P.3 或 3)';
                return;
            }
            
            const results = answerDatabase[query];
            
            if (results && results.length > 0) {
                noResult.style.display = 'none';
                
                let html = `<div class="page-header">頁碼：${query}</div>`;
                
                results.forEach(item => {
                    html += `
                        <div class="answer-item">
                            <div class="answer-title">題目 ${item.id}</div>
                            <div class="answer-content">${item.answer}</div>
                        </div>
                    `;
                });
                
                searchResults.innerHTML = html;
                
                // 觸發MathJax重新渲染
                if (typeof MathJax !== 'undefined') {
                    MathJax.typeset();
                }
            } else {
                noResult.style.display = 'block';
                searchResults.innerHTML = '';
                noResult.textContent = `找不到頁碼 ${query} 的相關解答`;
            }
        }
        
        // 事件監聽
        searchButton.addEventListener('click', searchAnswers);
        
        searchInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                searchAnswers();
            }
        });
        
        // 初始加載時渲染一些內容
        window.addEventListener('load', () => {
            if (typeof MathJax !== 'undefined') {
                MathJax.typeset();
            }
        });
    </script>
</body>
</html>
