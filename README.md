# spmap-texas
Create a simple map of Texas counties in STATA with spmap
Example displays DFPS regions: https://www.dfps.state.tx.us/Contact_Us/regional_map.asp


### General instructions

* install prerequisite programs
```
ssc install spmap
ssc install shp2dta
```
* make US county shapefiles
``
shp2dta using cb_2015_us_county_500k, database(usdb) coordinates(uscoord) genid(id)
``
* create fips variable by adding State (2 digits) with County (3 digits)
* fips converted to numeric because STATA doesn't like the leading 0
```
use usdb, replace
gen fips=STATEFP+COUNTYFP
destring fips, replace
```
* save as usdb2
```
save usdb2, replace
```
* open base dataset with HHS data region as an example
* merge in any data along the county identifiers
```
use texas_counties.dta, clear
```
* merge with map database usdb2 using fips county identifier 
```
merge 1:1 fips using usdb2
```
* if the dataset only has texas counties, the non-texas counties will not be merged
* drop the non-texas counties
```
drop if _merge==2
```
* map the base code using the uscoord_mod dataset 
* mod - has coordinates with Alaska and Hawaii moved

*base test map
```
spmap public_health_region using uscoord_mod, id(id)
```
* example with quantiles
```
spmap public_health_region using uscoord_mod, id(id) fcolor(YlGnBu) clmethod(quantile) legenda(on) legend(symy(*2) symx(*2) position(11)) ocolor(gs5 ..) osize(thin) ndocolor(gs6 ..) ndsize(thin) ndlabel(N/A) title("Center for Health Statistics - Public Health Regions", size(*0.8)) subtitle("2017", size(*0.8)) note("Source: https://www.dshs.texas.gov/chs/info/info_txco.shtm")

// example with unique values and different colors
spmap public_health_region using uscoord_mod, id(id) fcolor(Spectral) clmethod(unique) legenda(on) legend(symy(*1) symx(*1) position(11)) ocolor(gs5 ..) osize(thin) ndocolor(gs6 ..) ndsize(thin) ndlabel(N/A) title("Center for Health Statistics - Public Health Regions", size(*0.8)) subtitle("2017", size(*0.8)) note("Source: https://www.dshs.texas.gov/chs/info/info_txco.shtm")

graph export map_txhealthregions.png, width(500) replace

// DFPS Service Regions
// use and merge with tx_county
use tx_county_data
drop public_health_region
ren county_name county
drop _merge
merge 1:1 county using dfps_region
drop NAME
drop _merge
ren county county_name
// rename fips to county, if necessary
// ren county fips
merge 1:1 fips using usdb2
drop if _merge==2

// DFPS Service Regions
spmap region using uscoord_mod, id(id) fcolor(Spectral) clmethod(unique) legenda(on) legend(symy(*1) symx(*1) position(11)) ocolor(gs5 ..) osize(thin) ndocolor(gs6 ..) ndsize(thin) ndlabel(N/A) title("DFPS - Service Regions", size(*0.8)) subtitle("2017", size(*0.8)) note("Source: http://www.dfps.state.tx.us/contact_us/counties.asp")

// CCL Districts
spmap ccldist_code using uscoord_mod, id(id) fcolor(Spectral) clmethod(unique) legenda(on) legend(symy(*1) symx(*1) position(11)) ocolor(gs5 ..) osize(thin) ndocolor(gs6 ..) ndsize(thin) ndlabel(N/A) title("DFPS - CCL District Regions", size(*0.8)) subtitle("2017", size(*0.8)) note("Source: http://www.dfps.state.tx.us/contact_us/counties.asp")

// APS Districts
spmap apsdist_code using uscoord_mod, id(id) fcolor(Spectral) clmethod(unique) legenda(on) legend(symy(*1) symx(*1) position(11)) ocolor(gs5 ..) osize(thin) ndocolor(gs6 ..) ndsize(thin) ndlabel(N/A) title("DFPS - APS District Regions", size(*0.8)) subtitle("2017", size(*0.8)) note("Source: http://www.dfps.state.tx.us/contact_us/counties.asp")
```
