# TrapNet-EDA
Exploring archived and current data to see if we can match biological samples with normal specimen within our database.

NOTE: Initially, I referred to biological detailing specimen as 'historical' based on the nature if this data analysis task. However, 'biological detailing' is a more apt name (or 'bio' for short), so I have switched. However, 'hist' and 'historical' remain throughout the analysis (e.g., 'hist', 'n_hist', 'df_hist'), where they refer to the biologicaldetailing table.

# Summary of Findings
1. Data import into dm_apps was 1-1 with archived data.
    * one import issue with 6 specimen occurred when fork_length == Null - these bio samples were classified as normal specimen even though they have sex info
    * df_spec[df_spec.sex_id.notnull()]
        * 6 specimen in samples 4456 and 4475
2. Normal specimen (further refered to as specimen) and biological detailing specimen (bio) are differentiable by their fork_length distributions
    * bio have mm resolution, whereas specimen are binned into 5mm bins (last digit is 3 or 8 in over 97% of samples)
![image](https://user-images.githubusercontent.com/94803263/224027483-22d954d5-d6a1-44bf-84fb-d95a2a419280.png)
3. A strong correlation between error tolerance and number of matchable samples was noted, likley due to the above bin resolution issue.
    * further, this correlation was found to be almost entirely due to the bin size
    * for this reason, bins of width exactly == 5 (+- 2-3) were found to optimally match bio and specimen
![image](https://user-images.githubusercontent.com/94803263/224028520-fdfea951-c61c-45e3-b907-0723c9270ef1.png)
4. Matching and comparing distributions between bio and specimen did not correlate with projected matches because details distributions often vary significantly from the full sample.
    * this is not surprising, because detailing on a variety of fork_lengths is preferred 
![image](https://user-images.githubusercontent.com/94803263/224029547-8ff2b699-9c94-4cb5-bb26-65b381a3a5fc.png)
5. The implementation of a bin matching algorithm lead to the insight that left vs right inclusive bin edges significantly affected the number of exact matches
![image](https://user-images.githubusercontent.com/94803263/224030698-7771565f-b4ae-4d16-bec3-594b7454a43d.png)
    * using left inclusive bin edges (the default for matplotlib histograms), there were 203 fully matched samples (all bio within specimen bins)
    * example:
        * out of 793 samples this equates to 26% match rate, compared to less than 3% expected by chance
![image](https://user-images.githubusercontent.com/94803263/224031503-b53c552e-78dc-4c61-b0ec-5389037e9938.png)
    * using right inclusive bin edges (the default for pandas .cut()), there were 534 fully matched samples
    	* out of 793 samples this equates to 67% match rate, compared to less than 3% expected by chance
	* example:
![image](https://user-images.githubusercontent.com/94803263/224031404-d416c7ad-f0e4-4056-a144-78d76f4055a3.png)
6. In addition to exact matches, there were a number of samples where matches could be ruled out:
    * samples with no specimen, only bio
    * samples with more bio and specimen
    * samples with a high degree of error (number of bin offsets/errors required to assume match between bio with specimen)
7. Overall, recommendations were made for 604 out 793 samples (approximately 76%).
    * the remaining samples are ambigous and do not appear to have enough evidence to deduce whether or bio are included within the specimen or not

# APPENDIX

# Useful SQL Queries

```
-- number of samples with historical and specimen data
SELECT COUNT(*) FROM (
	SELECT trapnet_sample.id
		FROM trapnet_sample
			JOIN trapnet_biologicaldetailing ON trapnet_biologicaldetailing.sample_id = trapnet_sample.id
			JOIN trapnet_specimen ON trapnet_specimen.sample_id = trapnet_sample.id
	GROUP BY trapnet_sample.id		
)


-- list of sample id with historical and specimen data
SELECT trapnet_sample.id
	FROM trapnet_sample
		JOIN trapnet_biologicaldetailing ON trapnet_biologicaldetailing.sample_id = trapnet_sample.id
		JOIN trapnet_specimen ON trapnet_specimen.sample_id = trapnet_sample.id
GROUP BY trapnet_sample.id


-- historical data from matches
SELECT COUNT(*) FROM trapnet_biologicaldetailing
--SELECT * FROM trapnet_biologicaldetailing
WHERE trapnet_biologicaldetailing.sample_id IN (
	SELECT trapnet_sample.id
		FROM trapnet_sample
			JOIN trapnet_biologicaldetailing ON trapnet_biologicaldetailing.sample_id = trapnet_sample.id
			JOIN trapnet_specimen ON trapnet_specimen.sample_id = trapnet_sample.id
	GROUP BY trapnet_sample.id	
)


-- specimen data from matches
SELECT COUNT(*) FROM trapnet_specimen
--SELECT * FROM trapnet_specimen
WHERE trapnet_specimen.sample_id IN (
	SELECT trapnet_sample.id
		FROM trapnet_sample
			JOIN trapnet_biologicaldetailing ON trapnet_biologicaldetailing.sample_id = trapnet_sample.id
			JOIN trapnet_specimen ON trapnet_specimen.sample_id = trapnet_sample.id
	GROUP BY trapnet_sample.id
)


-- are all historical measurements electrofishing?
SELECT sample_type, COUNT(sample_type) FROM trapnet_sample
JOIN trapnet_biologicaldetailing ON trapnet_sample.id = trapnet_biologicaldetailing.sample_id
GROUP BY sample_type  -- yes


-- are they all atlantic salmon?
SELECT species_id, COUNT(species_id) FROM trapnet_sample
JOIN trapnet_biologicaldetailing ON trapnet_sample.id = trapnet_biologicaldetailing.sample_id
GROUP BY species_id  -- yes (79 pk -> 161996 tsn, atlantic salmon) 


-- how many samples have sweep_number == 0?
-- plenty
SELECT sweep_number, COUNT(*) FROM trapnet_specimen
JOIN trapnet_sweep ON trapnet_specimen.sweep_id = trapnet_sweep.id
GROUP BY sweep_number

-- how many samples have sweep_number == 0 and include historical data?
-- only 6 (as found in archived data)
SELECT sweep_number, COUNT(*) FROM trapnet_specimen
JOIN trapnet_sweep ON trapnet_specimen.sweep_id = trapnet_sweep.id
WHERE trapnet_specimen.sample_id IN (
	SELECT trapnet_sample.id
		FROM trapnet_sample
			JOIN trapnet_biologicaldetailing ON trapnet_biologicaldetailing.sample_id = trapnet_sample.id
			JOIN trapnet_specimen ON trapnet_specimen.sample_id = trapnet_sample.id
	GROUP BY trapnet_sample.id
	)
GROUP BY sweep_number


-- percent of sex_id in spec table
SELECT SUM(sex_id NOTNULL)*100.0 / COUNT(*) AS '% with sex_id' FROM trapnet_specimen

-- percent of sex_id in bio table
SELECT SUM(sex_id NOTNULL)*100.0 / COUNT(*) AS '% with sex_id' FROM trapnet_biologicaldetailing

-- percent of weight in spec table
SELECT SUM(weight NOTNULL)*100.0 / COUNT(*) AS '% with weight' FROM trapnet_specimen

-- percent of weight in bio table
SELECT SUM(weight NOTNULL)*100.0 / COUNT(*) AS '% with weight' FROM trapnet_biologicaldetailing



```
