# Research Gap: Condition Filtering on eBay Searches

## Issue

The eBay search methodology used for both `report.md` and `report2.md` did **not** strictly filter out **"for parts or not working"** listings. eBay category/shop pages (`ebay.com/shop/...`, `ebay.com/b/...`) and search-engine-derived results display all conditions intermixed by default. The `LH_ItemCondition` URL parameter was not applied during data collection.

## Impact

Price ranges, especially the low-end floor, may be slightly skewed by non-working listings that sell for less than functional cards. Specific examples discovered during verification:

| Card | "For Parts" Listing Found | Price | Condition |
|------|--------------------------|-------|-----------|
| GTX 1080 Ti 11GB | NVIDIA GeForce GTX 1080 Ti + Titan XP | Unknown | "For parts or not working" |
| GTX 1080 Ti 11GB | MSI GeForce GTX 1080 Ti ARMOR OC | Unknown | "For parts or not working" |
| RTX 3060 12GB | GIGABYTE RTX 3060 Eagle OC | $142.98 | "For parts" |
| RTX 3060 12GB | ZOTAC RTX 3060 12GB | Unknown | "FOR PARTS ONLY" |
| RTX 3060 12GB | MSI RTX 3060 Ventus 3X 12G OC | $148.95 | "For parts or not working" |
| RTX 3090 24GB | GIGABYTE RTX 3090 TURBO 24GB | Unknown | "FOR PARTS OR REPAIR" |
| RTX 3090 24GB | Gigabyte RTX 3090 OC 24GB | Unknown | "For parts or not working" |
| RTX 3090 24GB | HP RTX 3090 24GB OEM | Unknown | "For parts" / "No Core" |

## What Was Verified as Working

The specific anchor prices cited in the reports were drawn from listings explicitly marked as working, pre-owned, or refurbished:

- **GTX 1060 6GB:** $50–$65 pre-owned listings with working condition stated
- **GTX 1080 Ti 11GB:** $150–$165 from EVGA/Dell listings ($159.99–$164.97) marked pre-owned
- **RTX 3060 12GB:** $185 Lenovo OEM sold listing stating *"The item above is fully tested"*; $198.33 Dell sold listing
- **RTX 3090 24GB:** $578.89 ASUS TUF listing marked as working

## Recommended Fix for Future Research

Apply eBay's condition filter via URL parameter:

```
&LH_ItemCondition=3000  # Used / Pre-owned
```

Or manually inspect each sold listing title/description to confirm it is not marked "for parts or not working" before including it in price calculations.

## Affected Sections

- `report.md`: All "Detailed Findings" tier sections
- `report2.md`: All "Detailed Findings" tier sections

## Severity

**Low to Moderate.** The high-volume category/shop pages are dominated by working listings, so the overall price ranges are likely directionally correct. However, the low-end floor of each range may be 5–15% lower than a strictly filtered sample would produce.
