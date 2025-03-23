# การวิเคราะห์ข้อมูล LOB และการพัฒนากลยุทธ์เทรดระยะสั้นสำหรับ S50 Futures
## โครงสร้างไฟล์
```
.
├── split_data                 # โค้ดหลักของแอป
├── EDA.py 
├── mt5_data_XAUUSD_TF1H_FUB_TH.csv  # ข้อมูลราคาทองคำ
├── daily_sentiment.csv    # ข้อมูล Sentiment รายวัน
└── requirements.txt       # ไลบรารีที่ต้องติดตั้ง
```
## โครงสร้างไฟล์
```
.
├── submission-folder/                    # Main application code folder 
   └── split_data.ipynb           # Jupyter Notebook for data splitting
         ├── in_sample.csv                  # In-sample data file
         ├── out_sample.csv                 # Out-of-sample data file
   ├── EDA.py                         # Exploratory Data Analysis code
   ├── Strategy_in_sample.py          # Trading strategy for in-sample data
   ├── Strategy_out_sample.py         # Trading strategy for out-of-sample data
   ├── Performance_Metrics_in_sample.py      # Performance metrics for in-sample data
   ├── Performance_Metrics_out_sample.py     # Performance metrics for out-of-sample data
   └── requirements.txt               # Required libraries for the project

```

## แนวคิดหลัก
กลยุทธ์ของผมมุ่งเน้นไปที่การใช้ข้อมูล High-Frequency จาก Limit Order Book (LOB) และข้อมูลการเทรด เพื่อค้นหาโอกาสในตลาดที่มีความไร้ประสิทธิภาพในระดับ microstructure โดยเฉพาะ:
- **Order Imbalance:** ความไม่สมดุลในปริมาณคำสั่งซื้อและขายในแต่ละระดับ
- **Liquidity Imbalance:** การขาดสภาพคล่องที่สมดุลระหว่าง Bid กับ Ask
- **Spread Dynamics:** การแกว่งของ spread ในช่วงเวลาสั้น ๆ

ข้อมูลเหล่านี้ช่วยให้สามารถจับจังหวะเข้าออกตลาดได้แม้ผลกำไรต่อเทรดจะเล็กน้อย แต่เมื่อเทรดในความถี่สูงอย่างต่อเนื่อง ก็สามารถสะสมผลตอบแทนที่น่าสนใจได้

## กระบวนการคิดและขั้นตอนการดำเนินงาน

1. **การวิเคราะห์ข้อมูลเชิงสำรวจ (EDA):**
   - วิเคราะห์ราคาที่ดีที่สุด (Bid1, Ask1) และปริมาณในแต่ละระดับ (vBid1–vBid5, vAsk1–vAsk5)
   - คำนวณค่าเฉลี่ยของปริมาณ (Average Bids/Asks Volume) เพื่อหาสัญญาณความไม่สมดุล (Imbalance)
   - ศึกษาการเปลี่ยนแปลงของ spread และความผันผวนในข้อมูล

2. **การพัฒนากลยุทธ์เทรด:**
   - ใช้สัญญาณจาก Imbalance (แสดงด้วยสีเขียว/แดง) เป็นตัวบ่งชี้การเปิดและปิดสถานะ
     - **เปิด Long:** เมื่อสัญญาณเป็นสีเขียว (บ่งบอกความได้เปรียบของฝั่งซื้อ)
     - **เปิด Short:** เมื่อสัญญาณเป็นสีแดง (บ่งบอกความได้เปรียบของฝั่งขาย)
   - ตั้ง Stop Loss ไว้ที่ 5 จุดและคำนวณ Lot Size โดยคำนึงถึงทุนที่เสี่ยง (1% ของทุน) เพื่อบริหารความเสี่ยง

3. **Backtesting และประเมินประสิทธิภาพ:**
   - ทดสอบกลยุทธ์กับข้อมูลย้อนหลัง (in-sample) และเก็บ Trade Log พร้อมจุดเข้า/ออก (Entry/Exit)
   - ประเมินประสิทธิภาพด้วยตัวชี้วัดต่าง ๆ เช่น:
     - **Sharpe Ratio**
     - **Maximum Drawdown (MDD)**
     - **Profit Factor**
     - **Win Rate**
     - **Expectancy (ผลตอบแทนเฉลี่ยต่อเทรด)**
     
4. **การบริหารความเสี่ยง:**
   - คำนวณค่าใช้จ่ายในการเทรด เช่น ค่าคอมมิชชั่นและค่าธรรมเนียมตลาด
   - นำผลกระทบจาก Slippage มาพิจารณาใน Backtest
   - ใช้ Stop Loss เพื่อจำกัดขาดทุนและปรับ Lot Size ให้เหมาะสมกับระดับความเสี่ยงที่รับได้

## เหตุผลที่เลือกใช้กลยุทธ์นี้
- **ความละเอียดของข้อมูล:** ตลาด S50 Futures มีสภาพคล่องสูงและข้อมูล LOB ที่ละเอียด ทำให้สามารถจับสัญญาณความไม่สมดุลในระดับ microstructure ได้
- **การจับจังหวะในตลาด:** แม้ผลกำไรต่อเทรดจะเล็กน้อย แต่การเทรดแบบ high-frequency ช่วยให้สามารถรวมผลกำไรหลาย ๆ เทรดเข้าด้วยกันและสร้างผลตอบแทนที่น่าสนใจได้
- **ระบบบริหารความเสี่ยงที่เข้มงวด:** การกำหนด Stop Loss, การคำนวณ Lot Size ตามทุนที่เสี่ยง และการพิจารณาค่าใช้จ่ายในการเทรด ช่วยลดความเสี่ยงและปกป้องทุนได้

## การใช้ประโยชน์จากความไร้ประสิทธิภาพในตลาด
กลยุทธ์นี้ใช้ประโยชน์จาก **Market Microstructure Inefficiency** โดยเฉพาะ:
- **Order Imbalance:** ถ้าปริมาณ Bid สูงกว่า Ask หรือในทางกลับกัน ราคามักจะมีแนวโน้มเคลื่อนที่ตามทิศทางที่ไม่สมดุลนั้น
- **Liquidity Imbalance:** เมื่อมีการจัดเรียงคำสั่งซื้อขายไม่สมดุล ราคาจะเคลื่อนไหวออกจากมูลค่าที่แท้จริงได้

การใช้สัญญาณเหล่านี้ช่วยให้สามารถเข้าออกตลาดได้อย่างแม่นยำในช่วงเวลาสั้น ๆ ซึ่งเป็นจุดแข็งของกลยุทธ์นี้



## สรุป
กลยุทธ์ที่พัฒนานี้ผสมผสานการวิเคราะห์ข้อมูล LOB ด้วยความละเอียดสูงกับการออกแบบกฎการเทรดที่ชัดเจน พร้อมระบบบริหารความเสี่ยงที่เข้มงวด ทำให้สามารถจับจังหวะการเทรดในสภาวะตลาดที่มีความไร้ประสิทธิภาพในระดับ microstructure ได้อย่างมีประสิทธิภาพและสร้างผลตอบแทนที่น่าสนใจในระยะสั้น

---

*หมายเหตุ: รายงานนี้เป็นการสรุปแนวความคิดและกระบวนการคิดเบื้องต้นในการพัฒนากลยุทธ์เทรดที่ใช้ข้อมูล LOB ซึ่งสามารถปรับปรุงรายละเอียดเพิ่มเติมตามสภาพตลาดและข้อมูลจริงที่ใช้งานได้*






# Analysis of LOB Data and Development of a Short-Term Trading Strategy for S50 Futures

## Core Concept
My strategy focuses on utilizing high-frequency data from the Limit Order Book (LOB) along with trade data to exploit market inefficiencies at the microstructure level. Specifically, I target:

- **Order Imbalance:** Detecting discrepancies in the volumes on the bid and ask sides.
- **Liquidity Imbalance:** Recognizing situations where liquidity is unevenly distributed between bids and asks.
- **Spread Dynamics:** Capitalizing on short-term fluctuations in the bid-ask spread.

These inefficiencies create opportunities to enter and exit the market rapidly. Although each trade might yield a small profit, frequent trading can lead to significant overall returns.

## Process and Methodology

### 1. Exploratory Data Analysis (EDA)
- **Data Understanding:**  
  - Analyze key LOB data points such as the best bid (Bid1), best ask (Ask1), and the volume at various levels.
  - Examine the spread dynamics and volatility in the data.
- **Signal Generation:**  
  - Calculate average volumes on the bid and ask sides (Average Bids/Asks Volume) to detect imbalances.
  - Use these imbalances as trading signals.

### 2. Strategy Development
- **Signal-Based Entry/Exit Rules:**  
  - **Long Position:** Open when the imbalance signal is positive (green), indicating a stronger bid side.
  - **Short Position:** Open when the imbalance signal is negative (red), indicating a stronger ask side.
- **Risk Management:**  
  - Set a Stop Loss (SL) at an appropriate distance (e.g., 5 points) to limit downside risk.
  - Calculate the appropriate lot size based on risk per trade (e.g., 1% of capital).
- **Execution Considerations:**  
  - Ensure the strategy handles high-frequency data efficiently by avoiding simultaneous open and close orders in the same time unit.

### 3. Backtesting and Performance Evaluation
- **Historical Testing:**  
  - Backtest the strategy on in-sample historical data.
- **Performance Metrics:**  
  - Evaluate using measures such as Sharpe Ratio, Maximum Drawdown, Profit Factor, Win Rate, and Expectancy.

### 4. Risk Management
- **Transaction Costs:**  
  - Incorporate commissions, exchange fees, and slippage into the backtest to ensure realistic performance.
- **Position Sizing:**  
  - Adjust lot sizes based on the defined risk parameters (e.g., risking 1% of total capital per trade).
- **Stop Loss Management:**  
  - Use Stop Loss orders to limit adverse movements and protect capital.

## Rationale for Choosing This Strategy
- **High Liquidity & Detailed Data:**  
  S50 Futures markets provide high liquidity and detailed LOB data, which allow for precise timing when entering and exiting trades, even with small price differentials.
- **Exploitation of Microstructure Inefficiencies:**  
  The strategy takes advantage of market microstructure inefficiencies (order and liquidity imbalances) to capture short-term opportunities.
- **Risk Management:**  
  A robust risk management framework (including Stop Loss, proper lot sizing, and transaction cost integration) helps protect capital and reduce overall risk.
- **High-Frequency Advantage:**  
  By executing trades at a high frequency, even small profits per trade can accumulate to generate attractive overall returns.

## Exploiting Market Inefficiencies
The strategy leverages market microstructure inefficiencies, specifically:
- **Order Imbalance:**  
  When there is an excess of bid orders relative to ask orders (or vice versa), prices may deviate from their fair value.
- **Liquidity Imbalance:**  
  Uneven liquidity distribution can lead to temporary mispricings that the strategy can exploit.

By identifying and reacting to these signals, the strategy is designed to take advantage of rapid price movements in the market.

---

*This document summarizes the conceptual framework and methodology behind the trading strategy developed using LOB data. The approach combines detailed exploratory analysis, a signal-based trading system, rigorous backtesting, and comprehensive risk management to exploit short-term market inefficiencies effectively.*

