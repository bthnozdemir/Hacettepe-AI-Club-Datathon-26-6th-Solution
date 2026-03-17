# Hacettepe AI Club Datathon 2026 - 6th Place Solution

**Textile Dyeing Process Level Prediction with Temporal Stacking**

Final etkinlik sunumu: [Canva Link](https://www.canva.com/design/DAHDtlUSca8/5tkzbJ1fwbV4-f4gTRl49A/view?utm_content=DAHDtlUSca8&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h87eabbba23)

---

## Problem

Tekstil boyama proseslerinde kazan seviye değerlerini (`bk_level`) önceden tahmin ederek üretim planlamasını optimize etmek.

**Veri:** 2.1M satır, 1,878 proses, 16,032 blok  
**Metrik:** Weighted MAE (wMAE)  
**Hedef:** Blok ortalaması (`bk_mean`) tahmini

---

## Çözüm Özeti

### 1 — Veri Dönüşümü: Satır → Blok
Ham saniyelik veriyi **blok aggregasyonu** ile özetledik:
- Dinamizm tespiti: 5-dönemlik std > threshold
- Gap tespiti: Zaman farkı > 10-30 saniye
- Her blok: ortalama, std, first, last, min, max, target

### 2 — Feature Engineering (130+ özellik)
- **Temporal:** lag1-5, delta, accel, jerk, rolling mean/std
- **Target tracking:** hedef sapması, momentum, ETA
- **Gap features:** log(gap), gap×target, gap×dynamic
- **Categorical:** machine_id, command_no, program
- **Valve ratios:** bk/kk irtibat, fast/slow dosage

### 3 — Warmup Lookup (İlk Blok Sorunu)
Proses başında lag=0 → model kör.  
**Çözüm:** Her (machine_id, command_no) çifti için geçmiş ilk blok ortalamalarını lookup tablosu.  
**Kazanç:** MAE 33.69 → 16.98 (%27.7 azalma)

### 4 — Temporal Cross-Validation
5-fold temporal split (proses start time bazlı) → veri sızıntısı engellendi.

### 5 — Stacking Ensemble
- **Base models:** LightGBM×2, XGBoost×2, CatBoost×2, Ridge×2
- **Meta model:** Ridge (sample_weight=n_rows)
- **Sonuç:** CatBoost2 dominant (coef +0.588), wMAE 0.3965

---

## Sonuçlar

| Komut | wMAE | Blok | Satır | Veri Payı |
|-------|------|------|-------|-----------|
| 19 | 0.4812 | 5,164 | 291,029 | 13.7% |
| 20 | **1.4772** ⚠️ | 276 | 3,366 | **0.16%** |
| 21 | 0.4099 | 6,666 | 179,076 | 8.5% |
| 22 | **0.3777** ✅ | 3,926 | **1,637,351** | **77.5%** |
| **GENEL** | **0.3965** | 16,032 | 2,110,822 | 100% |

**Kritik bulgu:** Komut 20 az veri nedeniyle güvenilmez.

---

## 💡 Öğrendiklerim

 Warmup lookup ile cold start çözüldü  
 CatBoost kategorik özellikler için baskın  
 Temporal fold sızıntı engelledi  
 wMAE > MAE (blok boyutu önemli)  
 Az veri segmentleri risk yaratıyor  
 Blok aggregation hız kazandırdı ama saniye hassasiyeti kaybettirdi

---
