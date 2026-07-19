# Used Vehicle Pricing: A Practical Starting Point for the Pricing Team

This project shows how a used car business could make pricing more consistent without removing human judgement.

The analysis uses **97,673 historical UK used car listings** across **9 manufacturers and 195 vehicle models**.

> This page explains the business recommendation in simple terms. The full data audit and statistical work are documented in [METHODOLOGY_AND_DATA_AUDIT.md](METHODOLOGY_AND_DATA_AUDIT.md).

## The business problem

When a car enters inventory, someone must choose an asking price.

Different employees may reach different answers because they use different comparable cars, different rules and different levels of experience. Straightforward vehicles can take unnecessary time to price, while unusual or expensive vehicles may not receive enough attention.

The business needs a consistent starting point that answers two questions:

1. What advertised price would be normal for a vehicle with these recorded features?
2. Is this a routine vehicle, or should an experienced person review it more carefully?

## The recommendation

**Pilot a transparent advertised-price benchmark in shadow mode.**

The benchmark estimates what vehicles with similar recorded features are usually advertised for. It considers:

* Manufacturer and vehicle model
* Vehicle age
* Mileage
* Engine size
* Fuel type
* Transmission

It does not set the final price. A pricing analyst reviews the evidence and makes the decision.

Think of it as a consistent second opinion. It helps with routine vehicles and tells the team when it does not have enough evidence.

## Why the benchmark is worth testing

The benchmark was tested on **19,433 listings that were not used to build it**. It was compared with a strong method based on similar vehicles, using vehicle model, age and mileage wherever enough comparisons were available.

![Benchmark performance compared with hierarchical comparable prices](benchmark_vs_comparables.png)

| Test result | Comparable price method | Pricing benchmark |
|---|---:|---:|
| Typical percentage error | 9.61% | **7.77%** |
| Average error in pounds | £2,355 | **£1,613** |
| Estimates within 15% of the listed price | 67.65% | **80.04%** |
| Estimates within 20% of the listed price | 78.60% | **90.35%** |

In simple terms, the benchmark gave estimates that were closer to the observed advertised prices than the comparable price method.

The £742 difference in average error is **not money saved**. It only shows that the benchmark estimate was closer to the listed price in this historical dataset.

The benchmark also leaned slightly high. Its median signed percentage error was **+1.03%**, which means estimates tended to sit slightly above observed listed prices. For listings below £10,000, the corresponding figure was **+4.67%**. This is another reason to test the benchmark in shadow mode rather than applying its estimate automatically.

## The most useful result: decide where people should spend their time

The benchmark should not be trusted equally for every car. The analysis therefore separates the portfolio into three groups.

![Share of listings and listed price value by guidance tier](guidance_tiers.png)

The analyst does not choose the group manually. After the vehicle information is entered, the system applies the routing rules and displays the required level of review.

### First, check for manual appraisal

A vehicle goes to manual appraisal if **any** of these conditions apply:

* It is 11 years or older.
* It has at least 100,000 miles.
* Its expected benchmark price is £50,000 or more.
* It is electric, because the dataset contains only six electric records.
* Important information is missing or unclear.
* The benchmark does not have reliable support for the vehicle.

These vehicles should be priced by a specialist. The benchmark is background information only.

### Second, check for cautious review

If no manual appraisal condition applies, a vehicle goes to cautious review if **any** of these conditions apply:

* It is between 6 and 10 years old.
* It has between 60,000 and 99,999 miles.
* The model or comparable group has limited historical support.

The benchmark can still be shown, but a pricing analyst must review the vehicle more carefully.

### Finally, assign standard guidance

A vehicle receives standard guidance only when **all** of these conditions apply:

* It is no more than 5 years old.
* It has fewer than 60,000 miles.
* All required information is available.
* Its fuel type and vehicle model are supported by enough historical examples.
* Its expected benchmark price is below £50,000.
* No cautious review or manual appraisal condition applies.

The system checks historical support automatically. The analyst does not have to decide whether there are enough examples. Strong support means the model has enough historical observations to receive direct guidance rather than relying mainly on a broader fallback group.

If a vehicle meets rules from more than one group, the group requiring more review wins:

> **Manual appraisal takes priority over cautious review. Cautious review takes priority over standard guidance.**

For example:

* A three-year-old car with 120,000 miles goes to manual appraisal because of its mileage.
* A seven-year-old car with 30,000 miles goes to cautious review because of its age.
* A two-year-old car with 20,000 miles and an expected price of £60,000 goes to manual appraisal because of its expected price.
* A four-year-old car with 35,000 miles, complete information and strong model support receives standard guidance.

### How each group is used during shadow mode

| Guidance group | Historical share of listings | How the team should use it during the pilot |
|---|---:|---|
| **Standard guidance** | 79.00% | Show the benchmark beside the analyst’s normal price. Do not change the real asking price. Record whether the analyst agrees. |
| **Cautious review** | 17.56% | Show the benchmark, flag the vehicle for closer review and record the analyst’s decision and reason. |
| **Manual appraisal** | 3.43% | Send the vehicle to a specialist. Use the benchmark only as background information. |

The three displayed shares add to 99.99% because each percentage is rounded separately. The underlying record counts cover all 97,673 listings.

The percentages come from applying these rules to the 97,673 historical listings. A real dealer’s portfolio may have a different mix.

Standard guidance covers **80.32% of the total advertised price amount represented in the dataset**. Manual appraisal covers only **3.43% of listings**, but **5.25% of the total advertised price amount**.

This is not revenue, profit, inventory cost or completed sale value. It is the sum of the advertised prices in each historical group.

The larger share for manual appraisal is deliberate. Every vehicle with an expected benchmark price of at least £50,000 is sent to this group because even a small percentage error can create a large difference in pounds.

This routing system is the main business recommendation. It gives routine cars a consistent reference and concentrates expert attention on the vehicles where mistakes are more likely or more expensive.

## What the pricing team should do for each new vehicle

1. Record the vehicle model, year, mileage, engine size, fuel type and transmission.
2. Produce the benchmark estimate.
3. Produce a separate comparable price estimate.
4. Let the system assign standard guidance, cautious review or manual appraisal.
5. Let the pricing analyst choose the final asking price.
6. If the analyst disagrees with the benchmark, record the size and reason for the difference.

Common reasons for disagreement may include trim, condition, optional equipment, accident history, service history, local demand, margin targets, stock strategy or incorrect data.

The override record is important. It captures business information that is missing from the original dataset and shows where the benchmark needs improvement.

## Three pricing rules supported by the analysis

These rules are not new discoveries for an experienced pricing manager. Their value is that the analysis measures them and builds them into a consistent process.

### Rule 1: compare similar vehicles, not broad averages

The raw median diesel listing is about **£5,000 more expensive** than the raw median petrol listing. This could easily be mistaken for a diesel premium.

When the analysis accounts for vehicle model, age, mileage, engine size and transmission, diesel is instead associated with a **6.59% lower advertised price** than petrol.

![Raw fuel medians compared with like-for-like fuel relationships](diesel_reversal.png)

The raw difference is mainly caused by the mix of vehicles. Diesel listings include more large and expensive models.

**Practical rule:** do not create pricing adjustments from broad averages. Compare vehicles with similar recorded features.

The 6.59% result should not be used as a fixed diesel discount in a live business. It describes this historical dataset and would need to be checked again using current company data.

### Rule 2: consider age and mileage together

A simple rule such as “subtract the same amount for every year and every 10,000 miles” is too crude. The price difference changes as cars become older and accumulate more mileage.

![Supported age and mileage advertised price index](age_mileage_price_index.png)

The reference is a one-year-old vehicle with 10,000 miles and has an index of 100. A five-year-old vehicle with 60,000 miles has an index of about 54 after the other recorded features are held constant.

Blank cells have too few similar vehicles to support a standalone answer. Dashed cells have limited evidence and should be treated cautiously.

**Practical rule:** use the combined age and mileage estimate for common vehicle combinations. Send unusual combinations for additional review.

### Rule 3: start with the vehicle model, not only the brand

Brand-level differences exist, but they are too broad for pricing an individual car. Mercedes-Benz has an adjusted index of about 132 against a portfolio reference of 100, while Vauxhall has an index of about 74. Individual vehicle models vary much more widely.

![Manufacturer and selected model conditional price indices](model_vs_brand_positioning.png)

An index of 132 means the adjusted advertised price is about 32% above the portfolio reference. It is not a markup to add to a car.

**Practical rule:** start with the manufacturer and exact vehicle model. Use the wider brand only when there are too few useful comparisons for the model.

## The price range is a warning, not a menu

For routine vehicles, the benchmark can show a range based on its historical errors.

For example, when the central estimate is £15,000:

* The standard 90% historical range is approximately **£12,300 to £18,200**.
* The wider 95% review range is approximately **£11,800 to £18,900**.

The team should not select any convenient price inside this range. The range only shows how uncertain the benchmark has been in the past. A wider range means more judgement is required.

Manual appraisal vehicles should not receive a calibrated benchmark range.

## What this project cannot answer

The source contains advertised prices, not completed sales. It does not include enough information to answer several important commercial questions.

This project cannot tell the business:

* What price the customer will finally pay
* Whether a vehicle will sell quickly
* Which price will produce the highest profit
* Whether a benchmark gap proves overpricing or underpricing
* How condition, trim, optional equipment or accident history changes the price
* Whether the same relationships still hold in the current market

The data are historical and vehicle age assumes 2020 as the reference year. The results should not be used directly for live pricing today.

## A practical shadow pilot

The business should begin with a shadow pilot. During this period, the benchmark produces information but does not change any real asking price.

### Pricing manager

* Own the pilot and approve any movement toward live use.
* Review whether the three guidance groups remain sensible.
* Investigate segments where analysts repeatedly disagree with the benchmark in the same direction.

### Pricing analysts

* Continue pricing vehicles normally.
* Record whether they agree with the benchmark.
* Record the final chosen price and any reason for overriding the benchmark.

### Operations team

* Make sure required vehicle information is complete and entered consistently.
* Track missing fields and data corrections.

### Data and analytics team

* Monitor error, missing information, routing and override patterns.
* Report results separately for vehicle model, age, mileage and price level.
* Correct persistent problems or move weak segments to a higher review group.

## When the pilot can move forward

The benchmark should move from shadow mode to live advisory use only when:

1. Every vehicle is routed to the correct guidance group.
2. The required vehicle information is consistently available.
3. Analysts understand the output and the uncertainty range.
4. Override reasons are recorded and can be explained.
5. No important group shows a repeated pricing difference in one direction without investigation.

Even then, live use should begin with standard guidance vehicles only. Cautious vehicles should continue to require review, and manual appraisal vehicles should remain with specialists.

These conditions show that the process is stable. They do not prove that the benchmark improves sales or profit.

## Data the business should collect next

To test commercial value, the business would need:

1. Final sale price
2. Sale status
3. Listing date and sale date
4. Price changes
5. Vehicle condition, trim and optional equipment
6. Service and accident history
7. Acquisition and preparation costs
8. Customer enquiries and leads

With this information, the business could test whether benchmark-supported pricing improves sale speed, margin or another agreed commercial goal.

## Bottom line

This project does not discover a new rule of used car pricing. Its useful contribution is a clearer pricing process.

It provides a more consistent starting estimate for routine vehicles, directs experienced people toward the difficult vehicles and creates a record of why human judgement differs from the data.

The realistic recommendation is:

> **Use the benchmark as a second opinion in a controlled shadow pilot. Do not use it as an automatic pricing system or as proof that a vehicle is mispriced.**

## Repository contents

| File | Purpose |
|---|---|
| `README.md` | Business recommendation and operating plan |
| `METHODOLOGY_AND_DATA_AUDIT.md` | Data provenance, cleaning, statistical design and validation |
| `images/` | Final chart PNGs |

## Data source

[Kaggle: 100,000 UK Used Car Data Set](https://www.kaggle.com/datasets/adityadesai13/used-car-dataset-ford-and-mercedes). The source describes used car listings collected from the British market through web scraping.

This is an independent portfolio case study. It is not affiliated with or endorsed by Kaggle, a dealer, marketplace, lender or insurer. The source data are not included in this repository; follow the dataset owner’s licensing and access terms.
