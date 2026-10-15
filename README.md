## BrightCart: Profitability and Marketing Efficiency Analysis

## 1. What Questions Did This SQL Analysis Answer?

BrightCart wanted to understand which parts of its business generated profit after recorded order costs and where marketing spending could be reviewed.

## My SQL analysis addressed five questions:

## Category profitability
Which product categories had the highest and lowest profit margins?
How did product costs, shipping, discounts, and refunds differ across categories?
Which catalog products had selling prices below their listed product and shipping costs?

## Sales channel performance
Which channel generated the highest and lowest profit per order?
How did average order value, profit margin, and return rate vary by channel?
How much did platform fees contribute to Marketplace and Social Commerce costs?

## Returns and revenue loss
What was the return rate by category and sales channel?
Which category-channel combinations accounted for refunded revenue?
How much total revenue was refunded during the analysis period?

## Marketing efficiency
Which platforms delivered the strongest reported return on advertising spend?
How did cost per conversion, cost per click, click-through rate, and conversion rate compare?
Were any platform-months generating less attributed revenue than advertising spend?

## Marketing budget prioritization
What would a 20% reduction in the historical marketing budget equal?
Which platform-months should be reviewed first based on reported ROAS?
How could ranking and cumulative spending support budget-reduction decisions?

[SQLQuery1-BrightCart.sql](https://github.com/user-attachments/files/33505144/SQLQuery1-BrightCart.sql)

## 2. How Did I Approach the Analysis?

Step 1: Verified the imported datasets

I began by checking record counts and date ranges to understand the size and coverage of each table.

## Dataset	Records	Coverage
Orders	2,000	January 28, 2024–January 11, 2026
Products	207	Product catalog across eight categories
Marketing spend	144	January 2024–December 2025

This script focuses on validation and analysis rather than table creation or data loading.

Step 2: Checked data quality and financial consistency

Before calculating business metrics, I checked for:

1. Duplicate order IDs
2. Duplicate product IDs
3. Duplicate marketing platform-month combinations
4. Missing values in selected critical order fields
5. Inconsistencies between recorded and calculated financial amounts
6. Mismatches between return status and refund records

I validated three accounting relationships:

**Total Costs =
Product Cost + Shipping Cost + Platform Fee + Transaction Fee

**Net Revenue =
Gross Revenue − Discounts − Refunds

**Profit =
Net Revenue − Total Costs
I used a tolerance of $0.02 when checking financial differences.

Step 3: Created a reusable order analysis view

I created vw_order_analysis to standardize the order data used throughout the analysis.

The view:

1. Renamed primary_category to category
2. Retained the revenue and cost components
3. Converted return status into a numeric return flag
4. Calculated order-level profit margin
5. Returned NULL when an order’s net revenue was zero
This reduced repeated logic and provided a consistent foundation for subsequent queries.

Step 4: Calculated category profitability and cost drivers

I grouped orders by category and calculated:

Gross and net revenue
Discounts and refunds
Product, shipping, platform, and transaction costs
Total profit
Profit per order
Profit margin
Return rate
I calculated aggregate profit margin as:

sql;

**SUM(profit) / NULLIF(SUM(net_revenue), 0)**

This measures the category’s overall margin rather than giving every order equal weight through an average of individual order margins.

I also compared shipping and product costs as percentages of net revenue, and discounts and refunds as percentages of gross revenue.

Step 5: Analyzed product catalog economics separately

The order table did not contain a product ID, so I did not join orders to products using category alone. Doing so would have created multiple matches per order and inflated financial totals.

Instead, I analyzed catalog contribution separately:

**Unit Contribution =
Selling Price − Unit Cost − Shipping Cost per Unit
I identified products with negative contribution and summarized their distribution by category.

Step 6: Compared channel performance

I grouped orders by sales channel and compared:

Gross average order value
Profit per order
Profit margin
Total platform fees
Platform fees per order
Return rate
Refunded revenue

Platform fees were already included in recorded total costs and profit, so I did not subtract them again.

Step 7: Measured returns and refunded revenue

I calculated return rates by category, channel, and category-channel combination.

I used:

**Return Rate = Returned Orders / Total Orders
To measure revenue reversed through returns, I summed refund_amount rather than the original value of returned orders.

Step 8: Recalculated marketing performance
I calculated platform-level metrics from aggregated totals:

ROAS = Attributed Revenue / Marketing Spend
CPA = Marketing Spend / Recorded Conversions
CPC = Marketing Spend / Clicks
CTR = Clicks / Impressions
Conversion Rate = Conversions / Clicks
This avoided simply averaging monthly ratios.

I then examined individual platform-months to identify weak periods that could be hidden by strong overall platform performance.

Step 9: Built a budget-reduction prioritization framework
I calculated the value of a 20% budget reduction, then used:

CTEs to structure the analysis
RANK() to prioritize platform-months from lowest to highest ROAS
SUM() OVER() to calculate cumulative spending in that order
This provided a transparent starting point for reviewing spending.

The SQL did not calculate a finalized allocation, a partial cut in the final selected period, or the expected revenue impact of reductions.

## 3. What Was the Outcome?
Overall business performance

Metric	Result
Total orders	2,000
Distinct customers	734
Gross revenue	$277,969.13
Net revenue	$236,318.37
Recorded order costs	$179,984.54
Recorded order profit	$56,333.83
Aggregate profit margin	23.84%
The implemented data-quality checks found no duplicate keys, no missing values in the tested order fields, and no accounting-formula differences above the specified tolerance.

## Finding 1: Electronics led profitability; Books had the weakest margin
Category	Profit	Profit margin
Electronics	$13,973.46	31.13%
Toys	$8,786.25	26.15%
Home & Kitchen	$6,688.57	25.37%
Food & Beverage	$7,591.77	24.76%
Sports	$7,596.63	23.50%
Clothing	$6,272.48	19.99%
Beauty	$3,174.88	17.39%
Books	$2,249.79	11.94%

Shipping was the clearest observed difference between Electronics and Books:

Books shipping costs equaled 33.23% of net revenue
Electronics shipping costs equaled 15.91%
Books refunds equaled 9.40% of gross revenue, compared with 7.71% for Electronics
The catalog analysis also identified 41 negative-contribution SKUs, including 14 Books products.

## Business implication: Books should be prioritized for shipping-policy, bundling, pricing, and sourcing review. However, the missing order-to-product relationship prevents estimating realized losses for individual SKUs.

## Finding 2: Owned channels generated more profit per order
Channel	Profit per order	Profit margin	Platform fee per order
Mobile App	$36.32	29.76%	$0.00
Website	$31.60	27.01%	$0.00
Social Commerce	$17.11	15.37%	$9.87
Marketplace	$15.40	13.03%	$18.97
Mobile App generated approximately $20.92 more recorded profit per order than Marketplace.

Marketplace incurred $7,949.69 in platform fees, contributing to its weaker performance.

## Business implication: Review Marketplace fees and channel-specific economics while investigating retention opportunities on owned channels. The observed profit gap is not a guaranteed gain from moving customers because product mix and customer behavior may differ.

## Finding 3: Returns reversed more than $20,000 in revenue
144 orders were returned
Overall return rate was 7.20%
Refunded revenue totaled $20,582.45
Refunds equaled approximately 7.40% of gross revenue
Electronics had the highest category return rate at 8.61%
Social Commerce had the highest channel return rate at 9.14%

## Business implication: Investigate return reasons in higher-return segments. Return frequency and refunded dollars should both be monitored because they measure different aspects of the problem.

## Finding 4: TikTok led reported ROAS, while Email Marketing required closer review
Platform	ROAS	Cost per conversion	Cost per click
TikTok Ads	24.02x	$3.06	$0.15
Influencer	22.70x	$3.39	$0.17
Instagram Ads	15.73x	$4.70	$0.21
Google Ads	14.38x	$4.72	$0.25
Facebook Ads	11.45x	$6.27	$0.32
Email Marketing	4.81x	$17.85	$0.94
All platforms had aggregate ROAS above 1.0x.

However, Email Marketing in December 2025 generated:

Spend: $1,796.66
Attributed revenue: $1,198.99
ROAS: approximately 0.67x
Attributed revenue fell below advertising spend by $597.67, before accounting for any other costs.

## Business implication: Review weak platform-months rather than assuming that an entire platform performs consistently.

## Finding 5: A 20% marketing reduction equals $100,701.23
Budget measure	Amount
Historical marketing spend	$503,506.14
20% reduction target	$100,701.23
Budget after reduction	$402,804.91
The ranking identified Email Marketing in December 2025 as the first period to review, followed by other low-ROAS observations.

## Business implication: Use the ranked and cumulative-spend outputs to guide a future budget review, rather than applying equal percentage cuts across platforms.

Historical spending cannot be recovered, and the analysis does not establish how future cuts would affect incremental revenue.

**Important interpretation boundaries**
The supplied orders contain $277,969.13 in gross revenue, not the $1M+ stated in the business brief.
Marketing-attributed revenue totals $8,146,615.20, which does not reconcile with order revenue. The marketing and order results must therefore be interpreted separately.
Recorded order profit excludes marketing expense, overhead, taxes, and other unrecorded costs. It is not fully loaded company net profit.
ROAS above 1.0x does not establish profitability after product and fulfillment costs.
The script does not analyze monthly margin trends, so it does not verify the brief’s claim that margins were shrinking.
Recommendations describe actions supported by the findings; they are not implemented improvements or proven financial outcomes.

## 4. What Tools and Skills Did I Use?
Tools | Microsoft SQL Server — querying and analyzing the imported datasets
T-SQL — writing validation, financial analysis, and budget-prioritization queries
SQL skills

## Skill	Application
Aggregate functions	Calculated revenue, costs, profit, spending, clicks, and conversions
GROUP BY	Compared categories, channels, products, and marketing platforms
HAVING	Identified duplicate business keys
CASE expressions	Created return flags and conditional counts
ABS()	Checked differences between recorded and recalculated financial values
NULLIF()	Protected ratio calculations from zero denominators
Views	Created a reusable order-analysis layer
CTEs	Organized marketing-efficiency and cumulative-spend calculations
RANK()	Prioritized platform-months by reported ROAS
SUM() OVER()	Calculated cumulative spending for budget review
Sorting and filtering	Identified top performers, weak periods, and negative-contribution products

**Analytical and business skills**
Data profiling and targeted quality assurance
Financial reconciliation
KPI definition and weighted-ratio calculations
Category and channel profitability analysis
Product contribution analysis
Return-rate and refund analysis
Marketing efficiency evaluation
Budget prioritization
Recognition of missing join keys and incompatible data grain
Clear separation of verified findings, assumptions, and limitations

## Project takeaway
This project demonstrates how I use SQL to move beyond revenue reporting and investigate the factors associated with profitability. I translated transaction and marketing data into actionable areas for review while clearly identifying what the available data could—and could not—support.



