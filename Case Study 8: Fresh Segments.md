# Case Study #8: Fresh Segments


## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-8/). 

***

## Business Task

Fresh Segments is a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

***

## Entity Relationship Diagram

**Table: `interest_map`**
<kbd><img width="970" alt="image" src="https://user-images.githubusercontent.com/81607668/138912360-49996d84-23e4-40a7-98a1-9a8e9341b492.png"></kbd>

| id | interest_name             | interest_summary                                                                   | created_at              | last_modified           |
|----|---------------------------|------------------------------------------------------------------------------------|-------------------------|-------------------------|
| 1  | Fitness Enthusiasts       | Consumers using fitness tracking apps and websites.                                | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 2  | Gamers                    | Consumers researching game reviews and cheat codes.                                | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 3  | Car Enthusiasts           | Readers of automotive news and car reviews.                                        | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 4  | Luxury Retail Researchers | Consumers researching luxury product reviews and gift ideas.                       | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 5  | Brides & Wedding Planners | People researching wedding ideas and vendors.                                      | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 6  | Vacation Planners         | Consumers reading reviews of vacation destinations and accommodations.             | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:13.000 |
| 7  | Motorcycle Enthusiasts    | Readers of motorcycle news and reviews.                                            | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:13.000 |
| 8  | Business News Readers     | Readers of online business news content.                                           | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |
| 12 | Thrift Store Shoppers     | Consumers shopping online for clothing at thrift stores and researching locations. | 2016-05-26T14:57:59.000 | 2018-03-16T13:14:00.000 |
| 13 | Advertising Professionals | People who read advertising industry news.                                         | 2016-05-26T14:57:59.000 | 2018-05-23T11:30:12.000 |

**Table: `interest_metrics`**

| month | year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
|-------|------|------------|-------------|-------------|-------------|---------|--------------------|
| 7     | 2018 | Jul-18     | 32486       | 11.89       | 6.19        | 1       | 99.86              |
| 7     | 2018 | Jul-18     | 6106        | 9.93        | 5.31        | 2       | 99.73              |
| 7     | 2018 | Jul-18     | 18923       | 10.85       | 5.29        | 3       | 99.59              |
| 7     | 2018 | Jul-18     | 6344        | 10.32       | 5.1         | 4       | 99.45              |
| 7     | 2018 | Jul-18     | 100         | 10.77       | 5.04        | 5       | 99.31              |
| 7     | 2018 | Jul-18     | 69          | 10.82       | 5.03        | 6       | 99.18              |
| 7     | 2018 | Jul-18     | 79          | 11.21       | 4.97        | 7       | 99.04              |
| 7     | 2018 | Jul-18     | 6111        | 10.71       | 4.83        | 8       | 98.9               |
| 7     | 2018 | Jul-18     | 6214        | 9.71        | 4.83        | 8       | 98.9               |
| 7     | 2018 | Jul-18     | 19422       | 10.11       | 4.81        | 10      | 98.63              |

***

## Question and Solution

Please join me in executing the queries using PostgreSQL on [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17). It would be great to work together on the questions!

If you have any questions, reach out to me on [LinkedIn](https://www.linkedin.com/in/katiehuangx/).

## 🧼 A. Data Exploration and Cleansing

**1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a `date` data type with the start of the month**

```sql
ALTER TABLE fresh_segments.interest_metrics
ALTER month_year TYPE DATE USING month_year::DATE;
```

<kbd><img width="970" alt="image" src="https://github.com/user-attachments/assets/ecce0d9f-078f-4dc0-909d-f6b318435c29"></kbd>

***
  
**2. What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the `null` values appearing first?**

```sql
SELECT 
  month_year, COUNT(*)
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```
<kbd><img width="291" alt="image" src="https://github.com/user-attachments/assets/5f5c5a14-a3b6-4cba-9711-2901da22a7d9"></kbd>

**3. What do you think we should do with these `null` values in the `fresh_segments.interest_metrics`?**

The `null` values appear in `_month`, `_year`, `month_year`, and `interest_id`. The corresponding values in `composition`, `index_value`, `ranking`, and `percentile_ranking` fields are not meaningful without the specific information on `interest_id` and dates. 

Before dropping the values, it would be useful to find out the percentage of `null` values.

```sql
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 /
    COUNT(*)),2) AS null_perc
FROM fresh_segments.interest_metrics
```

<kbd><img width="100" alt="image" src="https://user-images.githubusercontent.com/81607668/138892507-5b89eba8-45c7-4edf-9c05-42347f47c746.png"></kbd>

The percentage of null values is 8.36% which is less than 10%, hence I would suggest to drop all the `null` values.

```sql
DELETE FROM fresh_segments.interest_metrics
WHERE interest_id IS NULL;

-- Run again to confirm that there are no null values.
SELECT 
  ROUND(100 * (SUM(CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 /
    COUNT(*)),2) AS null_perc
FROM fresh_segments.interest_metrics
```

<kbd><img width="120" alt="image" src="https://github.com/user-attachments/assets/e9e4956e-c49e-40a9-9744-702b26e2da30"></kbd>

Confirmed that there are no `null` values in `fresh_segments.interest_metrics`.

***
  
**4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?**.

Before we perform the actual Query for the solution we need to convert the interest_id to INTEGER DATA TYPE:

```sql
ALTER TABLE interest_metrics
ALTER COLUMN interest_id TYPE INTEGER
USING CAST(interest_id AS INTEGER);
```
 The Final Query :

```sql
SELECT 
  COUNT(DISTINCT map.id) AS map_id_count,
  COUNT(DISTINCT metrics.interest_id) AS metrics_id_count,
  SUM(CASE WHEN map.id is NULL THEN 1 END) AS not_in_metric,
  SUM(CASE WHEN metrics.interest_id is NULL THEN 1 END) AS not_in_map
FROM fresh_segments.interest_map map
FULL OUTER JOIN fresh_segments.interest_metrics metrics
  ON metrics.interest_id = map.id;
```

<kbd><img width="617" alt="image" src="https://github.com/user-attachments/assets/25311075-b607-4b77-975c-fd4311194f3c"></kbd>

- There are 1,209 unique `id`s in `interest_map`.
- There are 1,202 unique `interest_id`s in `interest_metrics`.
- There are no `interest_id` that did not appear in `interest_map`. All 1,202 ids were present in the `interest_metrics` table.
- There are 7 `id`s that did not appear in `interest_metrics`.
```sql
SELECT id AS Ids_not_in_metrics
FROM interest_map
WHERE id NOT IN (SELECT interest_id FROM interest_metrics);
```
<kbd><img width="120" alt="image" src="https://github.com/user-attachments/assets/d75793b6-c1e2-4055-9501-24a9092ced05"></kbd>
***
  
**5. Summarise the id values in the `fresh_segments.interest_map` by its total record count in this table.**

```sql
SELECT
  m.id,
  m.interest_name,
  COUNT(*) AS total_associations
FROM interest_map AS m
INNER JOIN interest_metrics AS me ON m.id = me.interest_id
GROUP BY m.id, m.interest_name
ORDER BY total_associations DESC;
```

|id|interest_name|total_associations|
|------|------------------------------|----|
19630|Toyota Vehicle Shopper|14|
171|Shoe Shoppers|14|
5929|Employee Management & HR Researchers|14|
10008|Japanese Luxury Car Enthusiasts|14|
19592|Cigars Shoppers|14|
6107|Ski House Second Home Owners|14|
6340|Kitchen and Home Goods Shoppers|14|
6171|High-End Camera Shoppers|14|
89|Travel Researchers|14|
5968|Luxury Kitchen Goods Shoppers|14|
5910|SEO Specialists|14|
73|Mailing & Shipping Shoppers|14|
6211|Charitable Organization Researchers|14|
72|Nurses|14|
4857|Business Accounting Software Researchers|14|
12133|Luxury Boutique Hotel Researchers|14|
19611|Jeep Vehicle Shoppers|14|
22419|Summer Festivals and Fairs Visitors|14|
6169|Videographers|14|
12027|Gen Z Apparel Shoppers|14|
4908|Back Pain Sufferers|14|
6302|Los Angeles Trip Planners|14|
5970|Luxury Home Fixture Shoppers|14|
20767|Beer Lovers|14|
11974|Price Conscious Home Shoppers|14|
137|Online Alcohol Shoppers|14|
10351|Military Personal Finance Researchers|14|
22421|Online Card Games Players|14|
20755|Movie Ticket Buyers|14|
19617|Nissan Vehicle Shoppers|14|
6345|Senior Caregivers|14|
18619|Asian Food Enthusiasts|14|
64|High-End Kids Furniture and Clothes Shoppers|14|
22403|Pandora Jewelry Shoppers|14|
19520|Building Service Contractors|14|
98|Restaurant Researchers|14|
132|Apple Fans|14|
18347|Affordable Hotel Bookers|14|
5961|Business Tax Filers|14|
6229|Sweet Tooths|14|
109|Joint Pain Sufferers|14|
6215|Roommate Seekers|14|
5957|Online Education Intenders|14|
6189|Texas Energy Providers|14|
70|Drug Store Shoppers|14|
7537|Ecommerce Platform Researchers|14|
31|Home Decor Shoppers|14|
4925|Gun Control Advocates|14|
6077|Horse Race Enthusiasts|14|
19623|Russia Trip Planners|14|
5938|Costa Rica Trip Planners|14|
10251|Personal Bank Comparison Researchers|14|
6095|Nursing Students|14|
160|Sightseeing Travelers|14|
113|Avid Readers|14|
19298|Beach Supplies Shoppers|14|
20|Moviegoers|14|
6318|Phoenix Trip Planners|14|
6323|New Orleans Trip Planners|14|
6343|Patio Furniture Shoppers|14|
43|Apartment Hunters|14|
6393|Eye Health Researchers|14|
4|Luxury Retail Researchers|14|
10250|Halloween Costume Shoppers|14|
126|Soccer Fans|14|
12|Thrift Store Shoppers|14|
10976|Christmas Decorations Shoppers|14|
6045|Engagement Ring Shoppers|14|
20814|Boston Celtics Fans|14|
20768|Beer Aficionados|14|
34|Teen Girl Clothing Shoppers|14|
19294|Patriotic Party Decoration Shoppers|14|
117|Boxing Fans|14|
4896|Streaming Music Enthusiasts|14|
18783|Nursing and Physicians Assistant Journal Researchers|14|
21241|Premier League Fans|14|
4928|Immigration Control Advocates|14|
23778|Venture Capitalists|14|
6264|Gastroenterologists|14|
13421|Los Angeles Mass Transit Commuters|14|
4937|Evangelical Christians|14|
100|Nutrition Conscious Eaters|14|
162|Retirees|14|
5907|Budget Small Business Researchers|14|
19622|Quads and ATV Enthusiasts|14|
5917|Caribbean Trip Planners|14|
21240|Italian Mens National Soccer Team Fans|14|
12026|California Trip Planners|14|
10834|Norfolk and Virginia Beach Trip Planners|14|
6320|Miami Trip Planners|14|
10988|Santa Monica Trip Planners|14|
46|Job Seekers|14|
7541|Alabama Trip Planners|14|
50|Discount Big Box Shoppers|14|
153|Phone Accessory Shoppers|14|
6315|Boston Trip Planners|14|
22096|Camping Enthusiasts|14|
13496|Professional Chefs|14|
6183|Accounting & CPA Continuing Education Researchers|14|
4903|Asthma Sufferers|14|
6234|Yachting Enthusiasts|14|
83|Hardware Shoppers|14|
5991|Gluten-Free Recipe Researchers|14|
7685|Home Healthcare Researchers|14|
6219|Cycling Enthusiasts|14|
19423|Kitchen Appliance Shoppers|14|
10833|Dallas Trip Planners|14|
39|Furniture Shoppers|14|
32703|School Supply Shoppers|14|
19631|Tractor Shoppers|14|
10249|Loan Comparison Researchers|14|
10009|Investment Services Intenders|14|
60|Online Health Researchers|14|
19519|Property and Facility Managers|14|
143|Health Insurance Researchers|14|
108|Pain Medication Users|14|
6030|Spa and Pool Owners|14|
10975|Thanksgiving Meal Planners|14|
6108|Beach House Second Home Owners|14|
77|Luxury Retail Shoppers|14|
19615|Mercedes-Benz Vehicle Shoppers|14|
14895|In-Market Car Shoppers|14|
6142|European Trip Planners|14|
27|Auto Insurance Shoppers|14|
22427|Yale University Fans|14|
7545|Florida Atlantic Coast Trip Planners|14|
20730|Readers of Chinese Content|14|
5913|Spa Goers|14|
87|Theme Park Researchers|14|
29|College Students|14|
6051|Readers of Italian Content|14|
101|Flower & Gift Basket Shoppers|14|
6314|Online Directory Searchers|14|
25|Doctors|14|
99|Order-in Eaters|14|
22428|Harvard University Fans|14|
134|Home Printer Shoppers|14|
6067|Coffee Lovers|14|
22095|Hiking Enthusiasts|14|
5911|Credit Score Researchers|14|
4924|Surfers|14|
10837|Denver Trip Planners|14|
6304|New York Trip Planners|14|
157|Car Rental Shoppers|14|
4921|Tire Shoppers|14|
84|Reggaeton Fans|14|
6110|Apartment Furniture Shoppers|14|
6325|Orlando Trip Planners|14|
4859|Business Logistics Managers|14|
79|Luxury Travel Researchers|14|
19625|Sony Fans|14|
4909|First Aid Researchers|14|
33191|Online Shoppers|14|
16119|San Diego Trip Planners|14|
11066|Chain Pizzeria Fans|14|
6378|Luggage Shoppers|14|
32486|Vacation Rental Accommodation Researchers|14|
6267|Pediatricians|14|
6286|Luxury Hotel Guests|14|
6133|Fantasy Football Enthusiasts|14|
44|Home Buyers|14|
5944|Financial Advisors|14|
19610|Jaguar Vehicle Shoppers|14|
6081|Hawaii Trip Planners|14|
6112|Olympic Sports Enthusiasts|14|
7529|Foreman and Construction Managers|14|
22417|Country Music Festivals Enthusiasts|14|
7540|North Carolina Travel Researchers|14|
7544|Texas Trip Planners|14|
16196|Mexico Trip Planners|14|
5958|Flooring Shoppers|14|
6223|Language Learners|14|
19596|Dirt Bike Enthusiasts|14|
4912|Vitamin Shoppers|14|
18204|Healthcare Thought Leaders|14|
4935|Conservative Think Tank Readers|14|
6385|Motorcycle Purchasers|14|
4898|Cosmetics and Beauty Shoppers|14|
4897|Parents of Teenagers Going to College|14|
111|Retirement Planners|14|
141|Skin Care Researchers|14|
6389|Oil Change Customers|14|
94|Day Care Users|14|
6322|Seattle Trip Planners|14|
5963|Weight Lifting Enthusiasts|14|
10970|Halloween Decorations Shoppers|14|
5972|Financial Investors|14|
19613|Land Rover Shoppers|14|
19295|Fishing Equipment Shoppers|14|
95|Stay-at-Home Parents|14|
6387|Wireless Service Provider Researchers|14|
4929|Right Wing Radicals|14|
10284|Alaskan Cruise Planners|14|
11067|NCAA Football Fans|14|
7557|Tailgaters|14|
4926|Gun Rights Advocates|14|
6336|Mid-Range Grocery Shoppers|14|
5934|Logistics Professionals|14|
21293|Home Wifi Researchers|14|
10832|Austin Trip Planners|14|
4910|Cholesterol Researchers|14|
19338|Summer Activities Researchers|14|
10980|Hanukkah Decorations Shoppers|14|
5926|Readers of Data Scientist blogs|14|
18782|Internal Medicine Journal Researchers|14|
18995|Commercial Architects and Designers|14|
5916|Shared Work Space Researchers|14|
16|NCAA Fans|14|
18996|Investment Banking Professionals|14|
22092|Water Park Visitors|14|
22423|Martial Arts Enthusiasts|14|
17729|Competitive Car Brand Shoppers|14|
6366|Rideshare App Users|14|
41|Intimates Shoppers|14|
19757|Natural Pet Food Shoppers|14|
18807|Food Safety QA Professionals|14|
28|Teachers|14|
12132|Last Minute Travelers|14|
6168|Christmas Enthusiasts|14|
12821|Commercial Healthcare Hospitality & Retail Designers|14|
19579|Hematologists|14|
15|NBA Fans|14|
18998|Investment Management Professionals|14|
6317|Houston Trip Planners|14|
4931|Marijuana Legalization Advocates|14|
22400|Oil Brand Shoppers|14|
6218|Running Enthusiasts|14|
6206|Preppy Clothing Shoppers|14|
21239|FC Barcelona Fans|14|
6321|Nashville Trip Planners|14|
6210|College Aspirants|14|
96|Exercise and Gym Researchers|14|
6316|Philadelphia Trip Planners|14|
10974|Thanksgiving Entertaining Researchers|14|
6255|Environmental Activists|14|
80|Democrats|14|
37|Jewelry & Watch Shoppers|14|
18622|Las Vegas Nightlife Researchers|14|
135|Major Appliance Shoppers|14|
19588|Audi Vehicle Shoppers|14|
5925|Business Incorporating & LLC Filing Researchers|14|
110|Pain Management Researchers|14|
6062|Italian Chain Restaurant Eaters|14|
6023|Sports Medicine Health Care Professionals|14|
158|Flight and Hotel Shoppers|14|
6237|Non-Profit Volunteers|14|
7483|Economy Grocery Shoppers|14|
5914|Career Focused Individuals|14|
6335|Outdoors Enthusiasts|14|
6085|Discount Womens Shoes Shoppers|14|
7454|Farmers|14|
6284|Gym Equipment Owners|14|
55|Seasonal Allergy Sufferers|14|
6024|Joint Pain Health Care Professionals|14|
17672|Indie Coffee Shoppers|14|
7461|Food Industry Professionals|14|
6226|Divorcees|14|
5|Brides & Wedding Planners|14|
6324|Las Vegas Trip Planners|14|
6216|Car Buyers|14|
17785|Luxury Womens Brands Shoppers|14|
5906|Business Communication Product Researchers|14|
82|HDTV Researchers|14|
6124|Cloud Storage Provider Researchers|14|
56|Discount Device Shoppers|14|
19250|World Cup Enthusiasts|14|
6149|Readers of Jewish News|14|
18490|Professional Sound Products Researchers|14|
17673|Coffee Chain Shoppers|14|
119|Mortgage Researchers|14|
19297|Alzheimer and Dementia Researchers|14|
18997|Insurance Professionals|14|
33|Kids Clothing Shoppers|14|
6333|Crafting Enthusiasts|14|
6184|Reusable Drinkware Shoppers|14|
35|Mens Clothing Shoppers|14|
18784|Healthcare Provider Education Researchers|14|
130|Automotive Safety Researchers|14|
10979|Hanukkah Celebration Planners|14|
155|Business Travelers|14|
6177|Mens Shaving Products Shoppers|14|
139|Makeup Researchers|14|
18|Nascar Fans|14|
74|Office Supply Shoppers|14|
6263|Physicians|14|
10835|Long Beach California Trip Planners|14|
6303|Atlanta Trip Planners|14|
7534|Canada Trip Planners|14|
6328|Mattress Researchers|14|
48|Charitable Donors|14|
163|Federal Employees|14|
6391|HVAC Service Researchers|14|
11975|Landscape Architects|14|
19628|Oral Hygiene Shoppers|14|
4943|Online Grocery Shoppers|14|
115|Mens Shoe Shoppers|14|
6013|Competitive Sports Participants|14|
6064|Solar Energy Solution Shoppers|14|
6220|Eyeglasses & Contact Lens Buyers|14|
5936|Fishing Enthusiasts|14|
6332|Tire Researchers|14|
6266|Respirologists and Pulmonologists|14|
151|Big & Tall Men|14|
18924|Commercial Banking Market Intelligence Researchers|14|
124|Healthcare Discount Shoppers|14|
5909|Web Design Researchers|14|
4914|Mens Health Researchers|14|
15879|Freelancers|14|
6144|North Carolina Football Fans|14|
19590|Auto Show Enthusiasts|14|
17674|Medical Advice Researchers|14|
6128|Realtors & Real Estate Researchers|14|
12602|Family Medical Practice Journal Researchers|14|
30|Department Store Shoppers|14|
19619|Pet Food Shoppers|14|
54|Online Investors|14|
17452|Clinic & Lab Immunologist Professionals|14|
22411|Chile Trip Planners|14|
4911|Dieters|14|
17540|Vacuum Shoppers|14|
6098|Dental Hygiene Students|14|
45|Moving Services Shoppers|14|
5933|Supply Chain Professionals|14|
19602|Ford Vehicle Shoppers|14|
4936|Liberal Think Tank Readers|14|
5951|Law Enforcement Officers|14|
38|Sporting Goods Shoppers|14|
6029|Indoor Cycling & Spinning Enthusiasts|14|
19621|Portugal Trip Planners|14|
21237|Chelsea Fans|14|
6334|Supermarket Shoppers|14|
6174|Photography Enthusiasts|14|
10838|Detroit Trip Planners|14|
5960|New Zealand Trip Planners|14|
49|NHL Fans|14|
7527|Democratic Donors|14|
6285|Competitive Tri-Athletes|14|
4944|Pre-Measured Grocery Shoppers|14|
10978|Family Christmas Celebration Researchers|14|
5952|Australia Trip Planners|14|
6050|Readers of Malayalam Content|14|
19604|Game of Thrones Fans|14|
12820|Commercial Contractors & Designers|14|
19600|Ducati Motorcycle Shoppers|14|
6367|Cable TV Shoppers|14|
6134|Dentists|14|
10836|Fort Lauderdale Trip Planners|14|
4919|Military Families|14|
6159|United Arab Emirates Trip Planners|14|
6232|Golf Enthusiasts|14|
11977|Civil Engineers|14|
65|Hands-On Parents|14|
20766|Senior Living Center Researchers|14|
10977|Christmas Celebration Researchers|14|
6148|Kosher Recipe Researchers|14|
6250|Tennis Players|14|
21292|Smart Home Product Researchers|14|
10972|Halloween Thrill Seekers|14|
5898|Oncology Researchers|14|
6253|Medicare Researchers|14|
22091|Zoo Visitors|14|
102|Plus Size Women|14|
20751|Commercial Vehicles Shoppers|14|
13498|Restaurant News Readers|14|
12032|Heart Health Researchers|14|
10973|Family Halloween Party Planners|14|
17|MLB Fans|14|
22093|Childrens Museum Visitors|14|
78|Contractors & Construction Professionals|14|
21242|World Cup Apparel Shoppers|14|
6209|Study Abroad Researchers|14|
21326|Video Game Shoppers|14|
17730|Luxury Department Store Shoppers|14|
5897|Orthodontics Researchers|14|
18350|Photo Memory Makers|14|
16137|Olympics Fans|14|
6305|Identity Theft Protection Researchers|14|
6364|Pizza Lovers|14|
22094|Fireworks Enthusiasts|14|
6386|Powerboat Purchasers|14|
145|Property Insurance Researchers|14|
7536|Medicare Price Shoppers|14|
10953|Beard Care Shoppers|14|
17669|Fragrance Shoppers|14|
6088|Whiskey Lovers|14|
12031|Neurologists|14|
14901|Audio Book Listeners|14|
5956|Recreational Sports Participants|14|
17269|At-Home Gym Intenders|14|
17320|March Madness Enthusiasts|14|
10326|IT News Readers|14|
18781|Gastroenterology Journal Researchers|14|
6301|Chicago Trip Planners|14|
7546|Florida Gulf Coast Travel Researchers|14|
6265|Pharmacists|14|
7542|Georgia Trip Planners|14|
32704|Major Airline Customers|14|
32701|Womens Equality Advocates|14|
17786|Womens Fashion Brands Shoppers|14|
6230|Vegans|14|
19605|Harley Davidson Shoppers|14|
22502|Lacrosse Enthusiasts|14|
22420|Online Games Enthusiasts|14|
6308|Food Safety Decision Makers|14|
6300|Washington DC Trip Planners|14|
6|Vacation Planners|14|
129|Aftermarket Accessories Shoppers|14|
6277|Harry Potter Fans|14|
6390|Pool and Spa Researchers|14|
18621|Italian Food Enthusiasts|14|
19593|Classic Car Shoppers|14|
5921|Arizona Trip Planners|14|
67|Lawyers|14|
13497|Restaurant Supply Shoppers|14|
18352|Pediatric Provider Researchers|14|
5969|Luxury Bedding Shoppers|14|
6235|Sailing Enthusiasts|14|
5895|Cardio Health Researchers|14|
6182|Finance Continuing Education Researchers|14|
7535|Medicare Provider Researchers|14|
6217|Weight Loss Researchers|14|
5901|Diabetes|14|
4870|Tech-Savvy Moms|14|
40|Bed & Bath Shoppers|14|
88|Ski and Snowboard Enthusiasts|14|
161|Travel Reward Points Enthusiasts|14|
6207|Anesthesiologists|14|
6319|San Francisco Trip Planners|14|
6143|College Football Fans|14|
5945|Parents of High School Students|14|
5967|Online Auto Shoppers|14|
4940|Auto-Looking for New Car Purchase or Lease|14|
17314|Lobbyists|14|
6173|Camera Shoppers|14|
6337|Diner and Ice Cream Lovers|14|
75|Ink & Toner Shoppers|14|
42|Fast Fashion Shoppers|14|
22398|Vaping Shoppers|14|
6257|Masters in Management Researchers|14|
85|Online Movie Downloaders|14|
16138|Special Olympics Fans|14|
18202|Real Estate Decision Makers|14|
4866|Small Business Employees|14|
18923|Online Home Decor Shoppers|14|
5918|Enterprise Field Services Researchers|14|
21243|Readers of Costa Rican Content|14|
12490|Gourmet Food Purchasers|14|
20754|Used Car Shoppers|14|
19603|France Trip Planners|14|
21238|Manchester United Fans|14|
32705|Certified Events Professionals|14|
7605|Organic Food Buyers|14|
118|Sleep Disorder Researchers|14|
6339|Eyeglasses Wearers|14|
10010|Satellite TV Buyers|14|
11065|Authors|14|
86|Sale Seekers|14|
4858|Business Filing Researchers|14|
19578|Sports Gamblers|14|
17271|Car Collector Enthusiasts|14|
19618|Off Road Accessories Enthusiasts|14|
4902|Yogis|14|
5922|Software Directory Researchers|14|
10981|New Years Eve Party Ticket Purchasers|14|
4932|Marijuana Prohibition Advocates|14|
7425|Life Insurance Researchers|14|
19624|RV Shoppers|14|
17318|Charitable Organization Supporters|14|
19251|Cinco de Mayo Celebrators|14|
61|Beauty & Skincare Buyers|14|
6115|Rugby Fans|14|
107|Cruise Travel Intenders|14|
6329|Mattress Shoppers|14|
6208|Recently Retired Individuals|14|
4927|Immigration Rights Advocates|14|
147|Broadway Fans|14|
5896|Orthopedic Health Researchers|14|
21236|Arsenal Fans|14|
4916|Arthritis Sufferers|14|
12025|Urban Skateboarding Sneaker Shoppers|14|
34468|Hard Rock Hotel & Casino Trip Planners|13|
22422|Poker Card Game Hobbyists|13|
69|Healthy Eaters|13|
8|Business News Readers|13|
33959|Boston Bruins Fans|13|
19633|UFC and MMA Fans|13|
19589|Auto Body Shoppers|13|
33960|Chicago Blackhawks Fans|13|
146|Audio Equipment Researchers|13|
170|Voters|13|
7543|Louisiana Trip Planners|13|
7453|Pilots|13|
4894|Credit Union Researchers|13|
20817|Golden State Warriors Fans|13|
19609|Hyundai Vehicle Shoppers|13|
34465|Toronto Blue Jays Fans|13|
6365|Executive and C-Suite|13|
18648|Architecture Enthusiasts|13|
4907|Headache and Migraine Sufferers|13|
19879|Personal Investment Researchers|13|
34466|Detroit Tigers Fans|13|
116|Home Security Shoppers|13|
20750|BMW Vehicle Shoppers|13|
34077|Candy Shoppers|13|
34463|Boston Red Sox Fans|13|
22401|As Seen on TV Shoppers|13|
62|Streaming Device Shoppers|13|
6162|Healthy Snacks Shoppers|13|
10869|Seattle Seahawks Fans|13|
6160|Eyeglasses Shoppers|13|
34469|Jazz Festival Enthusiasts|13|
34467|Little Rock Trip Planners|13|
6076|Horse Owners|13|
142|Credit Card Researchers|13|
6156|Ancestry Researchers|13|
91|Parents with Toddlers|13|
6259|Word Game Enthusiasts|13|
33474|Daily Fantasy Sports Enthusiasts|13|
14801|Cryptocurrency Enthusiasts|13|
138|Home Kitchen Bakers|13|
6172|Camera Review Readers|13|
112|Auto Repair Researchers|13|
6106|Luxury Second Home Owners|13|
6084|Media Buying Professionals|13|
32485|College Professors|13|
19758|Dog Lovers|13|
33967|New York Rangers Fans|13|
6049|Readers of Portuguese Content|13|
19296|BBQ Enthusiasts|13|
34574|F1 Racing Enthusiasts|13|
22413|China Trip Planners|13|
33971|Sun Protection Shoppers|13|
34464|New York Yankees Fans|13|
6231|Sleep Researchers|13|
34082|New England Patriots Fans|13|
34461|Jazz Music Fans|13|
4930|Occupy Movement Supporters|13|
10364|Documentary Film Fans|13|
22|Grill Masters|13|
34462|Baltimore Orioles Fans|13|
34083|New York Giants Fans|13|
22425|BBQ Recipe Researchers|13|
6227|Personal Stock Portfolio Managers|13|
4915|Food Allergies Researchers|13|
18203|Hunters|13|
125|Postal Employees|13|
6331|Price Conscious Grocery Shoppers|13|
15878|Readers of Catholic News|13|
34078|Cookie Eaters|13|
19627|Subaru Vehicle Shoppers|13|
92|Parents with Kids|13|
6141|Entrepreneurs|13|
4867|Sweepstakes Enthusiasts|13|
34471|Cabin Rental Researchers|13|
59|Menopause Researchers|13|
19606|Home Networking Researchers|13|
13|Advertising Professionals|13|
17671|Crypto Market Investors|13|
6224|Tattoo Fans|13|
128|Kitchen & Bathroom Remodelers|13|
34459|Coffee Bean Shoppers|13|
5930|Tax Filers|13|
24|Home Design Enthusiasts|12|
35905|Athleisure Shoppers|12|
6047|Readers of Tamil Content|12|
103|Live Concert Fans|12|
6194|Readers of Greek Content|12|
20816|Houston Rockets Fans|12|
4848|Working Parents|12|
6034|Backyard Farmers|12|
36|Baby Products Buyers|12|
6008|Digital Antenna Users|12|
168|Social Conservatives|12|
34909|Project Managers|12|
14|NFL Fans|12|
10872|Atlanta Falcons Fans|12|
4904|Adult Acne Sufferers|12|
34086|Pittsburgh Steelers Fans|12|
63|New & Expecting Parents|12|
35904|Utility Workwear Shoppers|12|
35953|Kitchen and Bath Professionals|12|
4895|Mature Parents|12|
6214|Home Remodelers|12|
4917|Teen Content Readers|12|
35169|SMB Commercial Insurance Researchers|12|
19422|Home Design and Living Publication Readers|12|
6221|RC Helicopters and Drones Hobbyists|12|
5920|Business Decision Makers|12|
6065|Solar Energy Researchers|12|
4873|TV Game Show Fans|12|
35906|MS Information Researchers|12|
19608|Honda Vehicle Shoppers|12|
6228|Paleo Eaters|12|
36020|Architects|12|
18647|Gourmet Food & Wine Researchers|12|
4913|Womens Health Researchers|12|
6111|Wine Lovers|12|
4852|Health & Fitness|12|
18351|Drama Show Watchers|12|
71|Lawn & Garden Enthusiasts|12|
4918|Gastrointestinal Researchers|12|
6120|Readers of Puerto Rican Content|12|
4905|Heartburn Sufferers|12|
6127|LED Lighting Shoppers|12|
12277|Market Intelligence Researchers|12|
18646|Recipe Researchers|12|
33968|Oklahoma City Thunder Fans|12|
33969|Pittsburgh Penguins Fans|12|
33964|Milwaukee Bucks Fans|12|
6122|Readers of Cuban Content|12|
35742|Disney Fans|12|
35903|Trendy Denim Shoppers|12|
10870|Philadelphia Eagles Fans|12|
140|Pet Owners|12|
81|Republicans|12|
123|Military Servicemembers & Veterans|12|
1|Fitness Enthusiasts|12|
6225|Piercing Fans|12|
34088|Soda Drinkers|12|
5942|Tax Policy Researchers|12|
21|DIYers|12|
19614|Mazda Vehicle Shoppers|12|
19634|Volkswagen Vehicle Shopper|12|
19597|Disability Insurance Researchers|12|
6006|Breast Cancer Fund Raisers|12|
18620|Mexican Food Enthusiasts|12|
5904|Truck Drivers|12|
36867|Blender Shoppers|11|
13428|Smoking Cessation Intenders|11|
34948|Comedy Movie and TV Enthusiasts|11|
15884|Video Gamers|11|
90|Coupon Researchers|11|
6164|Microsoft Friendly Developers|11|
36882|Airstream Enthusiasts|11|
36873|Quilting Enthusiasts|11|
154|Prepaid Smartphone Shoppers|11|
6089|Mens Fitness Publication Readers|11|
7597|Readers of Spanish Content|11|
37880|Solitaire Players|11|
7|Motorcycle Enthusiasts|11|
14800|Rapid City Trip Planners|11|
26|Streaming Video Fans|11|
36879|Embroidery Enthusiasts|11|
38353|Netherlands Trip Planners|11|
38360|Northeastern Voters|11|
10971|Halloween Enthusiasts|11|
21246|Readers of El Salvadoran Content|11|
36871|Model Building Hobbyists|11|
22424|Cosplay Enthusiasts|11|
37420|Smartphone Case Shoppers|11|
37413|Tailgate Enthusiasts|11|
38366|Green Party Policy Supporters|11|
38356|Grateful Dead Fans|11|
6341|Pharmacy Shoppers|11|
35932|Easter Decorations and Candy Shoppers|11|
38355|Private Jet Traveler|11|
32702|Romantics|11|
16197|Financial Business Intelligence Researchers|11|
19607|Home Theater Enthusiasts|11|
37411|Leather Goods Shoppers|11|
169|Social Liberals|11|
5943|Financial News Readers|11|
36876|Billiards Enthusiasts|11|
36880|Skateboarding Enthusiasts|11|
19594|Corvette Enthusiasts|11|
19595|Diesel Vehicle Shoppers|11|
36860|Casual Dining Restaurant Researchers|11|
37887|Gift Givers|11|
6222|Musical Instrument Purchasers|11|
15935|Pregnancy Resources Researchers|11|
51|Black Friday Deal Shoppers|11|
36862|Soap Shoppers|11|
4869|Womens Fitness Enthusiasts|11|
37414|Motor Scooter Enthusiasts|11|
7598|Readers of South American Content|11|
38359|Christian Voters|11|
19759|Pet Lovers|11|
33966|New York Knicks Fans|11|
6138|Open Source Developers|11|
97|Home Kitchen Chefs|11|
36870|Model Train Enthusiasts|11|
17670|Deodorant Shoppers|11|
6212|Online Daters|11|
6123|Misc-Deal Seekers or Coupon Shoppers|11|
21250|Readers of Colombian Content|11|
37883|Shapewear & Leggings Shoppers|11|
38362|South Atlantic Voters|11|
36884|Dance Clothing Shoppers|11|
21249|Readers of Dominican Republic Content|11|
4933|Family Values Advocates|11|
20753|Tesla Enthusiasts|11|
6048|Readers of Telugu Content|11|
10871|Dallas Cowboys Fans|11|
34637|Buffalo Bills Fans|11|
10007|Mobile Phone Purchase Intenders|11|
34084|New York Jets Fans|11|
37423|Promotional Products Purchasers|11|
36868|Chandelier and Lamp Shoppers|11|
36874|Bowling Enthusiasts|11|
2|Gamers|11|
37419|Light Bulb Shoppers|11|
36864|Vegetarians|11|
6330|Pet Store Goers|11|
93|Young Boomers|11|
4906|Sinus Researchers|11|
19620|PlayStation Enthusiasts|11|
37875|Vermont Trip Planners|11|
38283|Womens Subscription Box Shoppers|11|
22426|Bodybuilding Enthusiasts|11|
19587|Acura Vehicle Shoppers|11|
21247|Readers of Nicaraguan Content|11|
20963|Natural and Holistic Health Researchers|11|
37874|Spain Trip Planners|11|
34638|Carolina Panthers Fans|11|
5962|New Homeowners|11|
37879|Chess Enthusiasts|11|
33965|Minnesota Timberwolves Fans|11|
36883|Dance Enthusiasts|11|
6087|Metal and Rock Music Fans|11|
149|Streaming Radio Listeners|11|
38357|Jam Band Festival Fans|11|
38351|Xtreme Sports Fans|11|
6278|Junk Food Lovers|10|
6145|Fast Food Lovers|10|
34087|San Francisco 49ers Fans|10|
34647|Los Angeles Rams Fans|10|
17803|TV Advertising Professionals|10|
39646|Syracuse Orange Fans|10|
40195|U.S. History Buffs|10|
166|Doves|10|
33972|Toronto Raptors Fans|10|
121|Coders & Developers|10|
34643|Jacksonville Jaguars Fans|10|
39649|Duke Blue Devils Fans|10|
66|Fantasy & Comic Fans|10|
39658|Philadelphia Phillies Fans|10|
37878|New Orleans Pelicans Fans|10|
39336|Philadelphia 76ers Fans|10|
34639|Cleveland Browns Fans|10|
19|Eco-Conscious Consumers|10|
58|Budget Wireless Shoppers|10|
39198|Involved Parents|10|
34460|Ice Cream Shoppers|10|
40438|iPhone Users|10|
40691|Poetry Readers|10|
4861|IT Decision Makers|10|
6151|Wedding Registrants|10|
39768|Hair Loss Prevention Researchers|10|
40190|Luxury Watch Shoppers|10|
53|Hip Hop Fans|10|
21251|Readers of Belizean Content|10|
34276|Romance Movie and TV enthusiasts|10|
6344|Hair Care Shoppers|10|
40683|Aviation Enthusiasts|10|
40435|World War One History Buffs|10|
40434|History Buffs|10|
5971|Readers of Arabic Content|10|
21244|Readers of Guatemalan Content|10|
39656|Clemson Tigers Fans|10|
36878|Drawing Enthusiasts|10|
40183|Canoeing and Kayaking Enthusiasts|10|
38991|Aspen Trip Planners|10|
21248|Readers of Panamanian Content|10|
39665|Aspen Skiers and Snowboarders|10|
34079|Detroit Lions Fans|10|
40525|Hospital Executives|10|
36875|Painting Hobbyist|10|
39657|Kansas City Royals Fans|10|
34648|Los Angeles Chargers Fans|10|
5924|Cloud Security Researchers|10|
38994|Western Canada Skiers and Snowboarders|10|
39196|Retired Assisted Living Shoppers|10|
4899|Readers of Japanese Content|10|
40697|Santa Cruz Trip Planners|10|
40436|Egypt Trip Planners|10|
114|Womens Fashion Magazine Readers|10|
21245|Readers of Honduran Content|10|
40191|Revolutionary War History Buffs|10|
165|Liberal News Readers|10|
4920|LGBT Activists|10|
38993|Washington Skiers and Snowboarders|10|
6306|Newly Recruited Military Personnel|10|
40189|Native American Culture Enthusiasts|10|
19629|Tiny Home Researchers|10|
40439|Ski and Snowboard Apparel Shoppers|10|
40192|Rome Trip Planners|10|
39685|Credit Cards Rewards Researchers|10|
38996|Montana and Idaho Trip Planners|10|
34470|Drive-In Theater Enthusiasts|10|
104|Country Music Fans|10|
38989|Tahoe Skiers and Snowboarders|10|
38995|California Skiers and Snowboarders|10|
4872|Readers of Mexican Content|10|
39195|Retired Government Employees|10|
39193|Medical Science Researchers|10|
40698|GMC Vehicle Shopper|10|
38990|Colorado Skiers and Snowboarders|10|
36863|Hurricane Safety Researchers|10|
3|Car Enthusiasts|10|
39652|Gonzaga Bulldogs Fans|10|
40682|Ergonomic Office Supplies Shoppers|10|
40685|Rodeo Enthusiasts|10|
38992|Oregon Trip Planners|10|
5946|Readers of Filipinos Content|10|
7464|Guitar Enthusiasts|10|
40184|Scuba Diving Enthusiasts|10|
5989|Tech News Enthusiasts|10|
40700|Speedway Event Enthusiasts|9|
34650|Tennessee Titans Fans|9|
41197|Godiva Chocolate Fans|9|
39462|GIS Users|9|
20731|Readers of South Asian Content|9|
41070|Cleveland Indians Fans|9|
39335|Orlando Magic Fans|9|
19632|Truck Shoppers|9|
33961|Detroit Red Wings Fans|9|
41085|Pittsburgh Pirates Fans|9|
41192|Sea World Park Trip Planners|9|
41075|Houston Astros Fans|9|
36877|Crochet Enthusiasts|9|
22415|Colombia Trip Planners|9|
34651|Washington Redskins Fans|9|
41552|Restaurant Menu Researchers|9|
19591|Camaro Enthusiasts|9|
38363|Southeastern Voters|9|
39330|Atlanta Hawks Fans|9|
33962|Chicago Bears Fans|9|
41854|Swimming Enthusiasts|9|
41059|UConn Huskies Fans|9|
164|Conservative News Readers|9|
41083|Cininnati Reds Fans|9|
41190|Sunglasses Shoppers|9|
40437|Podcast Listeners|9|
38365|Midwestern Voters|9|
39331|Denver Nuggets Fans|9|
39334|Memphis Grizzlies Fans|9|
41547|Electronics Shoppers|9|
34644|Kansas City Chiefs Fans|9|
39648|UNC Tar Heels Fans|9|
40696|Palm Beach Trip Planners|9|
41855|Snow Removal Researchers|9|
40699|Automotive Racing Enthusiasts|9|
37881|Trivia Enthusiasts|9|
38368|Volvo Vehicle Shoppers|9|
41052|Health and Fitness Book Readers|9|
37882|Home Aquarium Enthusiasts|9|
39644|Florida State Seminoles Fans|9|
40688|Legal News Readers|9|
37422|Laptop Researchers|9|
41079|Miami Marlins Fans|9|
41078|Atlanta Braves Fans|9|
39642|Notre Dame Fighting Irish Fans|9|
42239|ESPN Enthusiasts|9|
41069|Big East Fans|9|
39339|Utah Jazz Fans|9|
6188|High End TV Shoppers|9|
19760|Cat Lovers|9|
42238|Online Calculator Users|9|
39332|Indiana Pacers Fans|9|
38364|Southern Central Voters|9|
42237|Ebay Shoppers|9|
39640|Big 12 Fans|9|
41076|Los Angeles Angels Fans|9|
37877|Dallas Mavericks Fans|9|
38361|Mid-Atlantic Voters|9|
41550|Africa Trip Planners|9|
41193|Christmas Tree Shoppers|9|
39650|Kentucky Wildcats Fans|9|
41191|Sports Memorabilia Shoppers|9|
40690|Business Book Readers|9|
6181|Cybersecurity Researchers|9|
38354|Train Travelers|9|
20815|Cleveland Cavaliers Fans|9|
37410|Bird Watching Enthusiasts|9|
41055|Travel Book Readers|9|
37524|RV Owners and Upgraders|9|
41553|Classical Music Enthusiasts|9|
41074|Minnesota Twins Fans|9|
34646|New Orleans Saints Fans|9|
6299|Readers of Bengali Content|9|
40692|eBook Readers|9|
34085|Oakland Raiders Fans|9|
41057|Auburn Tigers Fans|9|
39643|Louisiana State University Tigers Fans|9|
40194|Detroit Pistons Fans|9|
41548|Winter Apparel Shoppers|9|
41088|San Francisco Giants Fans|9|
41199|St. Louis Cardinals Fans|9|
41081|Washington Nationals Fans|9|
34642|Cincinnati Bengals Fans|9|
41080|New York Mets Fans|9|
39654|Oklahoma State Cowboys Fans|9|
41087|Arizona Diamondbacks Fans|9|
40703|Automotive Auctions Enthusiasts|9|
41856|Chocolate Lovers|9|
41058|Oklahoma Sooners Fans|9|
41082|Chicago Cubs Fans|9|
34640|Houston Texans Fans|9|
42203|Fitness Activity Tracker Users|9|
37418|Thai Food Enthusiasts|9|
39653|Indiana Hoosiers Fans|9|
19878|Coin Collectors|9|
41067|Arkansas Razorbacks Fans|8|
39770|Sparkling Water Enthusiasts|8|
37165|Home and Garden TV Fans|8|
21059|Executive Travelers|8|
37417|Malaysian Food Enthusiasts|8|
21060|Family Adventures Travelers|8|
38352|Lawn and Home Pest Control Researchers|8|
40526|Condom Shoppers|8|
39197|Budget Grocery Shoppers|8|
38367|Tea Party Policy Supporters|8|
148|Celebrity Gossip Fans|8|
39659|Los Angeles Dodgers Fans|8|
40686|Online Legal Document Service Researchers|8|
39194|Cook at Home Parents|8|
42964|Biohackers|8|
43539|Discount Flight Searchers|8|
43542|Waterfront Vacationers|8|
43544|Craft Beer Enthusiasts|8|
39641|Ohio State Buckeyes Fans|8|
37166|Cooking Show Fans|8|
40187|Organic Food Eaters|8|
37876|Jamband Fans|8|
36872|Knitting Enthusiasts|8|
19612|Kia Vehicle Shoppers|8|
37409|Antiquing Enthusiasts|8|
43540|Voice Over Internet Protocol (VoIP) Users|8|
39767|Mexican Food Shoppers|8|
37406|Metal Detecting Enthusiasts|8|
6363|Readers of Caribbean Content|8|
43550|Meal Kit Delivery Researchers|8|
38350|Indian Food Enthusiasts|8|
57|Mobile Phone Comparison Shoppers|8|
42398|Tea Drinkers|8|
41549|Bitcoin Enthusiasts|8|
41063|Florida Gators Fans|8|
39190|Retired Health Insurance Researchers|8|
21061|Work Hard, Play Hard Travelers|8|
38358|Off the Grid Living Enthusiasts|8|
41054|ACC Fans|8|
41857|Aquarium Trip Planners|8|
40193|Myrtle Beach Trip Planners|8|
43546|Personalized Gift Shoppers|8|
41196|Power Tools Shoppers|8|
41084|Milwaukee Brewers Fans|8|
41195|Ice Skating Enthusiasts|8|
21058|Countdown to the Beach Travelers|8|
37407|Archery Enthusiasts|8|
33963|Denver Broncos Fans|8|
39551|Ride Share Drivers|8|
36869|Wood Carving Enthusiasts|8|
34081|Miami Dolphins Fans|8|
21062|Getting Off the Grid Travelers|8|
42368|Online Thesaurus Users|8|
36343|Computer Processor and Data Center Decision Makers|8|
41072|Wyoming Cowboys Fans|8|
19601|Ford Mustang Enthusiasts|8|
4855|Entertainment & Tabloid Magazine Readers|8|
21057|Work Comes First Travelers|8|
40693|Nature and Outdoors Book Readers|8|
37416|Louisiana Creole Food Enthusiasts|8|
38284|Portuguese Food Enthusiasts|8|
41554|Northwestern Wildcats Fans|8|
34635|Arizona Cardinals Fans|8|
39337|Phoenix Suns Fans|8|
37257|Cold and Flu Prevention Researchers|8|
37415|Korean Food Enthusiasts|8|
36138|Haunted House Researchers|8|
45675|Home Medical Supplies Shoppers|7|
45671|Mens Suit Shoppers|7|
44489|Home Goods Supercenter Shoppers|7|
44450|Trendy Shoe Shoppers|7|
44446|Home Rug Shoppers|7|
22406|Green Bay Packers Fans|7|
22408|Super Mario Bros Fans|7|
44389|Mahjong Players|7|
44581|Darts Enthusiasts|7|
44453|Mens Business Clothes Shoppers|7|
40702|Automotive Service Professionals|7|
45679|Government Student Loan Researchers|7|
44481|Las Vegas Hotel Researchers|7|
44452|Primary Care Doctor Researchers|7|
45677|Sleepwear Shoppers|7|
44480|New Orleans Activities Researchers|7|
43547|Animal Humane Society Helpers|7|
37408|Survivalist Enthusiasts|7|
44585|Ballet Enthusiasts|7|
44588|Softball Enthusiasts|7|
44387|Puzzle Enthusiasts|7|
45673|Fundraising Advocates|7|
44466|Wedding Content Readers|7|
39189|Retired Budget Healthcare Shoppers|7|
44468|Tutoring Resources Researchers|7|
44584|Child Car Seat Researchers|7|
44589|Brooklyn Nets Fans|7|
44469|Home Interior Design Enthusiasts|7|
39192|Bargain Shopping Researchers|7|
44476|New York City Events Researchers|7|
36881|Paintball Enthusiasts|7|
44445|Teaching Resources Researchers|7|
45664|Headphone Shoppers|7|
44477|Houston Activity Researchers|7|
45678|Mens Underwear Shoppers|7|
43545|Homemade Gifts Crafters|7|
44487|San Diego Activities Researchers|7|
44478|San Francisco Activities Researchers|7|
4847|Readers of Hispanic Content|7|
44456|Craft Spirits Enthusiasts|7|
39645|Oregon Ducks Fans|7|
44386|Crossword Puzzles Players|7|
45662|Psychiatrists|7|
39199|Medical Consultation Researchers|7|
45516|Basketball Enthusiasts|7|
44448|Home Custom Bedroom Decorators|7|
45725|Trampoline Shoppers|7|
42011|League of Legends Video Game Fans|7|
44490|Luxury Fitness Shoppers|7|
39333|Los Angeles Clippers Fans|7|
5900|Dentures|7|
44473|Home Coffee Makers|7|
39651|Iowa Hawkeyes Fans|7|
19599|Dodge Vehicle Shoppers|7|
37412|Medieval History Enthusiasts|7|
44479|Nashville Activity Researchers|7|
44381|Symphony Music Enthusiasts|7|
44378|Internet Speed Testers|7|
44383|Guitar Music Researchers|7|
44582|Spy Gadgets Shoppers|7|
44444|Home DIY Enthusiasts|7|
42479|Smoky Mountains Trip Planners|7|
45670|Womens Swimsuit Shoppers|7|
45526|Dairy Industry New Readers|7|
45674|Medical Technology News Readers|7|
40188|Civil War History Buff|7|
44587|Baseball Enthusiasts|7|
34649|Tampa Bay Buccaneers Fans|7|
45518|Volleyball Enthusiasts|7|
45521|Rock Climbing Enthusiasts|7|
33970|San Antonio Spurs Fans|7|
15877|Readers of Pakistani Content|7|
45519|Competitive Wrestling Enthusiasts|7|
20752|Commercial Truck Researchers|7|
44272|Rowing Enthusiasts|7|
39547|Louisville Cardinals Fans|7|
44455|Kitchen Stove Shoppers|7|
41066|Texas AM Aggies Fans|7|
42000|Chicken Recipe Researchers|7|
44465|Luxury Athletic Footwear Shoppers|7|
44472|Country Home Decorators|7|
39338|Portland Trail Blazers Fans|7|
44488|San Diego Events Researchers|7|
41061|Penn State Nittany Lions Fans|7|
34636|Baltimore Ravens Fans|7|
41853|Atlanta United Fans|7|
44447|Home Goods Shoppers|7|
44470|Custom Home Lighting Shoppers|7|
45676|Scrubs Uniforms Shoppers|7|
45667|Blades and Hunting Knife Shoppers|7|
10839|Tampa and St Petersburg Trip Planners|6|
44457|New York City Fine Theatre Enthusiasts|6|
20764|Entertainment Industry Decision Makers|6|
19635|Xbox Enthusiasts|6|
23|Techies|6|
40427|Model Aircraft and Rocket Hobbyists|6|
41065|Tennessee Volunteers Fans|6|
33958|Astrology Enthusiasts|6|
44379|French Language Learners|6|
40695|Alternative Medicine Enthusiasts|6|
19616|Muscle Car Shoppers|6|
41048|SEC Fans|6|
46134|Pontoon Boat Researchers|6|
6298|Readers of Russian Content|6|
44579|Koi Pond Enthusiasts|6|
44586|Opera Enthusiasts|6|
44391|Spanish Language Learners|6|
46132|Counseling Researchers|6|
44449|United Nations Donors|6|
43549|Dog Breed Researchers|6|
39191|Retired Wealthy Assisted Living Researchers|6|
41551|Cookie Recipe Researchers|6|
44491|Disney Vacation Planners|6|
44474|Los Angeles Activities Researchers|6|
44458|Federal Employee Benefits Researchers|6|
41062|Michigan Wolverines Fans|6|
6131|Movie and Game Review Readers|6|
37421|Budget Mobile Phone Researchers|6|
47850|Asheville Trip Planners|6|
10955|Pokemon Enthusiasts|6|
45669|Cocktail Recipe Researchers|6|
45524|Mowing Equipment Shoppers|6|
34641|Indianapolis Colts Fans|6|
45685|Cosmetic Surgery Intenders|5|
44471|Retirement Financial Account Researchers|5|
47713|High School Sports Researchers|5|
48462|Self Storage Shoppers|5|
150|TV Junkies|5|
48154|Elite Cycling Gear Shoppers|5|
45666|Bird and Parrot Lovers|5|
46566|Fertility Treatment Researchers|5|
131|Android Fans|5|
44485|Home Exterior Design Enthusiasts|5|
39647|South Carolina Gamecocks Fans|5|
40687|Legal Advice Researchers|5|
20756|The Walking Dead Fans|5|
16198|World of Warcraft Enthusiasts|5|
44482|Orlando Activities Researchers|5|
48460|Online Payroll Services Users|5|
44385|Sudoku Players|5|
49973|Farm Finance Researchers|5|
44463|Immigration Policy Researchers|5|
33957|Call of Duty Enthusiasts|5|
49976|Agriculture and Climate Advocates|5|
44467|Family Tree Enthusiasts|5|
45520|Cricket Enthusiasts|5|
45515|US Judiciary System Researchers|5|
40689|Legal Help Seekers|5|
48465|HGTV Enthusiasts|5|
47130|Home Lending Researchers|5|
44475|Georgia Activities Researchers|5|
48463|PBS Enthusiasts|5|
49979|Cape Cod News Readers|5|
49502|Veterinarians|5|
44462|Immigration Resource Researchers|5|
41194|Product Manuals Researchers|5|
44459|Foot Pain Sufferers|5|
41546|Reality TV Show Watchers|5|
45517|Football Enthusiasts|5|
49974|Agricultural and Food Issues Researchers|5|
45672|Bible Study Researchers|5|
44583|Sustainable Lifestyle Enthusiasts|4|
51119|Skin Disorder Researchers|4|
44486|Heart Surgery Researchers|4|
43541|Wig Shoppers|4|
45528|Cannabis News Readers|4|
44484|Texas Activity Researchers|4|
51120|Foot Health Researchers|4|
43543|Home Brewing Enthusiasts|4|
45665|Reptile Enthusiasts|4|
44382|Lyrics Researchers|4|
44460|Military Benefits Researchers|4|
50860|Food Delivery Service Users|4|
45522|Hair Color Shoppers|4|
49977|DIY Upcycle Home Project Planners|4|
45527|CBD Oils Shoppers|4|
44388|Board Game Enthusiasts|4|
44380|Home Heating and Cooling Researchers|4|
49978|Homeschooling Parents|4|
40684|Space Travel Enthusiasts|4|
44483|Louisiana Activities Researchers|4|
47065|Diabetes Support Forum Readers|4|
43548|Animal Rights Researchers|4|
45668|Pizza Lovers|4|
44384|Action Figure and Collectibles Shoppers|4|
43551|Virtual Reality Enthusiasts|4|
44537|Milk and Dairy Shoppers|4|
44454|Marines|4|
6010|Star Wars Fans|4|
44390|Background Check Users|4|
44451|Music Trivia Researchers|4|
44464|Health Conscious Expecting Parents|4|
45525|Cattle Industry New Readers|4|
10366|Sci-Fi Movie Fans|3|
18348|Sci-Fi Show Watchers|3|
42399|Farming Simulator Video Game Fans|3|
40694|Anime Fans|3|
45523|Welding Tools and Supplies Shoppers|3|
47115|Electric Vehicle Shoppers|3|
51678|Plumbers|3|
33533|Los Angeles Lakers Fans|3|
15882|Online Role Playing Game Enthusiasts|3|
136|Tablet Researchers|3|
48022|Sushi Lovers|3|
5984|Readers of Indian Content|3|
48459|Work Visa Researchers|3|
49972|Horseback Riding Enthusiasts|3|
44580|Home Cleaning Researchers|3|
34951|Scifi Movie and TV Enthusiasts|2|
6260|Blockbuster Movie Fans|2|
49975|Cattlemen and Pork Producers|2|
46567|Bigfoot Folklore Enthusiasts|2|
34645|Minnesota Vikings Fans|2|
133|High End Camera Shoppers - Dupe|2|
42012|Destiny Video Game Fans|2|
6236|PokemonGo Fans|2|
14802|Online Bank Researchers|2|
39655|Arizona Wildcats Fans|2|
43552|Grand Theft Auto Video Game Fans|2|
17274|Readers of Jamaican Content|2|
15883|Video and Computer Gaming Enthusiasts|1|
42401|Hearthstone Video Game Fans|1|
106|Comedy Fans|1|
10954|Nintendo Enthusiasts|1|
5999|Readers of African American Content|1|
40701|Automotive News Readers|1|
42008|The Sims Video Game Fans|1|
33534|Miami Heat Fans|1|
34950|Action Movie and TV Enthusiasts|1|
44461|Gun Violence Researchers|1|
10011|Big Box Shoppers|1|
10365|Horror Movie Fans|1|
19626|Star Trek Fans|1|


***
  
**6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where 'interest_id = 21246' in your joined output and include all columns from `fresh_segments.interest_metrics` and all columns from `fresh_segments.interest_map` except from the id column.**

We should be using `INNER JOIN` to perform our analysis.

```sql
SELECT *
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.interest_id = 21246   
  AND metrics._month IS NOT NULL; -- There were instances when interest_id is available, however the date values were not - hence filter them out.
```

_month|_year|interest_id|composition|index_value|ranking|percentile_ranking|month_year|interest_name|interest_summary|created_at|last_modified|
|-------|---------|------|-------|----------|-------------|-----------|------------------------|------------|------------|--------------------------------|
7.0|2018.0|21246|2.26|0.65|722|0.96|01-07-2018|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
8.0|2018.0|21246|2.13|0.59|765|0.26|01-08-2018|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
9.0|2018.0|21246|2.06|0.61|774|0.77|01-09-2018|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
10.0|2018.0|21246|1.74|0.58|855|0.23|01-10-2018|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
11.0|2018.0|21246|2.25|0.78|908|2.16|01-11-2018|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
12.0|2018.0|21246|1.97|0.7|983|1.21|01-12-2018|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
1.0|2019.0|21246|2.05|0.76|954|1.95|01-01-2019|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
2.0|2019.0|21246|1.84|0.68|1109|1.07|01-02-2019|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
3.0|2019.0|21246|1.75|0.67|1123|1.14|01-03-2019|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
4.0|2019.0|21246|1.58|0.63|1092|0.64|01-04-2019|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|
nan|nan|21246|1.61|0.68|1191|0.25|nan|Readers of El Salvadoran Content|People reading news from El Salvadoran media sources.|11-06-2018 17:50|11-06-2018 17:50|



The results should come up to 10 rows only. 

***

**7. Are there any records in your joined table where the `month_year` value is before the `created_at` value from the `fresh_segments.interest_map` table? Do you think these values are valid and why?**

```sql
SELECT 
  COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < map.created_at::DATE;
```

<kbd><img width="106" alt="image" src="https://user-images.githubusercontent.com/81607668/139017976-48aade91-969c-432f-83b3-a14436f66056.png"></kbd>

There are 188 records where the `month_year` date is before the `created_at` date. 

However, it looks like these records are created in the same month as `month_year`. Do you remember that the `month_year` column's date is set to default on 1st day of the month? 

<kbd><img width="761" alt="image" src="https://user-images.githubusercontent.com/81607668/139018053-f948b63a-d502-4337-b347-8c24f736f32f.png"></kbd>

Running another test to see whether date in `month_year` and `created_at` are in the same month.

```sql
SELECT 
  COUNT(*)
FROM fresh_segments.interest_map map
INNER JOIN fresh_segments.interest_metrics metrics
  ON map.id = metrics.interest_id
WHERE metrics.month_year < DATE_TRUNC('mon', map.created_at::DATE);
```

<kbd><img width="110" alt="image" src="https://user-images.githubusercontent.com/81607668/139018367-ab5b5148-a2e1-4b53-968e-eedb7eb717a3.png"></kbd>

Seems like all the records' dates are in the same month, hence we will consider the records as valid. 

***

## 📚 B. Interest Analysis
  
**1. Which interests have been present in all `month_year` dates in our dataset?**

Find out how many unique `month_year` in dataset.

```sql
SELECT 
  COUNT(DISTINCT month_year) AS unique_month_year_count, 
  COUNT(DISTINCT interest_id) AS unique_interest_id_count
FROM fresh_segments.interest_metrics;
```

<img width="465" alt="image" src="https://user-images.githubusercontent.com/81607668/139030151-64461d42-4215-4da1-bc6e-c701b9a8f357.png">

There are 14 distinct `month_year` dates and 1202 distinct `interest_id`s.

```sql
WITH interest_cte AS (
SELECT 
  interest_id, 
  COUNT(DISTINCT month_year) AS total_months
FROM fresh_segments.interest_metrics
WHERE month_year IS NOT NULL
GROUP BY interest_id
)

SELECT 
  c.total_months,
  COUNT(DISTINCT c.interest_id)
FROM interest_cte c
WHERE total_months = 14
GROUP BY c.total_months
ORDER BY count DESC;
```

<img width="263" alt="image" src="https://user-images.githubusercontent.com/81607668/139029765-3403fb8b-e93d-4fde-989b-b648d62fcb3f.png">

480 interests out of 1202 interests are present in all the `month_year` dates.

***
  
**2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?**

Find out the point in which interests present in a particular number of months are not performing well. For example, interest id 101 only appeared in 6 months due to non or lack of clicks and interactions, so we can consider to cut the interest off. 

```sql
WITH cte_interest_months AS (
SELECT
  interest_id,
  MAX(DISTINCT month_year) AS total_months
FROM fresh_segments.interest_metrics
WHERE interest_id IS NOT NULL
GROUP BY interest_id
),
cte_interest_counts AS (
  SELECT
    total_months,
    COUNT(DISTINCT interest_id) AS interest_count
  FROM cte_interest_months
  GROUP BY total_months
)

SELECT
  total_months,
  interest_count,
  ROUND(100 * SUM(interest_count) OVER (ORDER BY total_months DESC) / -- Create running total field using cumulative values of interest count
      (SUM(INTEREST_COUNT) OVER ()),2) AS cumulative_percentage
FROM cte_interest_counts;
```

<img width="446" alt="image" src="https://user-images.githubusercontent.com/81607668/139035737-cfe32a44-5c48-4376-a9bc-96c15daf162b.png">

Interests with total months of 6 and above received a 90% and above percentage. Interests below this mark should be investigated to improve their clicks and customer interactions. 
***

**3. If we were to remove all `interest_id` values which are lower than the `total_months` value we found in the previous question - how many total data points would we be removing?**

***

**4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective. **

***

**5. If we include all of our interests regardless of their counts - how many unique interests are there for each month?**
  
***

## 🧩 C. Segment Analysis
 
1. Using the complete dataset - which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
2. Which 5 interests had the lowest average ranking value?
3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?

***

## 👆🏻 D. Index Analysis

The `index_value` is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients. Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

1. What is the top 10 interests by the average composition for each month?
2. For all of these top 10 interests - which interest appears the most often?
3. What is the average of the average composition for the top 10 interests for each month?
4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?

***

Do give me a 🌟 if you like what you're reading. Thank you! 🙆🏻‍♀️
