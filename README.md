# [SQL_Murder_Mystery_Solution](https://mystery.knightlab.com/)

Hi! I am Zahid and I am a beginner in SQL.
This is my attempt on solving [SQL Murder Mystery](https://mystery.knightlab.com/) ↓

Q: A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a **murder** that occurred sometime on **Jan.15, 2018** and that it took place in **SQL City**. Start by retrieving the corresponding crime scene report from the police department’s database.

Given the schema diagram below ↓
![schema.png](https://mystery.knightlab.com/schema.png)

### 1) Information about the murder
```SQL
SELECT *
FROM crime_scene_report
WHERE type = "murder"
AND date = "20180115"
AND city = "SQL City"
```

Answer:
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
FROM person
JOIN interview ON person.id = interview.person_id
WHERE address_street_name = "Northwestern Dr"
```

Answer:
|id   |name   |license_id   |address_number   |address_street_name    |ssn    |transcript|
|---  |---    |---          |---              |---                    |---    |---        |
|14887|Morty Schapiro|118009|4919|Northwestern Dr|111564949|I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".|

### 2.2) Finding the witnesses - The second witness
```SQL
SELECT
*
FROM person
JOIN interview ON person.id = interview.person_id
WHERE address_street_name = "Franklin Ave"
AND name LIKE "Annabel%"
```

Answer:
|id   |name   |license_id   |address_number   |address_street_name    |ssn    |person_id| transcript|
|---  |---    |---          |---              |---                    |---    |---        |---|
|16371|Annabel Miller|490173|103|Franklin Ave|318771143|16371|I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.|

### 3) Clue about the murderer
- He is a Get Fit Now Gym member
- Membership number starts with **"48Z"**, a **"gold"** member
- Car plate includes character **"H42W"**
