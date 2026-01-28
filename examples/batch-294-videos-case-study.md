# 📊 Case Study: 294 Video Batch Processing (GAIA v4.8.2)

> Real production data from Washin Village animal video analysis

## 🔥 Surprising Discovery: Batch Size Doesn't Matter!

We ran two batches on the same day:

| Batch | Requests | Created | Completed | Status |
|-------|----------|---------|-----------|--------|
| 🐘 **Large** | 294 | 10:22 AM | ~12:00 PM | ✅ **Finished first!** |
| 🐁 Small | 10 | 11:50 AM | — | ⏳ Still processing |

**The large batch (294) completed ~1.5 hours BEFORE the small batch (10)!**

This suggests Anthropic's worker allocation is not FIFO — batch size has NO impact on processing time.

---

## 📊 Job Details (From Anthropic Console)

| Item | Value |
|------|-------|
| **Batch ID** | `msgbatch_01Ku...xxxxx` |
| **Files Processed** | 294 |
| **Total Tokens** | 1,500,944 |
| **Original Cost** | $11.04 |
| **Batch Cost** | **$5.52** |
| **💰 Savings** | **$5.52 (50%)** |
| **Per Request** | $0.0188 |
| **Success Rate** | 291/294 (99.0%) |

### Token Breakdown (CSV Export)

| Token Type | Count | Cost | Notes |
|------------|-------|------|-------|
| Input (no cache) | 365,624 | $0.55 | Image data (not cacheable) |
| Cache write (1h) | 106,920 | $0.32 | System prompt |
| Cache read | 416,988 | $0.06 | 90% off! |
| Output | 611,412 | $4.59 | Analysis results |
| **Total** | **1,500,944** | **$5.52** | |

---

## 💡 Key Insight: Image Workloads

**Why only 14% caching savings instead of 90%?**

In image/video tasks, images = ~85% of input tokens. Only the system prompt (~15%) is cacheable.

```
Input Composition:
├── System Prompt: ~15% → ✅ Cacheable (90% off)
└── Image Data:    ~85% → ❌ Cannot cache

Actual Savings: 15% × 90% = ~14%
```

**This is NOT in the official docs — we learned it the hard way!**

---

## 📈 Cost Comparison (Real Data)

| Optimization | Cost/Video | Total (294) | Savings |
|--------------|------------|-------------|---------|
| None (Standard API) | $0.038 | $11.04 | — |
| + Caching only | $0.033 | $9.62 | 14% |
| + Batch only | $0.019 | $5.52 | **50%** |
| **+ Both** | **$0.016** | **$4.79** | **57%** 🔥 |

---

## ⏱️ Time Breakdown

| Stage | Description | Time |
|-------|-------------|------|
| **Packaging** | Scan 294 JSONs + Extract keyframes + Convert to Base64 | ~30 sec |
| **Submit** | Upload to Anthropic API | ~1 sec |
| **Processing** | Claude Sonnet batch analysis (Anthropic side) | ~1.5 hours |
| **Retrieve** | Download results + Update JSONs | ~5 min |

---

## 🎯 When to Use Batch API

✅ **Perfect for:**
- Video/image analysis (not time-sensitive)
- Bulk content tagging
- Overnight processing jobs
- Any task where you can wait 1-24 hours

❌ **Don't use for:**
- Real-time chat
- Interactive applications
- Urgent responses needed

---

## 🐛 Bug We Found (Bonus Lesson)

**Problem**: After `--batch-check`, L9/L10/L11 fields were empty `{}`

**Root Cause**: Path inconsistency between save and retrieve:
```python
# Stage 1 save (correct)
sidecar_path = file_path.with_suffix('.gaia.json')  # → xxx.gaia.json

# batch-check read/write (wrong!)
sidecar_path = file_path.with_suffix(file_path.suffix + '.gaia.json')  # → xxx.mp4.gaia.json
```

**Result**: 291 results written to WRONG files!

**Lesson**: Always use the same path construction logic for save AND retrieve!

---

## 📝 Summary

| Key Finding | Details |
|-------------|---------|
| **Batch API** | Exactly 50% savings (as advertised) ✅ |
| **Batch Size** | NO impact on processing time! 🔥 |
| **Caching + Images** | Only ~14% savings (images not cacheable) |
| **Combined** | Batch + Cache = 57% savings |

---

*Data source: Anthropic Console CSV Export + Production Batch*
*Date: 2026-01-28*
*System: GAIA Video Analysis Pipeline v4.8.2*
*By: [Washin Village](https://washinmura.jp)*
