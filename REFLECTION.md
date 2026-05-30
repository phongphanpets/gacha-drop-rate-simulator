# Reflection — Gacha Drop Rate Simulator

## ทำจริงเป็นอย่างไร

ใช้ AI (Claude) เป็นคู่คิด-คู่ทำตลอด เริ่มจากอ่านโจทย์ครบทุก tab ก่อนลงมือ แล้วทำงานเป็น phase ที่ชัด: design → mockup → implement → QA review → polish → CSV redesign → submission

แทนที่จะ "ถามแล้วก็อปมา" ผมเลือก approach แบบ pair programming: ถาม AI ให้เสนอตัวเลือก, ผมเลือก/ปรับ direction, AI implement, แล้วผม QA ตรวจสอบกับโจทย์ทุก checkpoint สำคัญ คุยทั้งหมดประมาณ [N] turn จาก design กระดาษ → 1,426-line production-grade simulator

## จุดที่ทำได้ดี

- **กำหนด process ก่อนเริ่ม** — ขอ Claude วาง plan 6 phase ก่อนเขียนโค้ดสักบรรทัด ทำให้รู้ตลอดว่าอยู่ตรงไหนของ rubric และเหลืออะไรอีก
- **เพิ่ม domain knowledge** — โจทย์ขอแค่ basic pity แต่ผมเลือกเพิ่ม TOSM-style soft pity (Rate-Up หลัง roll 80, ตัด N/R) จากประสบการณ์เล่นเกม gacha จริง ทำให้ feature เกินจาก checkbox-list ของ rubric
- **คิดในมุม designer ไม่ใช่แค่ engineer** — เพิ่ม metric อย่าง Cost per SSR / Percentile (P10/P50/P90) / Cost per Unique SSR / Conversion rate / Churn risk ที่ designer และ business owner ใช้จริงในการตัดสินใจ tune rate
- **QA Mindset** — ทุก phase มี test scripts ตรวจ distribution, free roll math, pity logic, regression. เจอ bug จริง (`budget is not defined` ใน `runSimsAsync`) ก่อนส่ง — bug ที่จะทำให้ Monte Carlo crash ตอน reviewer กดปุ่มจริง
- **Iterate ตามที่ใช้จริง** — ลองเล่นไฟล์เองแล้วเจอว่า CSV ใช้ไม่ได้ดี (multi-section ในไฟล์เดียว) จากที่ Claude เสนอแล้วผมเซ็นเอง ผมขอ redesign ใหม่ทั้งหมดเป็น 7 ไฟล์ tabular ที่นำไป pivot/วิเคราะห์ต่อได้

## ติดปัญหาตรงไหน

- **Scope creep** — feature เพิ่มเรื่อยๆ จาก 82 → 130+ ทำให้ต้อง balance ระหว่างคะแนนสูงสุดกับเวลาที่เหลือ ตัดสินใจตัด "Profit Max" preset เดิม (เพราะดู predatory monetization เกินไป) เป็นต้น
- **คิดถึง CSV usability ช้า** — ตอน design CSV ครั้งแรกเป็น multi-section ในไฟล์เดียว (rubric ผ่าน แต่ analyst ใช้ไม่ได้จริง) ต้องลองเล่นเอง + ขอความเห็น Claude อีกรอบ ถึงเจอว่ามันแย่ แล้ว rewrite เป็น 7 ไฟล์ tabular
- **Communicate domain concept ผ่าน AI** — ตอนอธิบาย TOSM pity system ครั้งแรกพูดถึง "ตัวหน้าตู้" ซึ่งระบบเราไม่มี ต้องคุยให้ตรงกันก่อนว่าจะ adapt mechanic ยังไง (เปลี่ยนจาก guaranteed character → rate-up sub-pool)

## AI แนะนำผิดแล้วแก้ยังไง

- **CSV design ครั้งแรก** — AI เสนอ multi-section CSV (metric/value pairs + raw data + config ในไฟล์เดียวกัน) ดูเรียบร้อยตอน implement แต่พอเปิดดูจริงในมุม analyst เห็นว่าใช้ไม่ได้ — Excel filter พัง, pandas ต้อง skip_rows manually แก้: ขอ AI redesign โดยให้ guideline ใหม่ "1 file = 1 table, ครอบคลุม profit + player value ทุก angle, ห้ามประหยัด" ได้ 7 ไฟล์ที่เป็น clean tabular
- **Compare Pity logic** — AI default เปรียบเทียบ Hard Pity ON vs OFF ตลอด แม้ผู้ใช้เปิด Soft Pity อยู่ แก้: ทำให้ dynamic — ถ้า user เปิด Soft Pity → เทียบ Soft ON vs OFF (โดยคง Hard ตาม user setting); ถ้าไม่เปิด → เทียบ Hard
- **Function signature bug** — AI ขยาย `runSimsAsync` ให้เก็บ analytics แต่ใช้ `budget` ที่ไม่ได้อยู่ใน parameter scope แก้: เพิ่ม `budget` เป็น param + update ทุก caller. **ถ้าไม่ได้รัน QA test ก็จะส่งไฟล์ที่ crash จริงตอน reviewer กดปุ่ม Monte Carlo** — เป็นบทเรียนว่าการรัน test script ไม่ใช่ optional
- **`pickRarity` edge case** — รอบแรก AI ใช้ `if (r < cum)` โดยไม่มี fallback ถ้าค่าตรงขอบเป๊ะ (probability ต่ำมากแต่มีโอกาส) แก้: เพิ่ม `return RARITIES[RARITIES.length - 1]` เป็น safety net

## ถ้าทำใหม่จะทำอะไรต่าง

- **คิด CSV format ตั้งแต่ phase 1** — เป็นจุดที่เปลี่ยน design ใหญ่สุดตอนใกล้จบ ถ้า prototype CSV ตั้งแต่แรกคงไม่เสียเวลา redesign
- **เขียน test ก่อน implement บาง logic** — เช่น soft pity sub-pool re-normalization ถ้าเขียน expected test ก่อน implement น่าจะหา edge case ได้เร็วกว่า
- **Track time ของแต่ละ phase** — เพื่อใช้ประมาณการรอบหน้าได้แม่นกว่านี้

## บทเรียนหลัก

ข้อสอบนี้พิสูจน์ว่า "ใช้ AI ให้คุ้ม" ไม่ได้แปลว่าให้ AI ทำทุกอย่าง แต่คือ:
1. **รู้ว่าจะถามอะไร** (มี process / plan ที่ชัด)
2. **รู้ว่าควรถามต่อ vs ลงมือเอง** (เลือก trade-off)
3. **ตรวจของจริง** (test scripts + ลองเล่นเองในฐานะ user)
4. **เอา domain expertise มาเสริม AI** (ระบบ pity, gacha mechanic, analyst needs ที่ generic LLM ไม่รู้)

AI เพิ่มความเร็วในการ implement ได้มหาศาล แต่การตัดสินใจ design และ verification ยังต้องเป็นคนทำ
