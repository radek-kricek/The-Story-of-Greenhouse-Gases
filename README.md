# The Story of Greenhouse Gases

My oldest Python & SQL project! Would make the code different now but so proud of the dashboard!

How do individual countries and sectors contribute to global warming? What are the trends and is economic growth always coupled with higher emissions? See the results in an interactive [Tableau story](https://public.tableau.com/app/profile/radek.k.ek/viz/TheStoryofGreenhouseGasesAndHowTheyRelatetoEconomicGrowth/TheStoryofGreenhouseGasesAndHowTheyRelatetoEconomicGrowth).

I created this project as the final project of the [Praha Coding School](https://prahacoding.cz/) (PCS) online data analytics course. I chose the theme because of my long-time interest in nature conservation and climate change.

I used **Python** to do initial cleaning and transformation of data, **SQL** to prepare a number of useful views and **Tableu Desktop** for visualizations.




## The Questions

1. Map the evolution of the main GHG (greenhouse gas) emissions overall and in individual countries.
2. Map the structure of GHG emissions with respect to individual gases and sectors ofo human activity.
    - Identify the room for improvement for individual countries.
3. Calculate the carbon footprint of one unit of GDP.
4. Establish the link between changes in GDP and GHG emissions.
    - Find best practices examples.
    


## The Data

The data for GHG emissions were downloaded as .xlsx files from the Joint Research Centre (EC-JRC)/Netherlands Environmental Assessment Agency (PBL), Emissions Database for Global Atmospheric Research (EDGAR) and IEA data from IEA (2021) World Energy Balances:

*Crippa, M., Guizzardi, D., Banja, M., Solazzo, E., Muntean, M., Schaaf, E., Pagani, F., Monforti-Ferrario, F., Olivier, J., Quadrelli, R., Risquez Martin, A., Taghavi-Moharamli, P., Grassi, G., Rossi, S., Jacome Felix Oom, D., Branco, A., San-Miguel-Ayanz, J. and Vignati, E., CO2 emissions of all world countries - 2022 Report, EUR 31182 EN, Publications Office of the European Union, Luxembourg, 2022, doi:10.2760/730164, JRC130363.*

Available at <https://edgar.jrc.ec.europa.eu/dataset_ghg70>

The data on GDP were downloaded as a CSV file from the National Accounts Main Aggregates Database of the Department of Economic and Social Affairs of the United Nations.

Available at <https://unstats.un.org/unsd/snaama/Basic>


## Cleaning and Transformations in Python

In the Eclipse integrated development environment, I modified Python scripts prepared by the PCS to upload and modify the data files and export them into the used SQL database. I modified predefined procedures and functions to sum up monthly GHG contributions in each year and created two new functions - `find_region` and `rename_country`.

```
# sums up monthly emissions and writes the result into a new column, assigns a region, renames countries in the GDP dataset:
def prepare_table(**dfs):
    for key, df in dfs.items():
        if key in ('sectors_methane', 'sectors_carbon_dioxide', 'sectors_nitrous_oxide', 'totals_methane', 'totals_carbon_dioxide', 'totals_nitrous_oxide'):
            df['yearly'] = df.fillna(0)['Jan'] + df.fillna(0)['Feb'] + df.fillna(0)['Mar'] + df.fillna(0)['Apr'] + df.fillna(0)['May'] + df.fillna(0)['Jun']
            + df.fillna(0)['Jul'] + df.fillna(0)['Aug'] + df.fillna(0)['Sep'] + df.fillna(0)['Oct'] + df.fillna(0)['Nov'] + df.fillna(0)['Dec']
            df[['region']] = df[['Name']].apply(
                    lambda row:
                        build_new_columns_region(key,row),
                    axis=1,
                    result_type='expand'
                )
            print(f"\n## Yerly emissions calculated for df {key}\n",df)
            df.info()
        if key == 'gdp_constant_prices':
            df[['Name']] = df[['Country/Area']].apply(
                    lambda row:
                        rename_column(key,row),
                    axis=1,
                    result_type='expand'
                )
            df[['GDP']] = df[['GDP, at constant 2015 prices - US Dollars']]
            print(f"\n## Yearly GDP at constant 2015 prices for countries in df {key}\n",df)
            df.info()

def rename_column(key, row):
    name = row['Country/Area']
    new_name = rename_country(name)
    output = [new_name]
    #print('provedeno prejmenovani')
    return output

def build_new_columns_region(key,row):
    name = row['Name']
    region = find_region(name)
    output = [region]
    return output
```

The `find_region` function assigns a world region to each country:

```
def find_region(country):
    rest_america = ('Bahamas', 'Bermuda', 'Canada', 'Anguilla', 'Antigua and Barbuda', 'Argentina', 'Aruba', 'Barbados', 'Belize', 'Bolivia', 'Brazil', 'Cayman Islands',
                    'Colombia', 'Costa Rica', 'Cuba', 'Dominica', 'Dominican Republic', 'Ecuador', 'El Salvador', 'Falkland Islands (Malvinas)', 'French Guiana', 'Grenada',
                    'Guadeloupe', 'Guatemala', 'Guyana', 'Haiti', 'Honduras', 'Chile', 'Jamaica', 'Martinique', 'Mexico', 'Netherlands Antilles', 'Nicaragua', 'Panama',
                    'Paraguay', 'Peru', 'Puerto Rico', 'Saint Kitts and Nevis', 'Saint Lucia', 'Saint Pierre and Miquelon',
                    'Saint Vincent and the Grenadines', 'Suriname', 'Trinidad and Tobago', 'Turks and Caicos Islands', 'Venezuela', 'Virgin Islands_British', 'Uruguay')
    eu = ('Austria', 'Belgium', 'Bulgaria', 'Croatia', 'Cyprus', 'Czech Republic', 'Denmark', 'Estonia', 'Finland', 'France', 'Germany', 'Greece', 'Hungary', 'Ireland',
          'Italy', 'Latvia', 'Lithuania', 'Luxembourg', 'Malta', 'Netherlands', 'Poland', 'Portugal', 'Romania', 'Slovakia', 'Slovenia', 'Spain', 'Sweden')
    rest_europe = ('Albania', 'Belarus', 'Bosnia and Herzegovina', 'Faroe Islands', 'Gibraltar', 'Greenland', 'Iceland', 'Macedonia, the former Yugoslav Republic of',
                   'Moldova, Republic of', 'Norway', 'Serbia and Montenegro', 'Switzerland', 'Ukraine', 'United Kingdom')
    africa = ('Algeria', 'Angola', 'Benin', 'Botswana', 'Burkina Faso', 'Burundi', 'Cameroon', 'Cape Verde', 'Central African Republic', 'Comoros', 'Congo',
              'Congo_the Democratic Republic of the', 'Cote d\'Ivoire', 'Djibouti', 'Egypt', 'Equatorial Guinea', 'Eritrea', 'Ethiopia', 'Gabon', 'Gambia', 'Ghana', 'Guinea',
              'Guinea-Bissau', 'Chad', 'Kenya', 'Lesotho', 'Liberia', 'Libyan Arab Jamahiriya', 'Madagascar', 'Malawi', 'Mali', 'Mauritania', 'Mauritius', 'Morocco',
              'Mozambique', 'Namibia', 'Niger', 'Nigeria', 'Reunion', 'Rwanda', 'Saint Helena', 'Sao Tome and Principe', 'Senegal', 'Seychelles', 'Sierra Leone', 'Somalia',
              'South Africa', 'Sudan', 'Swaziland', 'Tanzania_United Republic of', 'Togo', 'Tunisia', 'Uganda', 'Western Sahara', 'Zambia', 'Zimbabwe')
    rest_asia = ('Afghanistan', 'Armenia', 'Azerbaijan', 'Bahrain', 'Bangladesh', 'Bhutan', 'Brunei Darussalam', 'Cambodia', 'Georgia', 'Indonesia',
                 'Iran, Islamic Republic of', 'Iraq', 'Israel', 'Japan', 'Jordan', 'Kazakhstan', 'Korea, Democratic People\'s Republic of', 'Korea, Republic of', 'Kuwait',
                 'Kyrgyzstan', 'Lao People\'s Democratic Republic', 'Lebanon', 'Malaysia', 'Maldives', 'Mongolia', 'Myanmar', 'Nepal', 'Oman', 'Pakistan', 'Palau',
                 'Papua New Guinea', 'Philippines', 'Qatar', 'Saudi Arabia', 'Singapore', 'Sri Lanka', 'Syrian Arab Republic', 'Taiwan_Province of China', 'Tajikistan',
                 'Thailand', 'Timor-Leste', 'Turkey', 'Turkmenistan', 'United Arab Emirates', 'Uzbekistan', 'Viet Nam', 'Yemen', 'Russian Federation')
    australia_oceania = ('Australia', 'Cook Islands', 'Fiji', 'French Polynesia', 'Kiribati', 'New Caledonia', 'New Zealand', 'Samoa', 'Solomon Islands', 'Tonga', 'Vanuatu')
    china = ('Hong Kong', 'Macao', 'China')
    us = ('United States')
    india = ('India')
    non_national = ('Int. Aviation', 'Int. Shipping')
    if country in rest_america:
        region = 'Rest America'
    if country in eu:
        region = 'European Union'
    if country in rest_europe:
        region = 'Rest Europe'
    if country in africa:
        region = 'Africa'
    if country in rest_asia:
        region = 'Rest Asia'
    if country in australia_oceania:
        region = 'Australia and Oceania'
    if country in china:
        region = 'China'
    if country in us:
        region = 'United States'
    if country in india:
        region = 'India'
    if country in non_national:
        region = 'Int. Traffic'
    return region
```

The `rename_country` changes country names in the data on GDP so they are in accordance with the names in the GHG data:

```
def rename_country(country):
    if country == 'Bolivia (Plurinational State of)':
        new_country = 'Bolivia'
    elif country == 'Cabo Verde':
        new_country = 'Cape Verde'
    elif country == 'Democratic Republic of the Congo':
        new_country = 'Congo_the Democratic Republic of the'
    elif country == 'Côte d\'Ivoire':
        new_country = 'Cote d\'Ivoire'  
    elif country == 'Czechia':
        new_country = 'Czech Republic'      
    elif country == 'Czechia':
        new_country = 'Czech Republic'
    elif country == 'Kingdom of Eswatini':
        new_country = 'Swaziland'
    elif country == 'China, Hong Kong SAR':
        new_country = 'Hong Kong' 
    elif country == 'China (mainland)':
        new_country = 'China'
    elif country == 'China, Macao Special Administrative Region':
        new_country = 'Macao'
    elif country == 'Republic of Korea':
        new_country = 'Korea, Republic of'
    elif country == 'Libya':
        new_country = 'Libyan Arab Jamahiriya'
    elif country == 'Republic of North Macedonia':
        new_country = 'Macedonia, the former Yugoslav Republic of'
    elif country == 'Republic of Moldova':
        new_country = 'Moldova, Republic of'
    elif country == 'Former Netherlands Antilles':
        new_country = 'Netherlands Antilles'
    elif country == 'Türkiye':
        new_country = 'Turkey'
    elif country == 'United Kingdom of Great Britain and Northern Ireland':
        new_country = 'United Kingdom'
    elif country == 'Venezuela (Bolivarian Republic of)':
        new_country = 'Venezuela'
    elif country == 'British Virgin Islands':
        new_country = 'Virgin Islands_British'
    else:
        new_country = country
    return new_country
```

The tables for GHG emissions (in countries and sectors) and GDP (after some SQL changes regarding primary keys, see below) have the following structure:

![table_GHG](https://user-images.githubusercontent.com/130282838/230770916-d6b1c6cc-24c2-4c85-b670-18302e8a6a36.png)
![table_gdp](https://user-images.githubusercontent.com/130282838/230770903-409ac2a9-f8a7-4197-a214-f3d5538f2ffb.png)



## Transformations and Analysis in SQL

Further transformations and analysis were performed in Eclipse using SQL. The work consisted mainly of creating insigts by defining new variables and of preparing inputs for Tableu Desktop.

The first new variable was ***warming equivalent***. It is possible to express the effect of GHG emissions on future temperature in several ways. After some study, I decided to use coefficients allowing to express the CO<sub>2</sub> equivalent which would have to be released to have the same effect on Earth surface temperature in 100 years from the gas release. These coefficients are of course different for each GHG and even differ for methane of fossil and non-fossil origin. Apart from CO<sub>2</sub> and methane, also N<sub>2</sub>O data were available and taken into account. All the effects of individual GHGs can thus be summed up and contribute to the overall warming equivalent which is then expressed in kg of CO<sub>2</sub>.

The values were taken from *Forster, P., T. Storelvmo, K. Armour, W. Collins, J.-L. Dufresne, D. Frame, D.J. Lunt, T. Mauritsen, M.D. Palmer,
M. Watanabe, M. Wild, and H. Zhang, 2021: The Earth’s Energy Budget, Climate Feedbacks, and Climate
Sensitivity. In Climate Change 2021: The Physical Science Basis. Contribution of Working Group I to the Sixth
Assessment Report of the Intergovernmental Panel on Climate Change [Masson-Delmotte, V., P. Zhai, A. Pirani,
S.L. Connors, C. Péan, S. Berger, N. Caud, Y. Chen, L. Goldfarb, M.I. Gomis, M. Huang, K. Leitzell, E. Lonnoy,
J.B.R. Matthews, T.K. Maycock, T. Waterfield, O. Yelekçi, R. Yu, and B. Zhou (eds.)]. Cambridge University Press,
Cambridge, United Kingdom and New York, NY, USA, pp. 923–1054, doi:10.1017/9781009157896.009.*

I used multiple subqueries to extract data from tables with individuals GHGs and pivoting to make a view with all GHGs in one column for Tableu:

```
CREATE OR REPLACE VIEW v_ghgs AS
SELECT
	Name,
	region,
	Year,
	yearly_carbon_dioxide,
	4.7*yearly_methane_bio + 7.5*yearly_methane_fossil as eq_methane,
	233*yearly_nitrous_oxide as eq_nitrous_oxide,
	yearly_carbon_dioxide + 4.7*yearly_methane_bio + 7.5*yearly_methane_fossil + 233*yearly_nitrous_oxide as equivalent
FROM
	(SELECT
		Name,
		region,
		Year,
		yearly as yearly_carbon_dioxide,
		(SELECT coalesce(sum(yearly),0) from py_sectors_methane where fossil_bio = 'bio' and py_sectors_methane.Name = py_totals_carbon_dioxide.Name and py_sectors_methane.Year = py_totals_carbon_dioxide.Year) as yearly_methane_bio,
		(SELECT coalesce(sum(yearly),0) from py_sectors_methane where fossil_bio = 'fossil' and py_sectors_methane.Name = py_totals_carbon_dioxide.Name and py_sectors_methane.Year = py_totals_carbon_dioxide.Year) as yearly_methane_fossil,
		(SELECT yearly from py_totals_nitrous_oxide where py_totals_nitrous_oxide.Name = py_totals_carbon_dioxide.Name and py_totals_nitrous_oxide.Year = py_totals_carbon_dioxide.Year) as yearly_nitrous_oxide
		FROM py_totals_carbon_dioxide) as top
;
```

```
### pivoting wide to long

CREATE OR REPLACE VIEW v_ghgs_pivoted AS
SELECT
	Name,
	region,
	Year,
	'carbon_dioxide' as type_ghg,
	yearly_carbon_dioxide as equivalent
FROM
	v_ghgs
UNION ALL
SELECT
	Name,
	region,
	Year,
	'methane' as type_ghg,
	eq_methane as equivalent
FROM
	v_ghgs
UNION ALL
SELECT
	Name,
	region,
	Year,
	'nitrous_oxide' as type_ghg,
	eq_nitrous_oxide as equivalent
FROM
	v_ghgs
ORDER BY Name
;
```

Subqueries were used also for creating a view expressing warming equivalents in individual sectors and countries. To fasten the computation, primary keys were created as unique identificators combining for relevant columns in the first step.

```
ALTER TABLE pcsda_radekk.py_sectors_carbon_dioxide  MODIFY COLUMN Name VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_carbon_dioxide  MODIFY COLUMN ipcc_code_2006_for_standard_report VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_carbon_dioxide  MODIFY COLUMN fossil_bio VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_carbon_dioxide ADD CONSTRAINT py_sectors_carbon_dioxide_PK PRIMARY KEY (Name,`Year`,ipcc_code_2006_for_standard_report,fossil_bio);

ALTER TABLE pcsda_radekk.py_sectors_methane  MODIFY COLUMN Name VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_methane  MODIFY COLUMN ipcc_code_2006_for_standard_report VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_methane  MODIFY COLUMN fossil_bio VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_methane ADD CONSTRAINT py_sectors_methane_PK PRIMARY KEY (Name,`Year`,ipcc_code_2006_for_standard_report,fossil_bio);

ALTER TABLE pcsda_radekk.py_sectors_nitrous_oxide  MODIFY COLUMN Name VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_nitrous_oxide  MODIFY COLUMN ipcc_code_2006_for_standard_report VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_nitrous_oxide  MODIFY COLUMN fossil_bio VARCHAR(150) CHARACTER SET utf8mb4 COLLATE utf8mb4_czech_ci NULL;
ALTER TABLE pcsda_radekk.py_sectors_nitrous_oxide ADD CONSTRAINT py_sectors_nitrous_oxide_PK PRIMARY KEY (Name,`Year`,ipcc_code_2006_for_standard_report,fossil_bio);

CREATE OR REPLACE VIEW v_sectors_ghgs AS
SELECT
	Name,
	region,
	sector_code,
	sector_name,
	fossil_bio,
	Year,
	yearly_carbon_dioxide,
	CASE
		when fossil_bio='bio' then 4.7*yearly_methane
		when fossil_bio='fossil' then 7.5*yearly_methane
	END as eq_methane,
	233*yearly_nitrous_oxide as eq_nitrous_oxide,
  	CASE
  		when fossil_bio='bio' then COALESCE(yearly_carbon_dioxide,0) + 4.7*COALESCE(yearly_methane,0) + 233*COALESCE(yearly_nitrous_oxide,0)
  		when fossil_bio='fossil' then COALESCE(yearly_carbon_dioxide,0) + 7.5*COALESCE(yearly_methane,0) + 233*COALESCE(yearly_nitrous_oxide,0)
  	END as yearly_equivalent
FROM
	(SELECT
		Name,
		region,
		ipcc_code_2006_for_standard_report as sector_code,
		ipcc_code_2006_for_standard_report_name as sector_name,
		fossil_bio,
		Year,
		yearly as yearly_carbon_dioxide,
		(SELECT yearly from py_sectors_methane where py_sectors_methane.Name = py_sectors_carbon_dioxide.Name and py_sectors_methane.Year = py_sectors_carbon_dioxide.Year
			and py_sectors_methane.ipcc_code_2006_for_standard_report = py_sectors_carbon_dioxide.ipcc_code_2006_for_standard_report and py_sectors_methane.fossil_bio = py_sectors_carbon_dioxide.fossil_bio)
			as yearly_methane,
		(SELECT yearly from py_sectors_nitrous_oxide where py_sectors_nitrous_oxide.Name = py_sectors_carbon_dioxide.Name and py_sectors_nitrous_oxide.Year = py_sectors_carbon_dioxide.Year
			and py_sectors_nitrous_oxide.ipcc_code_2006_for_standard_report = py_sectors_carbon_dioxide.ipcc_code_2006_for_standard_report and py_sectors_nitrous_oxide.fossil_bio = py_sectors_carbon_dioxide.fossil_bio)
			as yearly_nitrous_oxide
		FROM py_sectors_carbon_dioxide) as top
;
```

The view below shows GDP for countries in different years. Because some countries fell apart in recent history or were split into more territories unlike in the case of GHG data, they were merged together using useful SQL commands so both data sets could be later joined together.

```
CREATE OR REPLACE VIEW v_countries_gdp_constant_prices AS
SELECT
	Name,
	Year,
	GDP
FROM py_gdp_constant_prices
UNION
SELECT
	'Tanzania_United Republic of' as Name,
	Year,
	SUM(GDP)
FROM py_gdp_constant_prices
WHERE Name like '%Tanzania%'
GROUP BY Year
UNION
SELECT
	'Serbia and Montenegro' as Name,
	Year,
	SUM(GDP)
FROM py_gdp_constant_prices
WHERE Name in ('Kosovo','Montenegro','Serbia')
GROUP BY Year
UNION
SELECT
	'Sudan' as Name,
	Year,
	SUM(GDP)
FROM py_gdp_constant_prices
WHERE Name like '%Sudan%'
GROUP BY Year
;
```

The two following views combine GHG emissions and GDP data together and introduce the second new variable, the ***warming cost***. I defined the warming cost as the ration between warming equivalent and GDP to express how much a unit of wellbeing costs in teh term fo future Earth warming. The coefficient ensures that the unit is g/$.

```
### joins warming equivalent and gdp (at 2015 prices)

CREATE OR REPLACE VIEW v_ghgs_gdp_constant_prices AS
SELECT
	v_ghgs.Name as Name,
	v_ghgs.region as region,
	v_ghgs.Year as Year,
	equivalent,
	round(GDP) as GDP,
	round(1000000000*equivalent/GDP) as warming_cost  # gram/dollar
FROM
	v_ghgs
	inner join v_countries_gdp_constant_prices
	on v_ghgs.Name = v_countries_gdp_constant_prices.Name and v_ghgs.Year = v_countries_gdp_constant_prices.Year
;


### calculates equivalents, GDPs and warming costs for every year in regions

CREATE OR REPLACE VIEW v_warming_costs_regions AS
SELECT
	region,
	Year,
	sum_eq,
	sum_gdp,
	round(1000000000*sum_eq/sum_gdp) as warming_cost
FROM
	(SELECT
		region,
		Year,
		sum(equivalent) as sum_eq,
		sum(GDP) as sum_gdp		
	FROM v_ghgs_gdp_constant_prices
	GROUP BY region, Year) as top
;
```

The last code shows a view comparing the percentage changes in warming equivalent and GDP between 2000 and 2021.

```
### selects countries and differences in equivalent and GDP between 2000 and 2021

CREATE OR REPLACE VIEW v_changes AS
SELECT
	Name,
	region,
	round((eq_end/eq_start-1)*100,1) as percent_change_eq,
	round((gdp_end/gdp_start-1)*100,1) as percent_change_gdp
FROM
	(SELECT
	Name,
	region,
	SUM(CASE
	  WHEN Year = '2000' THEN equivalent ELSE 0 END
	) AS eq_start,
	SUM(CASE
	  WHEN Year = '2021' THEN equivalent ELSE 0 END
	) AS eq_end,
	SUM(CASE
	  WHEN Year = '2000' THEN GDP ELSE 0 END
	) AS gdp_start,
	SUM(CASE
	  WHEN Year = '2021' THEN GDP ELSE 0 END
	) AS gdp_end
	FROM
		(SELECT
			Name,
			region,
			Year,
			equivalent,
			GDP
		FROM v_ghgs_gdp_constant_prices
		WHERE Year=2000 or Year=2021) as top
	GROUP BY Name, region) as top
ORDER BY Name
;
```

## Results and Visualizations

The answers for initial questions can be explored in an interactive [Tableau story](https://public.tableau.com/app/profile/radek.k.ek/viz/TheStoryofGreenhouseGasesAndHowTheyRelatetoEconomicGrowth/TheStoryofGreenhouseGasesAndHowTheyRelatetoEconomicGrowth). The user can go step by step through a series of partially interactive dashboards with text explanations.

It shows how the emissions of carbon dioxide (CO<sub>2</sub>), methane (CH<sub>4</sub>) and nitrous oxide (N<sub>2</sub>O) evolved during the time and in different regions. The effect of all these gases is visualized by the newly defined warming equivalent. The contribution of different sectors to the warming equivalent is shown globally and in regions. The time evolution of GDP and warming cost are shown for regions and countries and the final plot exhibits the changes in both variables between 2000 and 2021.

Using the plots, we can identify the worst and best performing countries in different cathegories while the texts highlight some easily adopted misconceptions in this highly complex topic. I personally find the last graph the most interesting, putting into perspective the developments in the past two decades across the world.
