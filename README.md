# Case Study: How Does a Bike-Share Navigate Speedy Success?

### By Gary Yan
### Last Updated: May 22, 2024

[Link to Tableau dashboard](https://public.tableau.com/views/Cyclistic_17163180347710/Dashboard1?:language=en-US&:sid=&:display_count=n&:origin=viz_share_link)

## Introduction

   This is my capstone project for the [Google Data Analytics course](https://www.coursera.org/professional-certificates/google-data-analytics). The goal of the project is answer a business question by following the six stages of the data analysis process: **ask, prepare, process, analyze, share,** and **act**
   
## Scenario 
    
   You are a junior data analyst working in the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of
marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, your
team wants to understand how casual riders and annual members use Cyclistic bikes dierently. From these insights, your team will
design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve your
recommendations, so they must be backed up with compelling data insights and professional data visualizations.

## About the company

   In 2016, Cyclistic launched a successful bike-share offering. Since then, the program has grown to a fleet of 5,824 bicycles that are
geotracked and locked into a network of 692 stations across Chicago. The bikes can be unlocked from one station and returned to
any other station in the system anytime.

Until now, Cyclistic’s marketing strategy relied on building general awareness and appealing to broad consumer segments. One
approach that helped make these things possible was the exibility of its pricing plans: single-ride passes, full-day passes, and
annual memberships. Customers who purchase single-ride or full-day passes are referred to as casual riders. Customers who
purchase annual memberships are Cyclistic members.

Cyclistic’s finance analysts have concluded that annual members are much more protable than casual riders. Although the pricing
exibility helps Cyclistic attract more customers, Moreno believes that maximizing the number of annual members will be key to
future growth. Rather than creating a marketing campaign that targets all-new customers, Moreno believes there is a very good
chance to convert casual riders into members. She notes that casual riders are already aware of the Cyclistic program and have
chosen Cyclistic for their mobility needs.

Moreno has set a clear goal: Design marketing strategies aimed at converting casual riders into annual members. In order to do
that, however, the marketing analyst team needs to better understand how annual members and casual riders differ, why casual
riders would buy a membership, and how digital media could affect their marketing tactics. Moreno and her team are interested in
analyzing the Cyclistic historical bike trip data to identify trends.

## Characters and teams
* Cyclistic: A bike-share program that features more than 5,800 bicycles and 600 docking stations. Cyclistic sets itself apart
by also offering reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities
and riders who can’t use a standard two-wheeled bike. The majority of riders opt for traditional bikes; about 8% of riders use
the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use them to commute to work each
day.

* Lily Moreno: The director of marketing and your manager. Moreno is responsible for the development of campaigns and
initiatives to promote the bike-share program. These may include email, social media, and other channels.

* Cyclistic marketing analytics team: A team of data analysts who are responsible for collecting, analyzing, and reporting
data that helps guide Cyclistic marketing strategy. You joined this team six months ago and have been busy learning about
Cyclistic’s mission and business goals — as well as how you, as a junior data analyst, can help Cyclistic achieve them.

* Cyclistic executive team: The notoriously detail-oriented executive team will decide whether to approve the
recommended marketing program.
        

## Ask

The question assigned to me is **"How do annual members and casual riders use Cyclistic bikes differently?"** In order to answer the question, I will be building a profile of the average casual member as well as the average annual member. I will be using the profiles to see the similarities and differences between the two groups by comparing across different metrics such as 

* Duration of trips on Cyclistic bikes
* Type of bike used
* Month trips are taken
* Day of the week trips are taken
* Time of day trips are taken
* Most common routes taken on trips

The key stakeholders of the project are the director of marketing, Lily Moreno, and the executive team for Cyclistic.


## Prepare

I will be using Cyclistic trip data for 2022 acquired from [here.](https://divvy-tripdata.s3.amazonaws.com/index.html) The data is provided according to the [Divvy Data License Agreement.](https://www.divvybikes.com/data-license-agreement) The data is proviced in files with a comma-seperated values(.csv) datatype. The data is sorted into individual trips and includes
* Trip start day and time
* Trip end day and time
* Trip start station
* Trip end station
* Rider type (member or casual)
* Bike type

The data has been pre-processed to remove trips taken by staff and trips below 60 seconds in length. I am not concerned with bias as the data represents a complete history of trips, nor am I worried about credibility as this is first party data from the company and the city of Chicago. The data has been anonymized in order to protect users' indentifiable information. Due to the anonymization, I will not be able to determine if casual riders use the service multiple times or how many times a particular rider, casual or member, used the service. But using the information that is available, I believe I will still be able to answer the business question.


## Process

I will be using MySQL to process the data. Firstly, I downloaded the data from the website as 12 seperate comma-seperated value (.csv) files. I created a database in MySQL Workbench called *cyclistic_trips_2022*. Afterwards, I created tables for each month and loaded the data from the csv files into the tables using the LOAD DATA INFILE statement. This left me with 12 tables, one for each month in 2022. In each table there were entries for rides that took place containing data for 9 fields: *ride_id, rideable_type, started_at, ended_at, start_station_name, start_station_id, end_station_name, end_station_id,* and *member_casual.* From there, I could copy data from the 12 tables into one big combined table, which makes it easier to access all of the data and I can clean without altering the originals. After compiling all of the year's data into one table, I took the datetime entries from the *started_at* and *ended_at* fields and separated them into date and time. Lastly, I added another field for the day of the week for each ride.

To clean the data, first I looked in the *member_casual* field and deleted entries where there were nulls. There were also some entries that were empty in the *start* and *end station* fields that I also removed. Next, I found that there were entries where the end time of the ride occurred prior to the starting time and I removed those entries. Due to the cleaning, a total of *1 298 519* entries were removed and *4 369 198* entries remain.


## Analyze

#### Proportion of trips taken by casual riders and members
First I'd like to see how many trips are taken by either groups throughout the year, as a baseline. I can do this by using a SQL query:

    SELECT member_casual, 
        COUNT(*) AS trips_taken, 
        ROUND(COUNT(*) / (SELECT COUNT(*) FROM cyclistic_trips_2022.trips_2022) * 100, 2) AS percentage
    FROM cyclistic_trips_2022.trips_2022
    GROUP BY member_casual;

giving us the following output.

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/8a717bc5-5d6c-4e41-bb45-d41d6bdac8bd)

Across the entire year, we can see that members take approximately 60% of trips. And we can also see how this changes depending on the day of the week using this query:

    WITH trip_counts AS (
        SELECT day_of_the_week,
            COUNT(DISTINCT ride_id) AS total_trips,
            SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips,
            SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips
        FROM cyclistic_trips_2022.trips_2022
        GROUP BY day_of_the_week
    )
    SELECT day_of_the_week,
        total_trips,
        casual_trips,
        member_trips,
        ROUND(casual_trips * 100.0 / total_trips, 2) AS casual_percentage,
        ROUND(member_trips * 100.0 / total_trips, 2) AS member_percentage
    FROM trip_counts
    ORDER BY total_trips DESC;

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/1ee07483-6480-49f0-9e4a-18ecf61716c9)


From these results, we can see that the number of trips taken by casual riders exceeds members on Saturdays and Sundays. We also observe that there are more trips on Saturdays than any other day of the week. 

We can also see how this breakdown changes depending on the month with a similar query:

    WITH trip_counts AS (
        SELECT MONTH(started_at) AS trip_month,
            COUNT(DISTINCT ride_id) AS total_trips,
            SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips,
            SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips
        FROM cyclistic_trips_2022.trips_2022
        GROUP BY trip_month
    )
    SELECT trip_month,
        total_trips,
        casual_trips,
        member_trips,
        ROUND(casual_trips * 100.0 / total_trips, 2) AS casual_percentage,
        ROUND(member_trips * 100.0 / total_trips, 2) AS member_percentage
    FROM trip_counts
    ORDER BY total_trips DESC;

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/d4fe23df-fc3c-4e61-817d-221a046d52c8)

Viewing this table, we can see that the number of trips is greatest in the summer months (May, June, July) and lowest in the winter months(December, January, February). We also see that the share of trips taken by casual riders peaks in the summer months at almost half of all trips and falls to fewer than 1/5 trips in the winter.

Combining this result with the previous one, this suggests that casual riders are more likely to use our service for recreation or leisure, as this coincides with when they'd likely have more time and the weather would be warmer.

#### Timing of trips
I'd also like to see the distribution of when trips begin, what the split is for casual/members, and also how this changes depending on if it's a weekday or weekend. To accomplish this, we can use the following query:

    WITH trip_timing AS(
        SELECT HOUR(started_time) AS start_time,
            COUNT(DISTINCT ride_id) AS total_trips,
            SUM(CASE WHEN member_casual = 'casual' THEN 1 ELSE 0 END) AS casual_trips,
            SUM(CASE WHEN member_casual = 'member' THEN 1 ELSE 0 END) AS member_trips
        FROM cyclistic_trips_2022.trips_2022
        GROUP BY start_time
    )
    SELECT start_time, 
        total_trips,
        casual_trips,
        member_trips,
        ROUND(casual_trips * 100.0 / total_trips, 2) AS casual_percentage,
        ROUND(member_trips * 100.0 / total_trips, 2) AS member_percentage
    FROM trip_timing;

    
![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/35946c8f-8a0e-4a39-84b6-c04eb787ae4e)

From the table, we can see there's a larger proportion of members taking trips in the morning and evening, which suggest that there's more members using our service for a commute. We can see how this differs on weekends and weekdays with a small adjustment to the query

![Weekdays](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/bc8532f5-4650-4613-b28f-1d8b0ee87def)
Weekdays (Mon-Fri)

![Weekends](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/bf26eacd-1713-4eeb-a85a-c9e77a59dde9)
Weekends (Sat & Sun)

Here, we can see a difference in the distribution of trips on weekends and weekdays, where we see a bit of a spike in trips in the mornings and evenings on weekdays, especially amongst members. Whereas on weekends, there isn't a spike in trips and we see that it's fairly well distributed throghout the day, with demand being highest in the early afternoon. This is additional evidence for members being more likely to use our services for their commute to work.

#### Duration of trips
Using the following SQL query:

    SELECT member_casual, 
        SEC_TO_TIME(AVG(timediff(ended_at, started_at))) AS trip_duration
    FROM cyclistic_trips_2022.trips_2022
    GROUP BY member_casual;

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/f0b56cac-22ea-4189-8031-125d1f29137a)

we can find the average trip duration based on member status.
For casual riders, an average trip lasts for 47:30 and for members it is 21:12. We can see how this changes depending on if it's a weekday or a weekend as well.

![Weekdays](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/76714e98-30e7-49e6-acdf-efae5436d61e)
Weekdays (Mon-Fri)

![Weekends](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/e7fd25c4-32b8-41ce-8a96-077ead86dc0d)
Weekends (Sat & Sun)

We can see that trips are typically shorter on weekdays than on weekends for both groups which we would expect if there's a tendency to use our service for leisure rather than commuting on weekends. We can also see that trips are still significantly longer for casual riders than members across the whole week, lending more support to the idea that casual riders are more likely to use our service for leisure.

#### Common trip routes

Since we don't have data tracking the path that our bikes travel during trips, we can try to look at where trips begin and end to make educated guesses as to the paths our riders are taking. For now, we are going to take a look at the top 20 "routes" for members and casual riders to get an idea of how they may be similar or may differ. To do that, we can use the following query:
    
    SELECT start_station_name,
        end_station_name,
        COUNT(*) as trip_count
    FROM cyclistic_trips_2022.trips_2022
    WHERE member_casual = 'casual'
    GROUP BY start_station_name, end_station_name
    ORDER BY trip_count DESC
    LIMIT 20;

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/70c42c60-25ec-40e0-8260-f20f038d1d22)
Casual riders

Looking at the casual riders, we see that a lot of the most taken trips involve two stations: *Streeter Dr & Grand Ave* and *DuSable Lake Shore Dr & Monroe St*. If we take a look at Chicago, we can see that both of these are near some large parks and close to a long mixed-use path called Lakefront Trail. A lot of the most used stations are near this path so I believe it's fair to assume that there's a lot of trips taken along the path. Some of the other commonly used stations are for attractions close to the lakeshore such as Adler Planetarium and Shedd Aquarium.

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/0994c887-0ab9-48e7-a6d6-7007c4dfe9ee)
Members

When we take a look at the members, the first thing that jumps out is that 10 of the top 20 routes either begin or finish on *Ellis Ave* or *University Ave*, these are all around the campus of the *University of Chicago*. We also see that four of the routes involve the station at *State St & 33rd St*, which is where the campus of the *Illinois Institute of Technology* is. Furthermore, of the remaining routes, five of them are centred around the *University of Illinois at Chicago*. This means of the top 20 routes, 19 of them are for arriving at or departing a university. As for the remaining route, it is the only one shared by casual riders and members which begins and finishes at *Streeter Dr & Grand Ave*.

This shows us that our hypothesis that casual riders using our service for leisure/entertainment  and members using it for commuting is at least plausible. The surprising result is how the trips taken by members are likely students using it not only to travel to and from school but also within the campus. If we look at some of the most common starting stations for members and overlay it onto a map, we'll see that they tend to be close to stops on the metro system suggesting that members are using our service as a last mile option. This trend also seems to exist for casual riders.

#### Bike Types

![image](https://github.com/GaryYan28/Google-Data-Analytics-Bike-Share-Case-Study/assets/170465501/ac34ab61-b1e1-4ea9-babe-7839b6715523)

Breaking down how members and casual riders choose a bike type, the data shows that throughout the year there isn't a significant difference in how often each group chooses electric bikes, with both having a bit under 40% of their rides being on e-bikes. One thing of note is that docked bikes are only ridden by casual riders which likely means it's only an option for casual riders. But without any more information on whether those bikes are classic or electric and how they work, I will choose to ignore them from the rest of my analysis. 

One trend that is worth observing and following is adoption of electric bikes has increased throughout the year. Electric bikes were used in only about 30% of all trips in January but by December, 44.50% of all trips were taken on electric bikes. This trend holds true for both casual riders as well as members but is more noticeable in casual riders with 54.06% of their trips being on electric bikes in December. This is an interesting trend but it's hard to draw a strong conclusion from it as there isn't that much data, especially because there is a dip in e-bike usage in May and June, which are two of the most popular months for our service. I believe that this is still worth tracking though and there's possibly insights to be gained with more data.


## Share

[Link to Tableau dashboard here](https://public.tableau.com/views/Cyclistic_17163180347710/Dashboard1?:language=en-US&:sid=&:display_count=n&:origin=viz_share_link)

Key points

* More trips are taken by members than casual riders throughout the year.
* Significantly more trips are taken in summer months than winter months, especially among casual riders.
* Spike in ridership among members in the morning and in the evening whereas casual trips gradually increase throughout morning and afternoon up to a peak in the evening.
* Members take more trips than casuals on weekdays and fairly consistent number of trips throughout week.
* Casual riders take significantly more trips on weekends, more than members in warmer months.
* Members will take mostly short trips and few long trips, casuals have more diverse duration of trips but still have more short trips than long ones.
* Classic bikes more popular than electric bikes but e-bikes are rapidly gaining popularity, especially among casual riders.


## Act

The largest block of casual riders are using our service for leisure so they're a good option to try to reach through advertising. A good way of achieving this is by highlighting the various activities and attractions across Chicago, particularly centred around the lakeshore. We could also emphasize biking as a healthy and fun option for exploring the city. If we can get these users interested in using our service more often then we can advertise the cost savings of a membership. Furthermore, we could offer different plans, such as a monthly plan or a promotional plan for only the summer months to attract members who do not want to commit to an annual plan. Another idea would be to offer group plans for families or friends who'd like to ride together.

Secondly, we can see in the data that a lot of members are riding to and from the universities in Chicago. Some of them are using it to commute to and from the schools while others are using it as a form of transportation within the school. It's mostly members that are making these trips but we can also see sizeable numbers of casual riders making these trips as well. By highlighting the cost savings by signing up for a membership if they're consistently making these trips, we may be able to convert some of these users into members. Once again, a monthly membership could make them more likely to sign up as they may not want to commit to an annual membership if there's no class during the summer. It's also fairly common for businesses to offer promotions or discounts students and faculty of universities, so that may be another avenue of making a membership more attractive.

Finally, from the data, we can observe that there is an uptick in trips taken by members during regular commute times. Thus, it is likely that members are interested in our service for commuting. We can also note that there's greater usage for our bike stations that are near transportation hubs or a stop on the metro line. From this, we can assume they are using our bikes to get from these areas to their destinations. I believe by advertising our service as a *last mile* solution, we could attract members who may not have considered biking as a form of commuting. By combining different forms of transportation, we may open up the possibility of using our service to users who would otherwise find it difficult to bike these distances. And by combining our service with public transit, we could position this as a greener and healthier form of commute and a solution for people who may want to avoid sitting in traffic.

For any of the above solutions, I believe the best time to advertise is going to be in the late spring and early summer (April - July) as these months have the greatest surge in trips taken by casual riders, the jump between April and May is especially large. Since this time coincides with the greatest volume of users, it's likely to reach the greatest number of eyes and gives us a good opporunity to convert casual riders into members. 
