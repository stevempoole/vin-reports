# Improved Pricing Subagent Instructions

When researching market pricing, follow these enhanced guidelines:

## Mileage Matching Requirements:

1. **Primary search**: Find vehicles within ±30,000 miles of the target vehicle's mileage
2. **Secondary search**: If <5 listings found, expand to ±50,000 miles but note the variance
3. **Mileage adjustment**: For listings outside the range, adjust pricing using:
   - High-mileage vehicles (200k+): -$0.05 per mile difference
   - Mid-mileage vehicles (100k-200k): -$0.08 per mile difference  
   - Low-mileage vehicles (<100k): -$0.12 per mile difference

## Enhanced Fair Price Calculation:

1. **Filter listings** by mileage proximity first
2. **Weight recent listings** more heavily (last 30 days)
3. **Account for regional differences** - note if sample is geographically skewed
4. **Document confidence level**:
   - High: 8+ listings within ±30k miles, multiple regions
   - Medium: 5-7 listings within ±50k miles  
   - Low: <5 listings or wide mileage variance

## Red Flags Documentation:

Document specific reasons why below-market listings exist:
- Salvage/flood title
- No maintenance records
- Mechanical issues noted
- Seller motivation (urgent sale, estate, etc.)

This ensures fair pricing reflects actual market conditions for vehicles with similar wear and age.