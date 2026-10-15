# Asynchronous-FIFO-Gray-Code-CDC

## 📌 1. Project Title
✅ **Asynchronous FIFO for Clock Domain Crossing (CDC) using Gray-Code Pointers**

---

## 📝 2. Description
This project implements an **Asynchronous FIFO (First-In First-Out buffer)** in Verilog to safely transfer data between **two independent clock domains**:

- ✍️ **Write clock domain** (`wr_clk`)
- 📖 **Read clock domain** (`rd_clk`)

To prevent metastability and ensure reliable CDC operation, the FIFO uses:
- 🧩 **Binary pointers** for memory indexing
- 🔘 **Gray-code pointers** for safe pointer transfer across clock domains
- 🧯 **2-flop synchronizers** for CDC
- 🚦 **Full** and **Empty** status flag generation

---

## ⭐ 3. Key Features
- ⚙️ Parameterized **depth** and **data width**
- ⏱️ Independent write/read clocks (true async FIFO)
- 🔘 Gray-code read/write pointers
- 🧯 Two-stage synchronizers for pointer crossing
- 🚦 `full` / `empty` flag generation
- 🎛️ Chip select (`cs`) gating for read/write operations

---

## 🔌 4. Module Interface

### 📄 Module Declaration
```verilog
module asy_fifo(cs,wr_clk,rd_clk,rst,
        wr_en,data_in,full,
        rd_en,data_out,empty
);
```

### 🧾 Ports
| Signal     | Dir | Width | Description |
|-----------|-----|-------|-------------|
| `cs`      | In  | 1     | 🎛️ Chip select (enables FIFO access) |
| `wr_clk`  | In  | 1     | ✍️ Write clock domain |
| `rd_clk`  | In  | 1     | 📖 Read clock domain |
| `rst`     | In  | 1     | 🔄 Reset (applied in both domains) |
| `wr_en`   | In  | 1     | ✍️ Write enable |
| `data_in` | In  | `FIFO_WIDTH` | 📨 Input write data |
| `full`    | Out | 1     | 🚫 FIFO full flag |
| `rd_en`   | In  | 1     | 📖 Read enable |
| `data_out`| Out | `FIFO_WIDTH` | 📤 Output read data |
| `empty`   | Out | 1     | 🈳 FIFO empty flag |

---

## ⚙️ 5. Parameters
| Parameter | Default | Meaning |
|----------|---------|---------|
| `FIFO_DEPTH` | 8 | 📦 Number of FIFO entries (**power of 2 recommended**) |
| `FIFO_WIDTH` | 32 | 🧱 Data width |
| `FIFO_DEPTH_LOG` | `$clog2(FIFO_DEPTH)` | 📍 Address width |

---

## 🧠 6. Internal Design (How It Works)

### ✍️ Write Domain
- Maintains:
  - `wr_ptr_bin` (binary pointer)
  - `wr_ptr_gray` (gray pointer)
- On `wr_clk` rising edge:
  - If `cs && wr_en && !full` ✅
    - Writes `data_in` into memory
    - Advances write pointer (binary + gray)

### 📖 Read Domain
- Maintains:
  - `rd_ptr_bin` (binary pointer)
  - `rd_ptr_gray` (gray pointer)
- On `rd_clk` rising edge:
  - If `cs && rd_en && !empty` ✅
    - Reads from memory into `data_out`
    - Advances read pointer (binary + gray)

---

## 🧯 7. CDC (Clock Domain Crossing) Protection
To safely compare pointers across clock domains:

- 🟦 Read pointer (Gray) is synchronized into write domain:
  - `rd_gray_sync1`, `rd_gray_sync2` (2-flop sync on `wr_clk`)

- 🟥 Write pointer (Gray) is synchronized into read domain:
  - `wr_gray_sync1`, `wr_gray_sync2` (2-flop sync on `rd_clk`)

This reduces metastability risk when transferring pointer values between clocks.

---

## 🚦 8. Full / Empty Logic

### 🈳 Empty Condition
FIFO is empty when:
```verilog
empty = (rd_ptr_gray == wr_gray_sync2);
```
✅ Meaning: read pointer caught up to synchronized write pointer.

### 🚫 Full Condition
FIFO is full when next write pointer equals the read pointer with inverted MSBs:
```verilog
full = (wr_ptr_gray_next ==
        {~rd_gray_sync2[FIFO_DEPTH_LOG:FIFO_DEPTH_LOG-1],
          rd_gray_sync2[FIFO_DEPTH_LOG-2:0]});
```
✅ Standard async FIFO full detection method.

---

## 📁 9. Files
- 📄 `asy_fifo.v` — Asynchronous FIFO module (Gray pointers + CDC synchronizers)

---

## 🧪 10. Suggested Testing (Optional)
To verify correct behavior ✅:
- ✍️ Write faster than read → FIFO should assert `full`
- 📖 Read faster than write → FIFO should assert `empty`
- 🔄 Apply reset and confirm:
  - pointers return to 0
  - `empty` becomes 1 (after sync settles)
- ⏱️ Random write/read enables under unrelated clocks

