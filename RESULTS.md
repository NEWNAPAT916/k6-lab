# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=min=med=max=p(90)=p(95)=260.81 ms (1 request)
- Step 0.2, requests, rate and p(95), and why the rate stays below 5:
  80 requests, 3.83 req/s, p(95) = 268.82 ms. Each VU sends one request
  (about 0.26 s) and then sleeps for 1 s, so one iteration takes about
  1.29 s (iteration_duration avg). Each VU therefore sends only about
  0.78 requests per second, and 5 VUs send about 3.9 requests per second,
  which is below 5. The sleep (think time) is the reason.
- Step 0.6, exit code of the normal run and of the run with P95=300:
  Normal run: exit code 0 (all thresholds passed, p(95) = 372.89 ms < 1500 ms).
  Run with P95=300: exit code 99 (p(95) = 376.22 ms is not < 300 ms, so the
  http_req_duration threshold was crossed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
  All 3 rules passed: http_req_failed rate = X%, p(95) = X ms (< 1000),
  p(99) = X ms (< 1500). After changing p(95)<1000 to p(95)<100,
  the p(95) rule failed and the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it: 
    ✗ no pepperoni
      ↳  80% — ✓ 60 / ✗ 15
    ✗ no knife needed
      ↳  74% — ✓ 56 / ✗ 19
  The request sent 'pepperoni' and 'knife' in lowercase, but the API
  names are 'Pepperoni' and 'Knife' (seen with explore.js), so the
  exclusions did not match and were ignored. The API still answered 200,
  so only the content checks found the problem: X% of 'no pepperoni' and
  Y% of 'no knife needed' checks failed. Fixing the names made all
  checks pass. The API picks a pizza at random, so one manual request
  can happen to return a correct pizza by luck. Many requests under
  load show how often the answer is wrong.

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |       |       |     |        |        |
| A2   | Average load | 3     |18     | avg=269.58ms min=234.51ms med=245.03ms max=358.13ms p(90)=344.45ms p(95)=350.8ms      |  0.00%    |   100.00%     |        |
| A3   | Stress       | 5     | 249   |352.57ms|368.14ms|0.00%  |100.00%|
| A4   | Spike        | 5     | 153      | 352.82ms      |384.73ms     |0.00% |100.00%|
| A5   | Soak         | 2     | 648      |353.76ms       |375.46ms     |0.00% | 100.00%|

## Part B

1.
2.สคริปต์มีการใส่คำสั่ง sleep(1) (คิดเป็นเวลาอย่างน้อย 1 วินาทีต่อ 1 ลูปการทำงานของ Virtual User) ทำให้ผู้ใช้แต่ละคนไม่ได้ส่งคำขอแบบรัวๆ ติดต่อกันทันทีทันใด แต่ละคนมี "ระยะเวลาคิด" (Think Time) ทำให้จำนวนคำขอต่อวินาทีรวมจึงถูกจำกัดโดยธรรมชาติของโค้ด ไม่ได้วิ่งไปถึง 5 requests/sec เต็มที่
3.ค่า p(95) เพิ่มขึ้นเล็กน้อย/ไม่ต่างกันมาก เนื่องจากโหลดสูงสุด 5 VUs ยังอยู่ไกลและใช้งานอยู่ลึก ภายใน Operational Envelope ของระบบ QuickPizza ทำให้เซิร์ฟเวอร์ยังคงตอบสนองได้รวดเร็วและไม่เกิดอาการคอขวด
4.ค่า Response Time เพิ่มขึ้นช่วงสั้นๆ ตอนเกิด Spike และใช้เวลาประมาณ 5–10 วินาที (หรือกลับสู่ระดับปกติทันที) หลังจากลดผู้ใช้ลงมาเหลือ 1 VU
5.ทั้ง Response Time และ Error Rate ไม่แย่ลงตลอดช่วงเวลา 10 นาที ตัวอย่าง Defect ที่ Soak Test ระยะยาวตรวจพบได้คือ Memory Leak
6.เพราะค่าเฉลี่ย (Average) สามารถถูกบิดเบือนโดยค่าส่วนใหญ่ได้ และบดบังผู้ใช้กลุ่มที่เจอเว็บช้ามากๆ (Outliers) การใช้ p(95)ช่วยการันตีว่าผู้ใช้อย่างน้อย 95% ได้รับประสบการณ์การใช้งานที่เร็วตามเกณฑ์ที่กำหนด
7.-Network Latency & Bandwidth: อินเทอร์เน็ตสาธารณะมีความหน่วงและความผันผวนสูงกว่าเครือข่ายความเร็วสูงภายในแลป
-External Traffic Noise: QuickPizza มีผู้ใช้งานอื่นจากทั่วโลกทดสอบพร้อมกัน ทำให้ภาระเครื่องเปลี่ยนไปมาตลอดเวลา ต่างจากแลปที่เป็นระบบปิด
8.เพราะการส่งโหลดมหาศาลอาจทำให้ระบบเป้าหมายล่ม รบกวนผู้ใช้อื่น และถือเป็นการโจมตีระบบที่ผิดกฎหมายและผิดจริยธรรมหากไม่ได้รับอนุญาต
9.
- Vague Requirement: "แอปสั่งอาหารต้องโหลดหน้าแรกให้เร็วและไม่ค้าง"
- Testable Requirement: "เมื่อมีผู้ใช้พร้อมกัน 10 คน หน้าแรกต้องตอบสนองภายใน 1,000 ms สำหรับ 95% ของคำขอ และมีอัตราความผิดพลาดต่ำกว่า 1%"
K6 Thresholds:
- thresholds: {
  'http_req_duration': ['p(95)<1000'],
  'http_req_failed': ['rate<0.01'],
}