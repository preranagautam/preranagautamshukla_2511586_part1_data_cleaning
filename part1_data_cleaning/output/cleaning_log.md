# Cleaning Log — Part 1: Business Data Cleaning

## 1. Issues Found
- 20 exact duplicate rows
- 12 conflicting duplicate order IDs
- 26 missing region values
- 22 missing ship_mode values
- 18 missing discount values
- Text casing inconsistencies in 10 columns
- Negative discounts in 16 rows
- High discounts (>50%) in 15 rows
- 6 different date formats mixed in date columns
- 21 date-related issues (invalid or ship before order)
- 65 sales calculation mismatches

## 2. Cleaning Actions Performed
- Removed 20 exact duplicate rows
- Applied TRIM + PROPER to all text columns
- Standardized all date formats to YYYY-MM-DD
- Filled missing region with "Unknown"
- Filled missing ship_mode with "Unknown"
- Set 18 missing discounts to 0 (sales fields valid)
- Converted "70%"/"85%" string discounts to 0.70/0.85

## 3. Business Rules Applied
| Rule | Action |
|------|--------|
| Missing region | Filled "Unknown" |
| Missing ship_mode | Filled "Unknown" |
| Missing discount | Set to 0 if other fields valid |
| Negative discount | Flagged Invalid |
| Discount > 50% | Flagged Warning |
| Cancelled orders | Excluded from sales summaries |
| Failed payments | Excluded from sales summaries |
| Refunded orders | Summarized separately |
| Ship date before order date | Flagged Invalid |

## 4. Assumptions Made
- Discount is a fraction (0.0 to 1.0), not a percentage
- >50% discount is "unusually high" (threshold assumed)
- Ambiguous dates parsed as MM/DD/YYYY when unclear
- Conflicting duplicates kept, not deleted

## 5. Records Removed
- 20 exact duplicate rows removed

## 6. Records Flagged
- 772 Clean, 89 Warning, 51 Invalid

## 7. Limitations
- Some date formats were ambiguous (MM/DD vs DD/MM)
- Conflicting duplicate IDs need business team review
- "Unknown" fills are placeholders, not real data