# Chapter 6.4: Applications of Percentages

[← Previous: Percentage Increase and Decrease](03-percentage-increase-decrease.md) | [Back to Contents](../README.md) | [Next: Understanding Ratios →](../07-Ratio-and-Proportion/01-understanding-ratios.md)

---

## 🎯 Learning Objectives

By the end of this chapter, you will be able to:
- Apply percentages to financial calculations
- Calculate simple and compound interest
- Work with discount and markup problems
- Understand percentages in statistics and data

---

## 1. Profit and Loss

### Key Definitions
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Cost Price (CP): What you pay to buy/make something   │
│   Selling Price (SP): What you sell it for              │
│                                                         │
│   If SP > CP → PROFIT (Gain)                            │
│   If SP < CP → LOSS                                     │
│   If SP = CP → No profit, No loss                       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Formulas
```
Profit = Selling Price - Cost Price
Loss = Cost Price - Selling Price

Profit % = (Profit ÷ Cost Price) × 100
Loss % = (Loss ÷ Cost Price) × 100

Important: Percentage is ALWAYS calculated on Cost Price!
```

### Finding Selling Price
```
When profit % is known:
SP = CP × (1 + Profit%/100)

When loss % is known:
SP = CP × (1 - Loss%/100)
```

### Example: Profit Calculation
```
A shopkeeper buys a watch for $80 and sells it for $100.

Profit = $100 - $80 = $20

Profit % = (20 ÷ 80) × 100 = 25%
```

### Example: Loss Calculation
```
A phone bought for $600 is sold for $480.

Loss = $600 - $480 = $120

Loss % = (120 ÷ 600) × 100 = 20%
```

### Example: Finding Selling Price
```
Cost price = $150
Desired profit = 30%

SP = $150 × (1 + 0.30)
   = $150 × 1.30
   = $195
```

---

## 2. Discount and Marked Price

### Key Definitions
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Marked Price (MP): The listed/tag price               │
│   Discount: Reduction from marked price                 │
│   Selling Price (SP): What customer actually pays       │
│                                                         │
│   SP = MP - Discount                                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Formulas
```
Discount = Marked Price × (Discount%/100)
Selling Price = Marked Price × (1 - Discount%/100)

Discount % = (Discount ÷ Marked Price) × 100
```

### Example: Finding Sale Price
```
Marked price: $120
Discount: 25%

Discount amount = $120 × 0.25 = $30
Selling price = $120 - $30 = $90

OR: SP = $120 × 0.75 = $90
```

### Example: Finding Original Price
```
After 30% discount, an item costs $56.
Find the marked price.

$56 = 70% of marked price
MP = $56 ÷ 0.70 = $80
```

### Successive Discounts
```
Two discounts of 20% and 10% (one after another):

Method 1: Step by step
Original: $100
After 20% off: $100 × 0.80 = $80
After 10% off: $80 × 0.90 = $72

Method 2: Combined multiplier
$100 × 0.80 × 0.90 = $100 × 0.72 = $72

Equivalent to 28% discount (not 30%!)
```

---

## 3. Simple Interest

### The Formula
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│         Simple Interest (SI) = (P × R × T) / 100        │
│                                                         │
│   P = Principal (initial amount)                        │
│   R = Rate of interest (% per year)                     │
│   T = Time (in years)                                   │
│                                                         │
│         Total Amount = Principal + Interest             │
│                A = P + SI                               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Key Feature of Simple Interest
```
Interest is calculated ONLY on the original principal.

Year 1: Interest on P
Year 2: Interest on P
Year 3: Interest on P
...

Interest is the SAME each year.
```

### Example 1: Finding Interest
```
Principal: $5,000
Rate: 8% per year
Time: 3 years

SI = (P × R × T) / 100
   = (5000 × 8 × 3) / 100
   = 120000 / 100
   = $1,200

Total amount = $5,000 + $1,200 = $6,200
```

### Example 2: Finding Rate
```
Principal: $2,000
Interest earned: $240
Time: 2 years

R = (SI × 100) / (P × T)
  = (240 × 100) / (2000 × 2)
  = 24000 / 4000
  = 6% per year
```

### Example 3: Finding Time
```
Principal: $4,000
Rate: 5%
Interest earned: $800

T = (SI × 100) / (P × R)
  = (800 × 100) / (4000 × 5)
  = 80000 / 20000
  = 4 years
```

---

## 4. Compound Interest

### The Concept
```
Interest is calculated on:
Principal + Previously accumulated interest

The interest GROWS each period because the base grows.
```

### The Formula
```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│          Amount = P × (1 + R/100)ⁿ                      │
│                                                         │
│   P = Principal                                         │
│   R = Rate per period (% per period)                    │
│   n = Number of periods                                 │
│                                                         │
│   Compound Interest = Amount - Principal                │
│                 CI = A - P                              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Comparison: Simple vs Compound
```
Principal: $1,000
Rate: 10% per year
Time: 3 years

SIMPLE INTEREST:
Year 1: $1,000 × 0.10 = $100    Total: $1,100
Year 2: $1,000 × 0.10 = $100    Total: $1,200
Year 3: $1,000 × 0.10 = $100    Total: $1,300

Total Interest: $300

COMPOUND INTEREST:
Year 1: $1,000 × 0.10 = $100    Total: $1,100
Year 2: $1,100 × 0.10 = $110    Total: $1,210
Year 3: $1,210 × 0.10 = $121    Total: $1,331

Total Interest: $331

Compound gives $31 more!
```

### Example: Compound Interest
```
Principal: $5,000
Rate: 8% compounded annually
Time: 2 years

A = P × (1 + R/100)ⁿ
  = $5,000 × (1.08)²
  = $5,000 × 1.1664
  = $5,832

CI = $5,832 - $5,000 = $832

(Simple interest would be: $5,000 × 0.08 × 2 = $800)
```

---

## 5. Tax Calculations

### Sales Tax
```
Sale price (before tax): $80
Tax rate: 7.5%

Tax amount = $80 × 0.075 = $6

Total price = $80 + $6 = $86

OR: Total = $80 × 1.075 = $86
```

### Value Added Tax (VAT)
```
Price excluding VAT: $200
VAT rate: 15%

VAT = $200 × 0.15 = $30
Price including VAT = $200 + $30 = $230
```

### Finding Pre-Tax Price
```
Price including 8% tax: $54

Pre-tax price = $54 ÷ 1.08 = $50

Tax amount = $54 - $50 = $4
```

### Income Tax Example
```
Taxable income: $50,000
Tax brackets:
- 10% on first $10,000
- 20% on next $30,000
- 30% on remainder

Tax calculation:
First $10,000: $10,000 × 0.10 = $1,000
Next $30,000: $30,000 × 0.20 = $6,000
Remaining $10,000: $10,000 × 0.30 = $3,000

Total tax: $1,000 + $6,000 + $3,000 = $10,000
Effective rate: ($10,000 ÷ $50,000) × 100 = 20%
```

---

## 6. Commission and Tips

### Commission
```
Commission: Percentage of sales earned by salesperson

Sales: $15,000
Commission rate: 6%

Commission = $15,000 × 0.06 = $900
```

### Tiered Commission
```
Commission structure:
- 5% on first $10,000
- 8% on sales above $10,000

Sales: $18,000

First tier: $10,000 × 0.05 = $500
Second tier: $8,000 × 0.08 = $640

Total commission: $500 + $640 = $1,140
```

### Tips/Gratuity
```
Restaurant bill: $75
Tip: 18%

Tip amount = $75 × 0.18 = $13.50

Total = $75 + $13.50 = $88.50
```

### Quick Tip Calculations
```
For 15% tip:
- Find 10% (move decimal): $75 → $7.50
- Find 5% (half of 10%): $7.50 ÷ 2 = $3.75
- Add: $7.50 + $3.75 = $11.25

For 20% tip:
- Find 10%: $7.50
- Double it: $7.50 × 2 = $15.00
```

---

## 7. Percentage in Data and Statistics

### Percentage Distribution
```
Survey results (100 people):
- Agree: 45 people = 45%
- Disagree: 35 people = 35%
- Neutral: 20 people = 20%

Total: 100% ✓
```

### Pie Chart Representation
```
Budget breakdown ($2,000 monthly):

         ╭──────────────────╮
        ╱                    ╲
       ╱      Rent 40%        ╲
      ╱        $800            ╲
     │ ─────────────────────── │
     │  Food    │    Savings   │
     │  25%     │     15%      │
     │  $500    │    $300      │
      ╲         │             ╱
       ╲  Other 20% ($400)   ╱
        ╲                   ╱
         ╰─────────────────╯

Rent: 40% = $800
Food: 25% = $500
Savings: 15% = $300
Other: 20% = $400
```

### Percentage Change in Data
```
Monthly sales comparison:
January: 1,200 units
February: 1,380 units

Change = 1,380 - 1,200 = 180 units
% Change = (180 ÷ 1,200) × 100 = 15% increase
```

### Percentage of Total
```
Department sales:
Electronics: $45,000
Clothing: $30,000
Home goods: $25,000
Total: $100,000

Electronics: (45000 ÷ 100000) × 100 = 45%
Clothing: (30000 ÷ 100000) × 100 = 30%
Home goods: (25000 ÷ 100000) × 100 = 25%
```

---

## 8. Depreciation

### Concept
```
Depreciation: Decrease in value over time

Common for: Cars, electronics, machinery, buildings
```

### Straight-Line Depreciation
```
Original value: $20,000
Depreciation: 10% per year (of original)

Annual depreciation = $20,000 × 0.10 = $2,000

Year 1: $20,000 - $2,000 = $18,000
Year 2: $18,000 - $2,000 = $16,000
Year 3: $16,000 - $2,000 = $14,000
```

### Reducing Balance (Compound) Depreciation
```
Original value: $20,000
Depreciation: 15% per year (of current value)

Year 1: $20,000 × 0.85 = $17,000
Year 2: $17,000 × 0.85 = $14,450
Year 3: $14,450 × 0.85 = $12,282.50

After 3 years: Value = $20,000 × (0.85)³ = $12,282.50
```

---

## 9. Solved Examples

### Example 1: Complete Sale Problem
```
A merchant buys goods for $800, marks them up by 50%,
then offers a 20% discount. Find profit percentage.

Cost price: $800
Marked price: $800 × 1.50 = $1,200
Selling price: $1,200 × 0.80 = $960

Profit = $960 - $800 = $160
Profit % = (160 ÷ 800) × 100 = 20%
```

### Example 2: Simple Interest Problem
```
At what rate will $2,500 become $3,250 in 5 years
at simple interest?

Interest = $3,250 - $2,500 = $750

R = (SI × 100) / (P × T)
  = (750 × 100) / (2500 × 5)
  = 75000 / 12500
  = 6% per year
```

### Example 3: Compound Interest
```
Find compound interest on $10,000 at 5%
compounded annually for 3 years.

A = P(1 + R/100)ⁿ
  = $10,000 × (1.05)³
  = $10,000 × 1.157625
  = $11,576.25

CI = $11,576.25 - $10,000 = $1,576.25
```

### Example 4: Tax Problem
```
An item's price including 12% VAT is $560.
Find the price before VAT and the VAT amount.

Pre-VAT price = $560 ÷ 1.12 = $500
VAT amount = $560 - $500 = $60

Check: $500 × 1.12 = $560 ✓
```

### Example 5: Population Problem
```
A town's population decreases by 5% each year.
Current population: 20,000
Population after 2 years?

After 2 years = 20,000 × (0.95)²
              = 20,000 × 0.9025
              = 18,050 people
```

---

## 10. Mental Math Tips 🧠

### Quick Interest Estimation
```
Rule of 72 (for doubling time):
Years to double ≈ 72 ÷ Interest rate

At 8%: 72 ÷ 8 = 9 years to double
At 6%: 72 ÷ 6 = 12 years to double
At 12%: 72 ÷ 12 = 6 years to double
```

### Quick Percentage Profit
```
If you sell at double the cost: 100% profit
If you sell at triple the cost: 200% profit
If you sell at 1.5× the cost: 50% profit
If you sell at 1.25× the cost: 25% profit
```

### Tax Shortcuts
```
For 10% tax: Add 1/10 of the amount
For 5% tax: Add half of 10%
For 7.5% tax: Add 5% + half of 5%

$80 + 10% = $80 + $8 = $88
$80 + 5% = $80 + $4 = $84
```

### Commission Quick Calc
```
6% of sales:
= 5% + 1%
= (sales ÷ 20) + (sales ÷ 100)

6% of $15,000:
= $750 + $150 = $900
```

---

## 📊 Summary Table

### Key Formulas

| Application | Formula |
|-------------|---------|
| Profit | SP - CP |
| Profit % | (Profit/CP) × 100 |
| Loss % | (Loss/CP) × 100 |
| Discount | MP × (Discount%/100) |
| Simple Interest | (P × R × T)/100 |
| Compound Amount | P × (1 + R/100)ⁿ |
| Tax | Price × (Tax%/100) |
| Depreciation | Value × (1 - Rate/100)ⁿ |

### Common Multipliers

| Scenario | Multiplier |
|----------|------------|
| 25% profit | × 1.25 |
| 20% loss | × 0.80 |
| 15% discount | × 0.85 |
| 8% tax added | × 1.08 |
| 10% depreciation | × 0.90 |

---

## ❓ Quick Revision Questions

1. **Profit/Loss**: Cost $120, sold for $96. Find loss percentage.

2. **Discount**: Marked $200, sold at 15% off. Find selling price.

3. **Simple Interest**: $3,000 at 7% for 4 years. Find total interest.

4. **Compound Interest**: $2,000 at 10% compounded annually for 2 years. Find amount.

5. **Tax**: Pre-tax price $85, tax rate 6%. Find total price.

6. **Application**: A car worth $25,000 depreciates 12% per year. Value after 2 years?

<details>
<summary>Click to see answers</summary>

1. Loss = $120 - $96 = $24
   Loss % = (24 ÷ 120) × 100 = **20% loss**

2. SP = $200 × 0.85 = **$170**

3. SI = (3000 × 7 × 4) ÷ 100 = **$840**

4. A = $2,000 × (1.10)² = $2,000 × 1.21 = **$2,420**
   (CI = $420)

5. Total = $85 × 1.06 = **$90.10**

6. Value = $25,000 × (0.88)² = $25,000 × 0.7744 = **$19,360**

</details>

---

## 🔗 Navigation

[← Previous: Percentage Increase and Decrease](03-percentage-increase-decrease.md) | [Back to Contents](../README.md) | [Next: Understanding Ratios →](../07-Ratio-and-Proportion/01-understanding-ratios.md)
