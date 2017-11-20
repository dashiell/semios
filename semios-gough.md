
## Relational Data: Answers (Can paste all of below directly into SQLFiddle)
-- 1. What are the names of the blocks which are less than 40 acres? 

select name from blocks where acres < 40;

-- 2. What are the names of the orchards that have at least one block that is more than 10 acres? 

select orchards.name from orchards 
inner join blocks on orchards.id = blocks.id where blocks.acres > 10;

-- 3. What are the names of the orchards which have exactly two blocks? 

select name from orchards inner join 
( select orchard_id, count(orchard_id) from blocks group by orchard_id having (count(orchard_id) = 2) ) as b
on orchards.id = b.orchard_id;

-- 4. What are the names of the orchards that are growing almonds? 

select orchards.name from orchards 
inner join blocks on blocks.orchard_id = orchards.id
inner join blocks_crops on blocks_crops.block_id = blocks.id
inner join crops on crops.id = blocks_crops.crop_id
where crops.id = 3;

-- 5. What is the name and size of the largest block on each orchard? 

select orchard_name, largest_block_name, acres from (
  select o.name as orchard_name, b.name as largest_block_name, b.acres,
  row_number() over(partition by o.id order by b.acres DESC) as size_rank
  from blocks b
  inner join orchards o on o.id = b.orchard_id
)t where t.size_rank = 1;
## Big Data: Answers
-- 1. What is the time range of this dataset?

select min(stamp) as min_date, max(stamp) as max_date from data_vis.batteries

-- 2. On Oct 25th 2017 (UTC), how many devices reported battery greater than 3.21V?

select count(*) as device_count_2017_10_25 from (
  select device
  from data_vis.batteries 
  where stamp>=timestamp('2017-10-25') and stamp < timestamp('2017-10-26') 
  and battery > 3.21
  group by device
)

-- 3. What is the most recent date & time that device: ExJk1KcFWI0= reported its lowest level?

select max(stamp) as most_recent from data_vis.batteries where device='ExJk1KcFWI0=' and battery = (
  select min(battery) 
  from data_vis.batteries 
  where device='ExJk1KcFWI0='
) 

-- 4. What is the second lowest battery level ever reported by device:RYhYNz0tx4o= ?

select min(battery) from data_vis.batteries where device='RYhYNz0tx4o=' and battery not in (
  select battery 
  from data_vis.batteries 
  where device='RYhYNz0tx4o='
  group by battery
  order by battery
  limit 1
) 

-- 5. Use a graphing/plotting tool of your choice to show the distribution of battery levels in Washington State (state:WA) on Oct 25th 2017 @ 11:20am PDT. i.e. for each battery level, how many devices were there?

select * from (
  select device, battery, timestamp(datetime(stamp, "America/Los_Angeles")) as pdt_time
  from data_vis.batteries
  where state='WA'
) t 
where t.pdt_time >= timestamp('2017-10-25 11:20:00') and t.pdt_time < timestamp('2017-10-25 11:21:00')



-- 6. Internal Semios Engineers will often be your customer. Use a graphing/plotting tool of your choice to display battery discharge curves on orchard:Vu2HeM9+

--query 1
select * from (
  select *, row_number() over() +0 as row_number 
  from `data_vis.batteries` 
  where orchard='Vu2HeM9+'
) t
where t.row_number <= 10000

-- query 2
select * from (
  select *, row_number() over() +0 as row_number 
  from `data_vis.batteries` 
  where orchard='Vu2HeM9+'
) t
where t.row_number > 10000 and t.row_number <= 20000

-- query 3
select * from (
  select *, row_number() over() +0 as row_number 
  from `data_vis.batteries` 
  where orchard='Vu2HeM9+'
) t
where t.row_number > 30000
Q5 - Plot

![q5.png](attachment:q5.png)




Q6 - Plot

![q6.png](attachment:q6.png)
