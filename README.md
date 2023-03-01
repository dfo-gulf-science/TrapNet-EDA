# TrapNet-EDA
Exploring old and new data to see if we can match them in our database.

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

```
