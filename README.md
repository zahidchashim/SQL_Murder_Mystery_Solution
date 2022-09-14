# [SQL_Murder_Mystery_Solution](https://mystery.knightlab.com/)

Hi! I am Zahid and I am a beginner in SQL.
This is my attempt on solving [SQL Murder Mystery](https://mystery.knightlab.com/) ↓

Q: A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a **murder** that occurred sometime on **Jan.15, 2018** and that it took place in **SQL City**. Start by retrieving the corresponding crime scene report from the police department’s database.

Given the schema diagram below ↓
![schema.png](https://mystery.knightlab.com/schema.png)

### 1) Information about the murder
```SQL
SELECT 
  *
FROM 
  crime_scene_report
WHERE type = "murder"
AND date = "20180115"
AND city = "SQL City"
```

Output:
| date  | type  | description |city |
|---    |---    |---          |---  |
|20180115|murder|Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".|SQL City

### 2.1) Finding the witnesses - The first witness
```SQL
SELECT
  id,
  name,
  license_id,
  MAX(address_number) AS address_number,
  address_street_name,
  ssn,
  interview.transcript
FROM 
  person
JOIN interview ON 
  person.id = interview.person_id
WHERE address_street_name = "Northwestern Dr"
```

Output:
|id   |name   |license_id   |address_number   |address_street_name    |ssn    |transcript|
|---  |---    |---          |---              |---                    |---    |---        |
|14887|Morty Schapiro|118009|4919|Northwestern Dr|111564949|I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".|

### 2.2) Finding the witnesses - The second witness
```SQL
SELECT
  *
FROM 
  person
JOIN interview ON 
  person.id = interview.person_id
WHERE address_street_name = "Franklin Ave"
AND name LIKE "Annabel%"
```

Output:
|id   |name   |license_id   |address_number   |address_street_name    |ssn    |person_id| transcript|
|---  |---    |---          |---              |---                    |---    |---        |---|
|16371|Annabel Miller|490173|103|Franklin Ave|318771143|16371|I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.|

### 3) Clues given by the witnesses about the murderer
- He is a **Get Fit Now Gym member**
- Membership number starts with **"48Z"**, a **"gold"** member
- Car plate includes character **"H42W"**
- Just recently work out at the gym on **January 9th**

### 4) Finding the murderer
- Based on the clues given by the witnesses (Refer no. 3), I decide to join the **"get_fit_now_check_in", "get_fit_now_members", "interview", "person" and "drivers_license"** tables.
```SQL
SELECT 
  person.name, 
  get_fit_now_check_in.membership_id, 
  drivers_license.plate_number, 
  transcript
FROM 
  get_fit_now_member
JOIN get_fit_now_check_in ON 
  get_fit_now_check_in.membership_id = get_fit_now_member.id
JOIN person ON 
  get_fit_now_member.person_id = person.id
JOIN drivers_license ON 
  person.license_id = drivers_license.id
JOIN interview ON 
  get_fit_now_member.person_id = interview.person_id
WHERE membership_status = "gold"
AND get_fit_now_member.id LIKE "48Z%"
AND drivers_license.plate_number LIKE "%H42W%" 
```
Output:
|name   |membership_id   |plate_number    |transcript   |
|---    |---             |---             |---          |
|Jeremy Bowers|48Z55|0H42W2|I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.|

## 5) Checking the answers
- We have found the murderer! Oh wait.. it seems that Jeremy Bowers is just the hitman. He said "I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.". We need to find her.

## 6) Clues given
- She has a lot of money
- Height is around 65" to 67"
- Hair color is red
- Drives a Tesla Model S
- Attended "SQL Sympony Concert" three times in December 2017

## 7) Finding the actual killer
```SQL
SELECT
  person.id as person_id,
  person.name,
  drivers_license.height,
  drivers_license.hair_color,
  drivers_license.car_make||" "||drivers_license.car_model AS car_name,
  COUNT(facebook_event_checkin.event_name) AS event_checkin_times
FROM 
  facebook_event_checkin
JOIN person ON 
  facebook_event_checkin.person_id = person.id
JOIN drivers_license ON 
  person.license_id = drivers_license.id
WHERE event_name = "SQL Symphony Concert"
AND date LIKE "201712%"
AND hair_color = "red"
AND car_name = "Tesla Model S"
GROUP BY person_id
```

Output:
|person_id    |name             |height           |hair_color   |car_name       |event_check_in_times |
|---          |---              |---              |---          |---            |---                  |
|99716        |Miranda Priestly |66               |red          |Tesla Model S  |3                    |

There you go! The actual killer is **Miranda Priestly**

## 8) Checking my solution
### Did you find the killer?
```SQL
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
        SELECT value FROM solution;
```

Output:
|value|
|---|
|Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.|

```SQL
INSERT INTO solution VALUES (1, 'Miranda Priestly');
        SELECT value FROM solution;
```
Output:
|value|
|---|
|Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!|
