# 🤖 Agent Patterns với LangGraph

Dự án này triển khai ba mô hình agent design patterns phổ biến và mạnh mẽ trong AI: **ReAct**, **Reflection**, và **Plan-and-Execute**. Mỗi pattern giải quyết các loại vấn đề khác nhau và phù hợp với các use case cụ thể.

## 📋 Mục lục

- [Cài đặt](#-cài-đặt)
- [1. ReAct Agent](#1️⃣-react-agent)
- [2. Reflection Agent](#2️⃣-reflection-agent)
- [3. Plan-and-Execute Agent](#3️⃣-plan-and-execute-agent)
- [So sánh các Patterns](#-so-sánh-các-patterns)
- [Dependencies](#-dependencies)

---

## 🚀 Cài đặt

### Yêu cầu hệ thống
- Python >= 3.12
- UV package manager

### Các bước cài đặt

**Bước 1:** Clone repository
```bash
git clone <repository-url>
cd Agents
```

**Bước 2:** Cài đặt dependencies
```bash
uv sync
```

**Bước 3:** Cấu hình biến môi trường

Tạo file `.env` trong thư mục gốc:
```bash
OPENAI_API_KEY=your_openai_api_key
TAVILY_API_KEY=your_tavily_api_key
```

---

## 1️⃣ ReAct Agent

> **File:** [ReAct_Agent.ipynb](ReAct_Agent.ipynb)  
> **Pattern:** Reasoning + Acting (Suy luận + Hành động)

### 📖 Tổng quan
ReAct (Reasoning and Acting) là pattern kết hợp suy luận logic với hành động thực tế. Agent sẽ **suy nghĩ từng bước** (reasoning) trước khi **thực hiện hành động** (acting), tạo thành vòng lặp liên tục cho đến khi có câu trả lời cuối cùng.

### 🧠 Cơ chế hoạt động

```
User Input → Think → Act → Observe → Think → Act → ... → Final Answer
```

**Chu trình ReAct:**
1. **Thought** (Suy nghĩ): Agent phân tích tình huống và quyết định bước tiếp theo
2. **Action** (Hành động): Thực hiện công cụ (tool) cụ thể
3. **Observation** (Quan sát): Nhận kết quả từ tool
4. *(Lặp lại nếu cần)*
5. **Final Answer**: Đưa ra câu trả lời cuối cùng

### 🏗️ Kiến trúc

```
┌─────────┐
│  Agent  │ ←─────────────┐
└────┬────┘                │
     │                     │
     ├── Thought            │
     │                     │
     ▼                     │
┌─────────┐                │
│  Tools  │                │
│ (Search,│                │
│ Triple) │                │
└────┬────┘                │
     │                     │
     └─ Observation ───────┘
```

### 🛠️ Components

**State:**
- `messages`: Lịch sử hội thoại
- `number_of_steps`: Số bước đã thực hiện

**Tools:**
- `TavilySearch`: Tìm kiếm web real-time
- `triple()`: Nhân một số với 3 (demo tool)

**Model:** GPT-4o-mini với tool calling

### 🚀 Cách chạy
1. Mở file [ReAct_Agent.ipynb](ReAct_Agent.ipynb)
2. Chạy lần lượt từng cell bằng cách nhấn nút ▶️ Run

### 💡 Ví dụ sử dụng

**Example 1:** Tìm kiếm đơn giản
```python
"Thời tiết ở Việt Nam như thế nào?"
```
→ Thought: Cần tìm kiếm thông tin thời tiết  
→ Action: TavilySearch("thời tiết Việt Nam")  
→ Observation: Kết quả tìm kiếm  
→ Final Answer: Trả lời người dùng

**Example 2:** Kết hợp nhiều bước
```python
"Thời tiết ở Việt Nam như thế nào? Sau đó nhân ba lên."
```
→ Thought: Tìm kiếm nhiệt độ  
→ Action: TavilySearch  
→ Observation: "Nhiệt độ 30°C"  
→ Thought: Nhân 30 với 3  
→ Action: triple(30)  
→ Observation: 90  
→ Final Answer: "Nhiệt độ hiện tại là 30°C, nhân 3 = 90°C"

### ✅ Ưu điểm
- Minh bạch: Nhìn thấy rõ quá trình suy nghĩ
- Linh hoạt: Tự quyết định khi nào dùng tool
- Hiệu quả: Giải quyết được nhiều loại task phức tạp

### ⚠️ Hạn chế
- Có thể "lạc đường" với task quá phức tạp
- Không có kế hoạch dài hạn
- Khó debug khi có nhiều bước

### 🎯 Khi nào sử dụng
- Task cần kết hợp nhiều tools khác nhau
- Cần giải thích quá trình reasoning
- Task không quá phức tạp (< 5-7 bước)

---

## 2️⃣ Reflection Agent

> **File:** [Reflection_Agent.ipynb](Reflection_Agent.ipynb)  
> **Pattern:** Generate → Reflect → Refine (Tạo → Phản ánh → Cải thiện)

### 📖 Tổng quan
Reflection Agent sử dụng **self-critique** (tự phê bình) để cải thiện chất lượng output. Agent sẽ tạo ra kết quả ban đầu, sau đó tự đánh giá và cải thiện qua nhiều vòng lặp.

### 🧠 Cơ chế hoạt động

```
Input → Generate → Reflect → Generate (v2) → Reflect → ... → Final Output
```

**Chu trình Reflection:**
1. **Generate**: Tạo output ban đầu
2. **Reflect**: Phê bình và đưa ra đề xuất cải thiện
3. **Generate**: Tạo phiên bản cải thiện dựa trên feedback
4. *(Lặp lại cho đến khi đạt tiêu chí)*
5. **Final**: Trả về phiên bản tốt nhất

### 🏗️ Kiến trúc

```
    ┌──────────┐
    │ Generate │
    └────┬─────┘
         │
         ▼
    ┌──────────┐
    │ Reflect  │
    └────┬─────┘
         │
         └─────► (Lặp lại max 6 lần)
                      │
                      ▼
                   END
```

### 🛠️ Components

**State:**
- `messages`: Danh sách các message (request, response, critique)

**Nodes:**
- `Generator`: Tạo nội dung (viết tweet)
- `Reflector`: Phê bình và đề xuất cải thiện

**Models:** 
- Generator: GPT-4o-mini
- Reflector: GPT-4o-mini

**Stopping Condition:** Dừng sau 3 vòng reflection (6 messages)

### 🚀 Cách chạy
1. Mở file [Reflection_Agent.ipynb](Reflection_Agent.ipynb)
2. Chạy lần lượt từng cell bằng cách nhấn nút ▶️ Run

### 💡 Ví dụ sử dụng

**Example:** Viết tweet chất lượng cao
```python
"Hãy viết một bài đăng trên Twitter về lý do tại sao 
Hoàng tử bé vẫn còn phù hợp với tuổi thơ hiện đại."
```

**Vòng 1:**
- Generate: "Hoàng tử bé dạy trẻ em về tình bạn và trách nhiệm..."
- Reflect: "Nội dung tốt nhưng hơi dài. Cần ngắn gọn hơn, thêm emoji, và hook mạnh mẽ hơn."

**Vòng 2:**
- Generate: "🌟 Hoàng tử bé = bài học vượt thời gian! Tình bạn, trách nhiệm, tưởng tượng..."
- Reflect: "Tốt hơn! Nhưng cần thêm call-to-action và hashtag."

**Vòng 3:**
- Generate: "🌟 Vì sao Hoàng tử bé vẫn phù hợp? Tình bạn thật, trách nhiệm, và sức mạnh tưởng tượng - điều trẻ hiện đại CẦN! 💫 #HoangTuBe #PhanTichSach"
- Final Output ✅

### ✅ Ưu điểm
- Chất lượng cao: Output được cải thiện qua nhiều vòng
- Tự động: Không cần human feedback
- Versatile: Áp dụng cho writing, code generation, reasoning

### ⚠️ Hạn chế
- Tốn token: Mỗi vòng reflection tiêu tốn thêm API calls
- Có thể "over-optimize": Cải thiện quá mức làm mất tự nhiên
- Không phù hợp với task cần sự kiện real-time

### 🎯 Khi nào sử dụng
- **Creative writing**: Tweet, blog post, marketing copy
- **Code generation**: Tự review và cải thiện code
- **Essay writing**: Văn phong học thuật, báo cáo
- **Bất kỳ task nào cần chất lượng output cao**

---

## 3️⃣ Plan-and-Execute Agent

> **File:** [Plan-and-Execute_Agent.ipynb](Plan-and-Execute_Agent.ipynb)  
> **Pattern:** Plan → Execute → Replan (Lập kế hoạch → Thực thi → Điều chỉnh)

### 📖 Tổng quan
Plan-and-Execute là pattern **chiến lược** cho các task phức tạp. Agent sẽ **lập kế hoạch toàn diện** trước, sau đó **thực thi từng bước**, và **điều chỉnh kế hoạch** dựa trên kết quả thực tế.

### 🧠 Cơ chế hoạt động

```
Input → Plan → Execute Step 1 → Replan → Execute Step 2 → ... → Final Answer
```

**Chu trình Plan-Execute:**
1. **Plan**: Tạo kế hoạch chi tiết với các bước cụ thể
2. **Execute**: Thực thi bước đầu tiên
3. **Replan**: Đánh giá kết quả và cập nhật kế hoạch
   - Nếu xong → Trả lời người dùng
   - Nếu chưa → Tiếp tục execute
4. *(Lặp lại cho đến khi hoàn thành)*

### 🏗️ Kiến trúc

```
      ┌─────────┐
      │ Planner │
      └────┬────┘
           │
           ▼
      ┌─────────┐
      │ Execute │ ← Execute step 1
      └────┬────┘
           │
           ▼
      ┌─────────┐
      │ Replan  │ ← Cập nhật plan
      └────┬────┘
           │
           ├─── Còn bước? ──► Execute
           │
           └─── Xong? ──────► END
```

### 🛠️ Components

**State:**
- `input`: Câu hỏi ban đầu của user
- `plan`: Danh sách các bước cần thực hiện
- `past_steps`: Các bước đã hoàn thành
- `response`: Câu trả lời cuối cùng

**Nodes:**
- `Planner`: Tạo kế hoạch chi tiết (GPT-4o-mini)
- `Agent Executor`: Thực thi từng bước với tools (ReAct agent)
- `Replanner`: Cập nhật kế hoạch dựa trên kết quả (GPT-4o)

**Tools:**
- `TavilySearch`: Tìm kiếm web

### 🚀 Cách chạy
1. Mở file [Plan-and-Execute_Agent.ipynb](Plan-and-Execute_Agent.ipynb)
2. Chạy lần lượt từng cell bằng cách nhấn nút ▶️ Run
3. **Lưu ý:** Sử dụng `async` để chạy nhanh hơn

### 💡 Ví dụ sử dụng

**Example:** Câu hỏi phức tạp
```python
"Con gà có trước hay quả trứng có trước?"
```

**Plan (Kế hoạch ban đầu):**
1. Tìm kiếm thông tin về nguồn gốc con gà
2. Tìm kiếm thông tin về quá trình tiến hóa của trứng
3. So sánh timeline tiến hóa của hai loài
4. Đưa ra kết luận dựa trên khoa học

**Execute Step 1:**
- Task: "Tìm kiếm thông tin về nguồn gốc con gà"
- Result: "Gà xuất hiện từ tiến hóa của loài chim cổ đại..."

**Replan:**
- ✅ Bước 1: Hoàn thành
- Updated Plan: [Bước 2, 3, 4]

**Execute Step 2:**
- Task: "Tìm thông tin về quá trình tiến hóa của trứng"
- Result: "Trứng có vỏ cứng xuất hiện trước gà..."

**Replan:**
- ✅ Bước 1, 2: Hoàn thành
- Đánh giá: Đã đủ thông tin để trả lời
- **Response**: "Theo khoa học, trứng có trước con gà vì..."

### ✅ Ưu điểm
- Có tổ chức: Kế hoạch rõ ràng, dễ theo dõi
- Linh hoạt: Điều chỉnh kế hoạch khi cần
- Phù hợp task phức tạp: Chia nhỏ thành các bước đơn giản
- Khả năng phục hồi: Xử lý được lỗi và thay đổi

### ⚠️ Hạn chế
- Overhead cao: Nhiều API calls (plan + replan + execute)
- Chậm: Phải chờ từng bước hoàn thành
- Over-planning: Có thể tạo plan quá chi tiết

### 🎯 Khi nào sử dụng
- **Task phức tạp**: Cần nhiều bước logic
- **Research tasks**: Tìm kiếm và tổng hợp thông tin
- **Multi-step reasoning**: "So sánh X và Y", "Phân tích A rồi đề xuất B"
- **Khi cần transparency**: Muốn biết agent đang làm gì

---

## 📊 So sánh các Patterns

| Tiêu chí | ReAct | Reflection | Plan-and-Execute |
|----------|-------|------------|------------------|
| **Độ phức tạp task** | Trung bình | Đơn giản | Cao |
| **Số bước thực hiện** | 3-7 bước | 1 task, nhiều vòng | 5-15 bước |
| **Chi phí API** | Trung bình | Cao | Rất cao |
| **Tốc độ** | Nhanh | Trung bình | Chậm |
| **Chất lượng output** | Tốt | Rất tốt | Tốt |
| **Tính minh bạch** | Cao | Trung bình | Rất cao |
| **Use case chính** | Q&A, Tool use | Creative writing | Complex research |

### 🎯 Chọn pattern nào?

**ReAct** - Khi:
- Cần kết hợp nhiều tools
- Task không quá phức tạp
- Cần balance giữa tốc độ và chất lượng

**Reflection** - Khi:
- Chất lượng output là ưu tiên số 1
- Creative tasks (writing, content generation)
- Có thể chấp nhận chi phí cao hơn

**Plan-and-Execute** - Khi:
- Task phức tạp với nhiều bước
- Cần kế hoạch rõ ràng
- Tính chính xác quan trọng hơn tốc độ

---

## 📦 Dependencies

### Core Frameworks
- `langchain==0.3.24` - Framework chính cho LLM applications
- `langchain-openai==0.3.14` - Integration với OpenAI models
- `langgraph==0.3.33` - State machine và graph-based workflows

### Tools & APIs
- `langchain-tavily==0.1.6` - Web search real-time
- `python-dotenv>=1.2.1` - Environment variables management

### Utilities
- `grandalf>=0.8` - Graph visualization library
- `ipykernel>=7.1.0` - Jupyter notebook support

📄 Xem [pyproject.toml](pyproject.toml) để biết danh sách đầy đủ.

---

## 🔧 Lưu ý kỹ thuật

### Performance
- ✅ Tất cả patterns đều nhẹ, không cần GPU
- ✅ Yêu cầu ~500MB RAM
- ⚠️ Plan-and-Execute có thể chậm với task phức tạp (nhiều API calls)

### API Costs
- **ReAct**: ~3-7 API calls/task
- **Reflection**: ~6-12 API calls/task (nhiều vòng reflection)
- **Plan-and-Execute**: ~10-30 API calls/task (tùy độ phức tạp)

### Best Practices
1. **ReAct**: Giới hạn số bước tối đa để tránh infinite loop
2. **Reflection**: Đặt stopping condition hợp lý (3-5 vòng)
3. **Plan-and-Execute**: Set `recursion_limit` cao (50+) cho task phức tạp