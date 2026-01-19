# influencer-agreement
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KOL Contract Generator - Xorigin AI</title>
    <!-- 引入 Vue.js -->
    <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>
    <!-- 引入 html2pdf -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <!-- 引入 Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f3f4f6; }
        
        /* 模拟 A4 纸张样式 */
        .a4-page {
            width: 210mm;
            min-height: 297mm;
            padding: 20mm;
            margin: 0 auto;
            background: white;
            box-shadow: 0 0 15px rgba(0,0,0,0.1);
            font-family: "Times New Roman", Times, serif;
            font-size: 11pt; /* 稍微调小字体以容纳更多内容，避免过多分页 */
            line-height: 1.4;
            color: #000;
        }

        .contract-title { text-align: center; font-weight: bold; font-size: 16pt; margin-bottom: 20px; text-decoration: underline; }
        .section-title { font-weight: bold; margin-top: 15px; margin-bottom: 5px; text-transform: uppercase; font-size: 11pt; }
        
        /* 输入框样式 */
        .input-group { margin-bottom: 12px; }
        .input-label { display: block; font-size: 0.85rem; font-weight: 600; color: #374151; margin-bottom: 4px; }
        .input-field { width: 100%; padding: 8px; border: 1px solid #d1d5db; border-radius: 4px; font-size: 0.9rem; transition: border 0.2s; }
        .input-field:focus { outline: none; border-color: #2563eb; ring: 2px; }
        
        /* 预设标签样式 */
        .preset-tag {
            display: inline-block;
            background-color: #e5e7eb;
            color: #4b5563;
            font-size: 0.75rem;
            padding: 2px 8px;
            border-radius: 12px;
            margin-right: 5px;
            margin-top: 4px;
            cursor: pointer;
            transition: all 0.2s;
            border: 1px solid transparent;
        }
        .preset-tag:hover { background-color: #d1d5db; color: #000; border-color: #9ca3af; }

        /* 合同中的变量文字样式 */
        .variable-text { color: #000; font-weight: bold; text-decoration: underline; }

        /* 签字区域特殊样式 */
        .signature-box { margin-top: 40px; display: flex; justify-content: space-between; page-break-inside: avoid; }
        .signature-line { border-bottom: 1px solid #000; margin-top: 50px; margin-bottom: 10px; } /* 增加 margin-top 拉大签字空间 */
    </style>
</head>
<body>

<div id="app" class="flex h-screen overflow-hidden">
    
    <!-- 左侧：输入控制面板 -->
    <div class="w-1/3 h-full bg-white border-r border-gray-200 overflow-y-auto p-6 shadow-xl z-10 custom-scrollbar">
        <h2 class="text-xl font-bold text-gray-800 mb-6 flex items-center border-b pb-4">
            📝 合同信息录入
        </h2>

        <form @submit.prevent="downloadPDF">
            <!-- 1. 基础信息 -->
            <div class="mb-6">
                <h3 class="text-xs uppercase tracking-wider text-blue-600 font-bold mb-3">Basic Info (基础信息)</h3>
                
                <div class="input-group">
                    <label class="input-label">签约日期 (Date)</label>
                    <input type="text" v-model="form.signDate" class="input-field">
                    <div class="text-xs text-gray-400 mt-1">* 自动生成今日日期，可修改</div>
                </div>

                <div class="input-group">
                    <label class="input-label">达人姓名 (Influencer Name)</label>
                    <input type="text" v-model="form.influencerName" class="input-field" placeholder="Full Legal Name">
                </div>

                <div class="input-group">
                    <label class="input-label">社交账号 (Account Link/Handle)</label>
                    <input type="text" v-model="form.account" class="input-field" placeholder="@username">
                </div>
            </div>

            <!-- 2. 交付详情 -->
            <div class="mb-6">
                <h3 class="text-xs uppercase tracking-wider text-blue-600 font-bold mb-3">Deliverables (交付内容)</h3>

                <div class="input-group">
                    <label class="input-label">发布平台 (Platforms)</label>
                    <input type="text" v-model="form.platforms" class="input-field" placeholder="e.g. TikTok">
                    <!-- 快捷标签 -->
                    <div>
                        <span v-for="tag in presets.platforms" @click="form.platforms = tag" class="preset-tag">{{ tag }}</span>
                    </div>
                </div>

                <div class="input-group">
                    <label class="input-label">内容形式 (Content Format)</label>
                    <input type="text" v-model="form.contentFormat" class="input-field" placeholder="e.g. 1 Video">
                    <!-- 快捷标签 -->
                    <div>
                        <span v-for="tag in presets.contentFormat" @click="form.contentFormat = tag" class="preset-tag">{{ tag }}</span>
                    </div>
                </div>

                <div class="input-group">
                    <label class="input-label">交付数量 (Quantity)</label>
                    <input type="text" v-model="form.quantity" class="input-field" placeholder="e.g. 1 Video">
                    <!-- 快捷标签 -->
                    <div>
                        <span v-for="tag in presets.quantity" @click="form.quantity = tag" class="preset-tag">{{ tag }}</span>
                    </div>
                </div>

                <div class="input-group">
                    <label class="input-label">发布时间 (Posting Timeline)</label>
                    <input type="text" v-model="form.postingDate" class="input-field">
                    <!-- 快捷标签 -->
                    <div>
                        <span v-for="tag in presets.postingDate" @click="form.postingDate = tag" class="preset-tag">{{ tag }}</span>
                    </div>
                </div>
            </div>

            <!-- 3. 付款信息 -->
            <div class="mb-6">
                <h3 class="text-xs uppercase tracking-wider text-blue-600 font-bold mb-3">Payment (付款信息)</h3>

                <div class="input-group">
                    <label class="input-label">基础费用 (Base Fee)</label>
                    <input type="text" v-model="form.baseFee" class="input-field" placeholder="e.g. USD $500.00">
                </div>

                <div class="input-group">
                    <label class="input-label">支付方式 (Payment Method)</label>
                    <input type="text" v-model="form.paymentMethod" class="input-field" placeholder="e.g. PayPal">
                    <!-- 快捷标签 -->
                    <div>
                        <span v-for="tag in presets.paymentMethod" @click="form.paymentMethod = tag" class="preset-tag">{{ tag }}</span>
                    </div>
                </div>

                <div class="input-group">
                    <label class="input-label">支付时间 (Payment Timeline)</label>
                    <input type="text" v-model="form.paymentTimeline" class="input-field" placeholder="When to pay">
                    <!-- 快捷标签 -->
                    <div>
                        <span v-for="tag in presets.paymentTimeline" @click="form.paymentTimeline = tag" class="preset-tag">{{ tag }}</span>
                    </div>
                </div>
            </div>

            <div class="mt-8 pb-10">
                <button type="button" @click="downloadPDF" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded shadow transition duration-200 flex justify-center items-center">
                    <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                    生成并下载 PDF
                </button>
            </div>
        </form>
    </div>

    <!-- 右侧：PDF 预览区域 -->
    <div class="w-2/3 h-full bg-gray-500 overflow-y-auto p-8 flex justify-center">
        <!-- 合同内容正文 -->
        <div id="contract-content" class="a4-page">
            <div class="contract-title">Collaboration Agreement</div>
            
            <p>This Agreement (the "Agreement") is entered into on <span class="variable-text">{{ form.signDate }}</span> by and between:</p>
            
            <p><strong>Party A (Brand):</strong><br>
             Company Name: Xorigin AI<br>
             Brand Contact: Ashley Yang<br>
             Contact Phone: +86 18389799201</p>
            
            <p><strong>Party B (Influencer):</strong><br>
             Influencer Name: <span class="variable-text">{{ form.influencerName || '________________________' }}</span><br>
             Account: <span class="variable-text">{{ form.account || '________________________' }}</span></p>
            
            <div class="section-title">Recitals</div>
            <p>Party A wishes to engage Party B to leverage their social media influence to promote Party A's designated brand/product/service; and Party B agrees to provide such promotional services in accordance with the terms of this Agreement.</p>

            <div class="section-title">1. Scope of Services & Deliverables</div>
            <p>
                1.1 Campaign/Project: Yonbo Campaign<br>
                1.2 Platforms: <span class="variable-text">{{ form.platforms || '________________________' }}</span><br>
                1.3 Content Format: <span class="variable-text">{{ form.contentFormat || '________________________' }}</span><br>
                1.4 Quantity of Deliverables: <span class="variable-text">{{ form.quantity || '________________________' }}</span><br>
                1.5 Content Requirements:<br>
                Posting Date/Timeline: <span class="variable-text">{{ form.postingDate || '________________________' }}</span><br>
                Brand Mentions/Tags: @yonbo @xorgin #yonbo<br>
                Other Requirements: Content to be submitted for Party A’s approval prior to posting.<br>
                1.6 Content Approval: Party B shall submit all promotional content to Party A for review and approval prior to posting. Party A may request revisions, which Party B shall reasonably accommodate.
            </p>

            <div class="section-title">2. Compensation & Payment</div>
            <p>
                2.1 Base Fee<br>
                Party A agrees to pay Party B a total base fee of: <span class="variable-text">{{ form.baseFee || '________________________' }}</span>.<br>
                2.2 Product Compensation<br>
                Party A shall provide Party B with one unit of Yonbo Children’s AI Robot as part of the collaboration compensation.<br>
                2.3 Payment Method: <span class="variable-text">{{ form.paymentMethod || '________________________' }}</span><br>
                2.4 Payment Timeline: <span class="variable-text">{{ form.paymentTimeline || '________________________' }}</span>
            </p>

            <div class="section-title">3. Intellectual Property & Usage Rights</div>
            <p>
                3.1 Ownership<br>
                All rights, title, and interest in and to the content created by Party B (“Content”) shall remain the sole and exclusive property of Party B.<br>
                3.2 License Grant<br>
                Party B hereby grants Party A a limited, non-exclusive, non-transferable, non-sublicensable, non-assignable, revocable license to use the Content solely on Party A’s official brand-owned social media channels (including Party A’s verified Instagram, TikTok, Facebook, and YouTube accounts) for a period of twelve (12) months from the date of first posting (“License Term”).<br>
                3.3 Restrictions on Use<br>
                Unless expressly agreed to in a separate written agreement with additional compensation, the following are strictly prohibited:<br>
                (a) Use by third parties, affiliates, resellers, distributors, partners, or any entity other than Party A;<br>
                (b) Use of the Content in television, print, out-of-home, retail placements, e-commerce listings, or any commercial medium beyond Party A’s own organic social media channels.
            </p>

            <div class="section-title">4. Confidentiality</div>
            <p>Both parties agree to keep the terms of this Agreement and any confidential information obtained during the collaboration confidential, unless required by law to disclose.</p>

            <div class="section-title">5. Breach of Contract</div>
            <p>
                5.1 If Party B fails to submit the first draft of the video within 14 days of receiving the product, and still fails to deliver the draft after an additional 3 days of delay, while also exhibiting an uncooperative and hostile attitude (including but not limited to refusing communication without justifiable reasons, maliciously delaying the process, using offensive language, etc.), such conduct shall be deemed a material breach of this Agreement by Party B.<br>
                5.2 In the event of the aforementioned material breach, Party A shall be entitled to take the following measures:<br>
                Unilaterally require Party B to return the Yonbo Robot provided by Party A within 3 days, with all related return shipping costs to be borne by Party B. If Party B fails to return the product within the time limit or intentionally damages the product, Party A shall have the right to claim property compensation from Party B.<br>
                Refuse to pay Party B the entire base fee as stipulated in this Agreement, and shall not be obligated to make any additional compensation to Party B whatsoever.<br>
                5.3 Party B shall actively cooperate with Party A’s request for product return. If the product cannot be recovered or suffers any devaluation due to Party B’s fault, Party A shall be entitled to safeguard its legitimate rights and interests through legal channels.<br>
                Any party that breaches this Agreement shall be liable for any losses incurred by the other party as a result of such breach.
            </p>

            <div class="section-title">6. Miscellaneous</div>
            <p>
                6.1 This Agreement constitutes the entire agreement between the parties concerning the subject matter herein and supersedes all prior and contemporaneous agreements, understandings, negotiations, and discussions, whether oral or written.<br>
                6.2 Any amendments to this Agreement must be in writing and signed by both parties.<br>
                <!-- 新增条款 6.3 -->
                <strong>6.3 If the performance of the cooperation video is significantly lower than the average level of the same batch of cooperation videos, and the actual view count is less than one-fifth of the average view count, Party A shall have the right to request the Influencer to take remedial promotion measures. Remedial methods include but are not limited to publishing an additional cooperation video with the same theme or pinning the original cooperation video as the featured content of the account. The specific implementation plan shall be separately negotiated by both parties and confirmed in writing.</strong>
            </p>

            <!-- 签字区域 -->
            <div class="signature-box">
                <div style="width: 45%;">
                    <p><strong>Party A (Brand) Signature:</strong></p>
                    <p style="font-family: 'Brush Script MT', cursive; font-size: 24pt; margin: 20px 0;">Ashley</p>
                    <p>Date: <span class="variable-text">{{ form.signDate }}</span></p>
                </div>
                <div style="width: 45%;">
                    <p><strong>Party B (Influencer) Signature:</strong></p>
                    <!-- 空大一点，方便签字 -->
                    <div class="signature-line" style="height: 40px;"></div> 
                    <!-- 日期直接使用签约日期 -->
                    <p>Date: <span class="variable-text">{{ form.signDate }}</span></p>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
    new Vue({
        el: '#app',
        data: {
            form: {
                signDate: '',
                influencerName: '',
                account: '',
                platforms: '',
                contentFormat: '',
                quantity: '',
                postingDate: '',
                baseFee: '',
                paymentMethod: '',
                paymentTimeline: ''
            },
            // 常用选项配置 (Presets)
            presets: {
                platforms: ['TikTok', 'Instagram', 'YouTube', 'TikTok & IG'],
                contentFormat: ['1 Dedicated Video', '1 Integrated Video', '1 Story', 'Photo Post'],
                quantity: ['1 Video', '1 Video + 1 Story', '2 Videos'],
                postingDate: ['Within 7 days after receiving product', 'Within 14 days after receiving product'],
                paymentMethod: ['PayPal', 'Bank Transfer', 'Wise'],
                paymentTimeline: ['Net 30', 'Net 15', '50% Upfront, 50% Post']
            }
        },
        mounted() {
            // 自动生成今天的英文日期
            const date = new Date();
            const month = date.toLocaleString('en-US', { month: 'long' });
            const day = date.getDate();
            const year = date.getFullYear();
            
            const suffix = (d) => {
                if (d > 3 && d < 21) return 'th';
                switch (d % 10) {
                    case 1:  return "st";
                    case 2:  return "nd";
                    case 3:  return "rd";
                    default: return "th";
                }
            };
            this.form.signDate = `${month} ${day}${suffix(day)}, ${year}`;
        },
        methods: {
            downloadPDF() {
                const element = document.getElementById('contract-content');
                
                // 优化 PDF 导出配置
                const opt = {
                    margin:       [10, 15, 10, 15], // 上 右 下 左
                    filename:     `Contract_Xorigin_${this.form.influencerName.replace(/\s+/g, '_') || 'Influencer'}.pdf`,
                    image:        { type: 'jpeg', quality: 0.98 },
                    html2canvas:  { scale: 2, useCORS: true }, 
                    jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' },
                    pagebreak:    { mode: ['avoid-all', 'css', 'legacy'] } // 避免切断文字
                };

                html2pdf().set(opt).from(element).save();
            }
        }
    });
</script>

</body>
</html>
